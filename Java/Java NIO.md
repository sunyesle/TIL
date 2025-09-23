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
- Channel
- Buffer
- Selector

---
**Reference**<br>
- https://jenkov.com/tutorials/java-nio/overview.html
