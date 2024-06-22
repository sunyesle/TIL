# synchronized
Java에서 제공하는 동기화 메커니즘 중 하나인 `synchronized` 키워드에 대해 알아보자.

## 동기화란?
**공유 자원(Shared Resources)**<br>
다수의 스레드가 접근하고 수정할 수 있는 데이터나 자원

**경쟁 조건(Race Condition)**<br>
둘 이상의 스레드가 동시에 같은 데이터를 조작할 때 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황

**임계 영역(Critical Section)**<br>
경쟁 조건에서 공유 자원을 사용하게 되는 영역

**동기화(Synchronization)**<br>
여러 스레드가 공유하는 자원의 일관성을 유지하는 것

## 고유락(Intrinsic Lock)
고유락은 Java의 동기화 방법 중 하나로, 객체 단위로 동기화를 제어하는 데 사용된다.
Java의 모든 객체(인스턴스, 클래스)는 내부적으로 락을 가지고 있다.
`synchronized` 키워드를 사용하여 블록 단위로 락을 획득하고 해제할 수 있으며, 이를 통해 스레드 간의 동기화를 구현할 수 있다.

`synchronized` 키워드를 사용할 때, 해당 객체에 대한 암시적인 락이 걸린다.
`synchronized` 블록을 진입할 때 락의 획득이 일어나고, 블록을 벗어날 때 락의 해제가 일어난다.
JVM이 자동으로 락의 획득과 해제를 처리하기 때문에 프로그래머는 락을 명시적으로 관리할 필요가 없다.

## synchronized 사용 방법

**synchronized method**

클래스의 인스턴스 단위로 락을 건다.
```java
class SynchronizedTest {
    public synchronized void a() {
        //...
    }
}
```

**static synchronized method**

클래스 단위로 락을 건다.
```java
class SynchronizedTest {
    public static synchronized void a() {
        //...
    }
}
```

**synchronized block**

락을 거는 객체를 지정할 수 있다.
```java
class SynchronizedTest {
    Object obj = new Object();
    
    public void a() {
        synchronized (this) { // 현재 객체에 대한 락 획등
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
- https://hbase.tistory.com/311
