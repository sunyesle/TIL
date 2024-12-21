# Lock
`java.util.concurrent.locks` 패키지는 `synchronized`보다 더 유연하고 세밀한 동기화 제어를 제공한다.

## Lock API
잠금 작업을 제공한다.
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

항상 잠금을 해제해야 한다.<br>
일반적으로 `try-catch-finally` 문의 `finally`에서 `unlock` 메서드를 호출하여 잠금이 해제되도록 보장한다.
```java
Lock lock = ...; 
lock.lock();
try {
    // 공유 자원 접근 로직
} finally {
    lock.unlock();
}
```

## 종류
- `ReentrantLock` : 재진입이 가능한 lock. 가장 일반적인 배타적(Exclusive) lock
- `ReentrantReadWriteLock` : 읽기에는 공유적이고, 쓰기에는 배타적인 lock
- `StampedLock` : ReentrantReadWriteLock에 낙관적 읽기(Optimistic Reading) lock의 기능을 추가

---
**Reference**<br>
- https://www.baeldung.com/java-concurrent-locks
- https://ttl-blog.tistory.com/799

