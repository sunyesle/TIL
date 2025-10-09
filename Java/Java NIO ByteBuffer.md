# Java NIO ByteBuffer

## ByteBuffer란
Java NIO에서 바이트 데이터를 효율적으로 입출력하기 위해 사용되는 클래스이다.
내부적으로 바이트 배열을 관리하며, 읽기/쓰기 작업을 편리하게 수행할 수 있는 메서드를 제공한다.

## Non-Direct Buffer vs Direct Buffer
### Non-Direct Buffer
non-direct 버퍼는 **JVM이 관리하는 힙 메모리** 공간을 이용하는 버퍼이다.

I/O 작업 수행 시 내부적으로 다음과 같이 동작한다.
1. 임시 direct 버퍼를 생성한다.
2. non-direct 버퍼의 데이터를 direct buffer로 복사한다.
3. direct buffer를 사용하여 I/O 작업을 수행한다.
4. 작업이 끝나면 임시 direct 버퍼는 GC된다.

이처럼 중간 복사 과정이 발생하므로, 대용량 또는 빈번한 I/O 작업에서 성능 저하가 생길 수 있다.

### Direct Buffer
direct 버퍼는 **운영체제의 커널 영역의 메모리** 공간을 이용하는 버퍼이다.

운영체제가 관리하는 메모리 영역에 직접 할당되기 때문에,
JVM의 메모리에서 운영체제의 메모리로 복사할 필요 없이 바로 I/O 작업을 수행할 수 있다.
이로인해 운영체제 시스템을 이용하는 입출력 (파일 I/O, 소켓 I/O)에서 성능 이점이 있다.

반면, non-direct 버퍼에 비해 메모리 할당과 해제 비용이 크며, JVM의 가비지 컬렉터가 관리하는 힙 메모리 영역 밖 위치하기 때문에 메모리 추적이 어렵고 관리 부담이 존재한다.

따라서, direct 버퍼는 운영체제 수준의 I/O 입출력 작업에 사용되며, 할당과 해제가 빈번하지 않는 경우에 사용하는 것이 좋다.
일반적으로 측정 가능한 퍼포먼스 이득이 있을 때만 사용하는 것이 좋다.

## ByteBuffer 생성
```java
// non-direct 버퍼를 생성한다.
ByteBuffer buffer = ByteBuffer.allocate(10);
```

```java
// direct 버퍼를 생성한다.
ByteBuffer buffer = ByteBuffer.allocateDirect(10);
```

```java
// 기존 바이트 배열을 랩핑하여 버퍼를 생성한다. 배열과 버퍼가 같은 메모리 공간을 공유하게 된다.
byte[] bytes = new byte[10];
ByteBuffer buffer = ByteBuffer.wrap(bytes);

// 또는
ByteBuffer buffer = ByteBuffer.wrap(bytes, 0, bytes.length);
```

### 속도 비교
`allcate()`와 `allcateDirect()` 메서드의 속도를 비교해보자.
```java
// allcate()
long start = System.currentTimeMillis();

for (int i = 0; i < 1000000; i++) {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
}

long end = System.currentTimeMillis();
System.out.println("실행 시간(ms) : " + (end - start)); // 실행 시간(ms) : 42
```

```java
// allcateDirect()
long start = System.currentTimeMillis();

for (int i = 0; i < 1000000; i++) {
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
}

long end = System.currentTimeMillis();
System.out.println("실행 시간(ms) : " + (end - start)); // 실행 시간(ms) : 782
```
`allcate()`에 비해 `allcateDirect()`는 15배 이상 속도 차이가 나는 것을 확인할 수 있다.

direct 버퍼는 생성 비용이 높기 때문에, ByteBuffer를 매번 생성하는 방식은 프로그램 성능에 영향을 줄 수 있다.<br>
따라서, 버퍼 풀(Buffer Pool)을 통해 재활용하는 것이 좋다.

---
**Reference**
- https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html
- https://junhyunny.github.io/java/jvm/direct-memory-in-java/
- https://px201226.github.io/java-nio-buffer/
- http://eincs.com/2009/08/java-nio-bytebuffer-channel-file/

