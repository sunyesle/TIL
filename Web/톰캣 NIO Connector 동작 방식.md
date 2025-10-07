# 톰캣 NIO Connector 동작 방식
## 구성 요소
<img width="901" height="381" alt="nio connector" src="https://github.com/user-attachments/assets/f3e4c8e5-3d3a-4392-adfd-914af135ae53" />

- **Acceptor**: 새로운 TCP 연결을 수락하는 스레드이다.
- **Poller**: Selector 기반으로 I/O 이벤트를 감시 스레드이다.
- **Worker**: 스레드 풀에서 관리되는 스레드로 실질적인 요청 처리를 담당한다.

## 동작 방식
소스 코드를 기반으로 NIO Connector의 내부 동작 흐름을 분석해보자.

톰캣이 시작되면 worker 스레드 풀을 생성하고, acceptor와 poller 스레드를 시작하게 된다.
```java
public class NioEndpoint extends AbstractNetworkChannelEndpoint<NioChannel,SocketChannel> {

    /**
     * Start the NIO endpoint, creating acceptor, poller threads.
     */
    @Override
    public void startInternal() throws Exception {
        if (!running) {
            ...
            // worker 스레드 풀 생성
            if (getExecutor() == null) {
                createExecutor();
            }

            initializeConnectionLatch();

            // poller 스레드 시작
            poller = new Poller();
            Thread pollerThread = new Thread(poller, getName() + "-Poller");
            pollerThread.setPriority(threadPriority);
            pollerThread.setDaemon(true);
            pollerThread.start();

            // acceptor 스레드 시작
            startAcceptorThread();
        }
    }
}
```

```java
public abstract class AbstractEndpoint<S, U> {

    public void createExecutor() {
        internalExecutor = true;
        if (getUseVirtualThreads()) {
            executor = new VirtualThreadExecutor(getName() + "-virt-");
        } else {
            TaskQueue taskqueue = new TaskQueue(maxQueueSize);
            TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
            executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), getThreadsMaxIdleTime(),
                    TimeUnit.MILLISECONDS, taskqueue, tf);
            taskqueue.setParent((ThreadPoolExecutor) executor);
        }
    }

    protected void startAcceptorThread() {
        acceptor = new Acceptor<>(this);
        String threadName = getName() + "-Acceptor";
        acceptor.setThreadName(threadName);
        Thread t = new Thread(acceptor, threadName);
        t.setPriority(getAcceptorThreadPriority());
        t.setDaemon(getDaemon());
        t.start();
    }
}
```

**Acceptor**는 클라이언트의 새로운 TCP 연결을 수락하는 스레드로, 무한 루프 형태로 동작한다.<br>
내부적으로는 다음과 같은 절차를 수행한다.

1. `endpoint.serverSocketAccept()` 메서드를 호출하여 새로운 연결을 수락한다.
2. 생성된 소켓이 유효하면 `endpoint.setSocketOptions()`를 호출하여 적절한 옵션을 설정한다.
3. 이후 해당 소켓을 `PollerEventQueue`에 등록한다. 

```java
public class Acceptor<U> implements Runnable {
    
    @Override
    public void run() {
        ...
        while (!stopCalled) {
            ...
            // 연결을 수락한다.
            socket = endpoint.serverSocketAccept();
            
            ...
            // 소켓 설정
            if (!stopCalled && !endpoint.isPaused()) {
                // 소켓이 유효하면 적절한 옵션을 설정한다.
                if (!endpoint.setSocketOptions(socket)) {
                    endpoint.closeSocket(socket);
                }
            } else {
                endpoint.destroySocket(socket);
            }
        }
    }
}
```

`setSocketOptions()` 메서드에서는 소켓 채널을 논블로킹으로 전환하고,
소켓을 `SocketChannel -> NioChannel -> NioSocketWrapper` 형태로 캡슐화한 뒤 `poller.register()`를 호출하여 Poller 이벤트 등록을 진행한다.

`poller.register()`가 호출되면, 내부적으로 다음과 같은 작업이 수행된다.
1. 소켓의 관심 이벤트를 `OP_READ`로 설정한다.
2. `NioSocketWrapper`와 이벤트 타입 `OP_REGISTER`을 묶어 `PollerEvent` 객체를 생성한다.
3. 생성된 `PollerEvent` 객체를 `PollerEventQueue`에 추가한다.

이로써 Acceptor의 역할은 끝나고 다음 작업은 Poller 스레드에서 이루어진다.

```java
public class NioEndpoint extends AbstractNetworkChannelEndpoint<NioChannel,SocketChannel> {

    @Override
    protected boolean setSocketOptions(SocketChannel socket) {
        ...
        channel = createChannel(bufhandler);
        
        ...
        // NioChannel -> NioSocketWrapper 캡슐화
        NioSocketWrapper newWrapper = new NioSocketWrapper(channel, this);
        channel.reset(socket, newWrapper);
        connections.put(socket, newWrapper);
        socketWrapper = newWrapper;

        ...
        // 논블로킹 설정
        socket.configureBlocking(false);
        
        ...
        // Poller 이벤트 등록
        poller.register(socketWrapper);
        return true;
    }

    public class Poller implements Runnable {
        private final Selector selector;
        private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

        // Poller 이벤트 등록
        public void register(final NioSocketWrapper socketWrapper) {
            socketWrapper.interestOps(SelectionKey.OP_READ);
            PollerEvent pollerEvent = createPollerEvent(socketWrapper, OP_REGISTER);
            addEvent(pollerEvent);
        }

        private void addEvent(PollerEvent event) {
            events.offer(event);
            if (wakeupCounter.incrementAndGet() == 0) {
                selector.wakeup();
            }
        }
    }
}
```

**Poller**는 NIO 기반의 논블로킹 I/O 이벤트 감지를 담당하는 스레드로, 무한 루프 형태로 동작한다.<br>
내부적으로는 다음과 같은 절차를 수행한다.

1. `PollerEventQueue`에서 `PollerEvent` 객체를 꺼내 소켓을 `Selector`에 등록한다.
2. `Selector`를 통해 감지된 I/O 이벤트를 처리한다.

```java
public class NioEndpoint extends AbstractNetworkChannelEndpoint<NioChannel,SocketChannel> {
    
    public class Poller implements Runnable {

        @Override
        public void run() {
            while (true) {
                ...
                // 등록된 PollerEvnet 확인 및 처리
                hasEvents = events();
                
                ...
                // 이벤트가 감지된 집합을 순회
                Iterator<SelectionKey> iterator = keyCount > 0 ? selector.selectedKeys().iterator() : null;
                
                while (iterator != null && iterator.hasNext()) {
                    SelectionKey sk = iterator.next();
                    iterator.remove();
                    NioSocketWrapper socketWrapper = (NioSocketWrapper) sk.attachment();
                    
                    if (socketWrapper != null) {
                        // 프로세싱
                        processKey(sk, socketWrapper);
                    }
                }
            }
        }
    }

    public boolean events() {
        // 이벤트 큐에서 PollerEvent 객체를 하나씩 꺼내 처리한다.
        for (int i = 0, size = events.size(); i < size && (pe = events.poll()) != null; i++) {
            NioSocketWrapper socketWrapper = pe.getSocketWrapper();
            SocketChannel sc = socketWrapper.getSocket().getIOChannel();
            int interestOps = pe.getInterestOps();
            
            ...
            } else if (interestOps == OP_REGISTER) {
                // Selector에 소켓 채널을 등록한다.
                sc.register(getSelector(), SelectionKey.OP_READ, socketWrapper);
            }
            ...
        }

        return result;
    }

    protected void processKey(SelectionKey sk, NioSocketWrapper socketWrapper) {
        ...
        // SelectionKey의 I/O 이벤트 유형에 따라 실제 이벤트 처리 로직으로 분기한다.
        if (sk.isReadable()) {
            ...
            } else if (!processSocket(socketWrapper, SocketEvent.OPEN_READ, true)) {
                closeSocket = true;
            }
        }
        if (!closeSocket && sk.isWritable()) {
            ...
            } else if (!processSocket(socketWrapper, SocketEvent.OPEN_WRITE, true)) {
                closeSocket = true;
            }
        }
    }
}
```

`processSocket()` 메서드는 Tomcat이 자체 구현한 스레드 풀(ThreadPoolExecutor)에 작업을 위임한다.

구체적으로는 `SocketProcessor`라는 작업(Task) 객체를 생성하여 스레드 풀에 제출하고, 이후 워커 스레드가 실질적인 요청 처리를 수행하게 된다.
```java
public abstract class AbstractEndpoint<S, U> {
    public boolean processSocket(SocketWrapperBase<S> socketWrapper, SocketEvent event, boolean dispatch) {
        ...
        // SocketProcessor라는 작업(Task) 객체를 생성한다.
        SocketProcessorBase<S> sc = null;
        if (sc == null) {
            sc = createSocketProcessor(socketWrapper, event);
        } else {
            sc.reset(socketWrapper, event);
        }
        
        // 스레드 풀에 작업을 위임한다.
        Executor executor = getExecutor();
        if (dispatch && executor != null) {
            executor.execute(sc);
        } else {
            sc.run();
        }
        
        return true;
    }
}
```

---
**Reference**
- https://ego2-1.tistory.com/30
- https://velog.io/@garden6/Tomcat-은-어떻게-동작할까-Spring-과의-연동을-중점으로-3
- https://github.com/apache/tomcat/blob/main/java/org/apache/tomcat/util/net/Acceptor.java
- https://px201226.github.io/tomcat/

