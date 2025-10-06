# 톰캣 NIO Connector 동작 방식

## Acceptor
**Acceptor**는 클라이언트의 새로운 TCP 연결을 수락하는 전용 스레드로 동작한다.<br>
이 스레드는 내부적으로 다음과 같은 절차를 수행한다.

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

Acceptor가 호출한 `setSocketOptions()` 메서드에서는 소켓 채널을 논블로킹으로 전환하고,
소켓을 `SocketChannel -> NioChannel -> NioSocketWrapper` 형태로 캡슐화한 뒤 `PollerEventQueue`에 등록한다.
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
        // Poller에 등록
        poller.register(socketWrapper);
        return true;
    }
}
```

## Poller
**Poller**는 NIO 기반의 논블로킹 I/O 이벤트 감지를 담당하는 스레드로,
내부적으로 `Selector`를 이용하여 등록된 소켓의 읽기/쓰기 이벤트를 감지한다.

소켓 등록은 단순히 `Selector`에 바로 등록하는 방식이 아니라,
`PollerEvent`라는 작업 단위를 생성하여, Poller 내부 큐(`PollerEventQueue`)에 등록하는 방식으로 진행된다.

`poller.register()`가 호출되면, 내부적으로 다음과 같은 작업이 수행된다.

1. 소켓의 관심 이벤트를 `OP_READ`로 설정한다.
2. `NioSocketWrapper`와 이벤트 타입 `OP_REGISTER`을 묶어 `PollerEvent` 객체를 생성한다.
3. 생성된 `PollerEvent` 객체를 `PollerEventQueue`에 추가한다.

```java
public class NioEndpoint extends AbstractNetworkChannelEndpoint<NioChannel,SocketChannel> {

    public class Poller implements Runnable {
        private final Selector selector;
        private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

        public void register(final NioSocketWrapper socketWrapper) {
            // 소켓 채널이 관심 있는 이벤트를 OP_READ로 설정한다.
            socketWrapper.interestOps(SelectionKey.OP_READ);
            // NioSocketWrapper를 이벤트 타입 OP_REGISTER과 함께 PollerEvent로 캡슐화한다.
            PollerEvent pollerEvent = createPollerEvent(socketWrapper, OP_REGISTER);
            // 생성된 이벤트를 PollerEventQueue에 추가한다.
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

Poller 스레드는 내부 루프에서 다음과 같은 절차를 수행한다.

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
