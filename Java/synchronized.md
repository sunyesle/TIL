# synchronized
Java에서 제공하는 _동기화_ 메커니즘 중 하나인 `synchronized` 키워드에 대해 알아보자.

> **동기화(Synchronization)**: 여러 스레드가 공유하는 자원의 일관성을 유지하는 것

## 고유락(Intrinsic Lock)
Java의 **모든 객체**(**인스턴스, 클래스**)는 내부적으로 락을 가지고 있다. 이를 모니터, 모니터 락, 뮤텍스라고도 한다.<br>
고유락은 Java의 동기화 방법 중 하나로, **객체 단위로 동기화를 제어**하는 데 사용된다.

> ### Java 객체에서 Lock은 어디에 있을까?
> 모든 자바 객체는 헤더라는 메타데이터를 포함하고 있다. 객체 헤더에는 JVM이 객체를 관리하는 데에 필요한 정보가 저장된다. 이중 Mark Word 부분에 락 정보를 담고 있다.<br>
> int, long 같은 원시 타입은 객체가 아니기 때문에 `synchronized` 키워드를 사용할 수 없다.
> 
> <img width="600" alt="Image" src="https://github.com/user-attachments/assets/29f70e0e-768a-4248-a1f2-e420078b378b" />

## synchronized 키워드
`synchronized` 키워드를 사용하여 블록 단위로 락을 획득하고 해제할 수 있으며, 이를 통해 스레드 간의 동기화를 구현할 수 있다.

`synchronized` 키워드를 사용할 때, 해당 객체에 대한 암시적인 락이 걸린다.<br>
`synchronized` 블록을 진입할 때 락의 획득이 일어나고, 블록을 벗어날 때 락의 해제가 일어난다.

JVM이 자동으로 락의 획득과 해제를 처리하기 때문에 프로그래머는 락을 명시적으로 관리할 필요가 없다.

### 특징
- **상호 배제(Mutual exclusion)**: 한 번에 하나의 스레드만 `synchronized` 블록에 진입할 수 있다.
- **가시성 보장(Visibility)**: `synchronized` 블록에 들어가면 스레드에 표시되는 모든 변수값이 새로 고쳐지고, 스레드가 `synchronized` 블록을 종료하면 변수의 모든 변경 사항이 메인 메모리에 커밋된다.
- **재진입 가능(Reentrancy)**: 한 스레드가 모니터 객체의 락을 획득하면, 같은 모니터 객체에 대해 선언된 모든 `synchronized` 블록으로의 진입권을 갖는다.

### 주의점
- **오버헤드**: 락의 획득과 해제에 따른 오버헤드가 발생한다.
    - 짧은 루프 내에서 동기화 블록에 여러 번 들어가고 나가는 경우 주의해야한다.
    - 필요 이상으로 큰 동기화 블록은 두지 않는 것이 좋다.
- **데드락 가능성**: 여러 락을 사용할 때 데드락이 발생할 수 있다.
- **중첩 모니터 잠금**: 여러개의 락을 사용하는 경우 코드를 신중하게 설계하지 않으면 중첩 모니터 잠금(Nested Monitor Lockout)이 발생할 수 있다.

### 제한 사항
간단한 동기화에는 synchronized를 사용하는 것이 더 효율적일 수 있으나 세밀한 제어가 힘들다.<br>
더 세밀한 동시성 제어가 필요한 경우 `java.util.concurrent` 패키지를 활용하자.

### 사용 방법
**synchronized method**<br>
클래스의 인스턴스 단위로 락을 건다.
```java
class SynchronizedTest {
    public synchronized void a() {
        //...
    }
}
```

**static synchronized method**<br>
클래스 단위로 락을 건다.
```java
class SynchronizedTest {
    public static synchronized void a() {
        //...
    }
}
```

**synchronized block**<br>
락을 거는 객체를 지정할 수 있다.
```java
class SynchronizedTest {
    Object obj = new Object();
    
    public void a() {
        synchronized (this) { // 현재 객체에 대한 락 획득
            //...
        }
    }

    public void b() {
        synchronized (SynchronizedTest.class) { // SynchronizedTest 클래스에 대한 락 획득
            //...
        }
    }
    
    public void c() {
        synchronized (obj) { // obj 객체에 대한 락 획득
            //...
        }
    }
}
```

---
**reference**
- https://jgrammer.tistory.com/entry/Java-%ED%98%BC%EB%8F%99%EB%90%98%EB%8A%94-synchronized-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%A0%95%EB%A6%AC
- https://developer-rabbit.tistory.com/entry/Java-%EA%B3%A0%EC%9C%A0-%EB%9D%BD-Intrinsic-Lock
- http://happinessoncode.com/2017/10/04/java-intrinsic-lock/
- https://code-boki.tistory.com/214
- https://jenkov.com/tutorials/java-concurrency/synchronized.html
