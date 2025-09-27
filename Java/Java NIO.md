# Java NIO
Java NIO(New Input/Output)는 Java의 기존 I/O API를 대체하기 위해 Java 1.4 버전에서 도입된 I/O API이다.

## 특징
### 채널과 버퍼
Java NIO에서는 채널과 버퍼를 사용한다.
데이터는 항상 채널에서 버퍼로 읽히거나, 버퍼에서 채널로 쓰여진다.

### 논블로킹(non-blocking) I/O
Java NIO에서는 논블로킹 IO를 사용할 수 있다.
스레드가 버퍼로 데이터를 읽어달라고 요청하면 채널이 버퍼에 데이터를 넣는 동안 해당 스레드는 다른 작업을 수행할 수 있다.
이후 채널이 버퍼에 데이터를 채워 넣고 나면 스레드는 해당 버퍼를 이용해 계속 처리를 진행할 수 있다.
이는 데이터를 쓸 때도 동일하다.

### 셀렉터
Java NIO에는 셀렉터라는 개념이 있다.
셀렉터는 여러 채널에서 이벤트(연결 생성, 데이터 도착 등)를 모니터링 할 수 있는 객체이다.
이를 통해 단일 스레드가 여러 채널에서 데이터를 모니터링 할 수 있다.

## 구성 요소
Java NIO의 주요 구성 요소는 다음과 같다.
- **Channel**
- **Buffer**
- **Selector**

### Channel
NIO의 모든 IO는 채널로 시작한다.

Chennel은 Stream과 유사하지만 몇 가지 차이점이 있다.
- 채널은 읽기와 쓰기 모두 가능하다. 스트림은 일반적으로 단방향이다.
- 채널은 비동기적으로 읽고 쓸 수 있다.
- 채널은 항상 버퍼를 통해 읽고 쓴다.

대표적인 채널 구현체는 다음과 같다. 
- `FileChannel`: 파일에 데이터를 읽고 쓴다.
- `ServerSocketChannel`: 클라이언트의 TCP 연결 요청을 수신(listening)할 수 있으며, 각 수신 연결에 대해 `SocketChannel`이 생성된다.
- `SocketChannel`: TCP를 통해 네트워크에서 데이터를 읽고 쓴다.
- `DatagramChannel`: UDP를 통해 네트워크에서 데이터를 읽고 쓴다.

### Buffer
버퍼는 채널과 상호작용 할 때 사용된다. 데이터는 채널에서 버퍼로 읽히고, 버퍼에서 채널로 쓰여진다.

버퍼는 다음과 같은 단계로 사용한다.
1. 버퍼에 데이터를 쓴다.
2. `buffer.filp()` 메서드를 호출한다. (버퍼를 쓰기 모드에서 읽기 모드로 전환)
3. 버퍼에서 데이터를 읽는다.
4. `buffer.clear()` 또는 `buffer.compact()` 메서드를 호출한다. (버퍼를 다시 쓸 수 있도록 함)

### Selector
셀렉터는 하나 이상의 채널 인스턴스를 검사하고 어떤 채널이 읽기 또는 쓰기 작업을 수행할 준비가 되었는지 판단하는 구성 요소다.
이를 통해 단일 스레드가 여러 채널을 관리할 수 있다.

셀렉터에 등록할 수 있는 이벤트는 다음 4가지이다.
- `OP_ACCEPT`: 클라이언트가 서버에 접속했을 때 발생하는 이벤트
- `OP_CONNECT`: 서버가 클라이언트의 접속을 허락했을 때 발생하는 이벤트
- `OP_READ`: 서버가 클라이언트의 요청을 읽을 수 있을 때 발생하는 이벤트
- `OP_WRITE`: 서버가 클라이언트의 응답을 쓸 수 있을 때 발생하는 이벤트

구현체마다 등록될 수 있는 이벤트는 다음과 같다.

| 채널 구현체                | 등록할 수 있는 이벤트                        |
|-----------------------|-------------------------------------|
| `ServerSocketChannel` | `OP_ACCEPT`                         |
| `SocketChannel`       | `OP_CONNECT`, `OP_READ`, `OP_WRITE` |
| `DatagramChannel`     | `OP_READ`, `OP_WRITE`               |


---
**Reference**<br>
- https://jenkov.com/tutorials/java-nio/overview.html
- https://engineering.linecorp.com/ko/blog/do-not-block-the-event-loop-part2
- https://brewagebear.github.io/fundamental-nio-and-io-models/
