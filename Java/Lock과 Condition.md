# java.util.concurrent.locks
`java.util.concurrent.locks` 패키지는 synchronized보다 더 유연하고 세밀한 동기화 제어를 제공한다.

## 인터페이스
**종류**<br>
- `Lock`
- `ReadWriteLock`
- `Condition`

## Lock
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
Lock 인터페이스가 갖고 있는 메서드는 다음과 같다.
- `lock()`: 락을 획득한다. 락이 사용중인 경우 획득할 때까지 스레드를 대기시킨다.(Blocking)
- `lockInterruptibly()`: lock()과 유사하나 락 획득 대기 중 인터럽트 발생 시 `InterruptedException` 예외를 던지고 락 획득을 포기한다.
- `tryLock()`: 락 획득 시도 후 획득 여부를 반환한다.
- `boolean tryLock(long time, TimeUnit unit)`: 지정한 시간 동안 락 획득 시도 후 획득 여부를 반환한다. 락 획득 대기 중 인터럽트 발생 시 InterruptedException 예외를 던지고 락 획득을 포기한다.
- `unlock()`: 락을 해제한다.
- `newCondition()`: 락과 결합하여 사용 가능한 `Condition` 객체를 생성 후 반환한다. `Condition` 객체는 스레드가 특정 조건을 기다리거나 신호를 받을 수 있도록 한다.

**주의 사항**<br>
자동적으로 락의 잠금과 해제가 관리되는 synchronized 블록과 달리 Lock은 수동으로 락을 잠그고 해제해야 한다.<br>
일반적으로 `try-catch-finally` 문의 `finally`에서 `unlock` 메서드를 호출하여 락이 해제되도록 보장한다.
```java
Lock lock = ...; 
lock.lock();
try {
    // 공유 자원 접근 로직
} finally {
    lock.unlock();
}
```

## ReadWriteLock
읽기/쓰기 잠금 획득 기능을 제공한다.
```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

## Condition
`Object` 모니터 메서드(`wait`, `notify`, `notfiyAll`)는 스레드의 종류를 구분하지 않고 공유 객체의 waiting-pool에 넣는다.<br>
`Condition`은 이러한 문제를 해결한다.

스레드의 종류에 따라 `Condition`을 생성하면, 별도의 waiting pool에서 따로 기다리도록 하여 선별적인 통지가 가능하다.<br>
`Condition`은 생성된 lock으로부터 `newCondition` 메서드를 호출하여 생성할 수 있다.

| Object                 | Condition                                    |
|------------------------|----------------------------------------------|
| void wait()            | void await() <br> void awaitUninterruptibly() |
| void wait(long timeout) | boolean await(long time, TimeUnit unit)      |
| void notify()          | void signal()                                |
| void notifyAll()       | void signalAll()                             |

## 구현체
**종류**<br>
- `ReentrantLock` : 재진입이 가능한 lock. 가장 일반적인 배타적(Exclusive) lock
- `ReentrantReadWriteLock` : 읽기에는 공유적이고, 쓰기에는 배타적인 lock
- `StampedLock` : ReentrantReadWriteLock에 낙관적 읽기(Optimistic Reading) lock의 기능을 추가

## ReentrantLock
### lock, unlock
명시적으로 락을 획득하고 해제한다. 락 해제가 보장되도록 finally 블록에서 `unlock` 메서드를 호출했다.
```java
public class SharedObject {
    ReentrantLock lock = new ReentrantLock();
    int count = 0;

    public void perform() {
        lock.lock(); // 락을 획득한다.
        try {
            // 임계 영역
            count++; 
        } finally {
            lock.unlock(); // 락을 해제한다.
        }
    }
}
```

### tryLock
락을 획득할 수 없는 스레드들이 무한정 대기에 빠지는 문제를 해결할 수 있다.
```java
public class SharedObject {
    ReentrantLock lock = new ReentrantLock();
    int count = 0;

    // 즉시 획득 시도 후 실패 시 넘어간다.
    public void performTryLock() {
        if (lock.tryLock()) {
            try {
                count++;
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println(Thread.currentThread().getName() + ": 다른 스레드가 락을 사용 중입니다.");
        }
    }

    // 지정한 시간 동안 획득 시도 후 실패 시 넘어간다.
    public void performTryLockTime() {
        try {
            if (lock.tryLock(1, TimeUnit.SECONDS)) {
                try {
                    count++;
                } finally {
                    lock.unlock();
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### newCondition
```java
public class SharedStack {
    Stack<String> stack = new Stack<>();
    int CAPACITY = 5;

    ReentrantLock lock = new ReentrantLock();
    Condition stackEmptyCondition = lock.newCondition(); // 스택이 비어 있을 때 대기하는 Condition
    Condition stackFullCondition = lock.newCondition(); // 스택이 가득 찼을 때 대기하는 Condition

    public void push(String item) {
        try {
            lock.lock();
            while (stack.size() == CAPACITY) {
                System.out.println(Thread.currentThread().getName() + "가 대기 중입니다. 스택이 가득 찼습니다.");
                stackFullCondition.await();
            }
            stack.push(item);
            stackEmptyCondition.signalAll();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }

    public String pop() {
        try {
            lock.lock();
            while (stack.size() == 0) {
                System.out.println(Thread.currentThread().getName() + "가 대기 중입니다. 스택이 비어 있습니다.");
                stackEmptyCondition.await();
            }
            String item = stack.pop();
            stackFullCondition.signalAll();
            return item;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }
}
```

## ReentrantReadWriteLock
- `ReentrantReadWriteLock.ReadLock`
  - 읽기 락은 여러 스레드가 동시에 획득 가능하다.
  - 모든 읽기 락이 해제될 때까지 쓰기 락은 획득 불가능하다.
  - `Conditon`을 지원하지 않는다.
- `ReentrantReadWriteLock.WriteLock`
  - 한 번에 하나의 스레드만 획득 가능하다.
  - 쓰기 락이 해제될 때까지 읽기 락 획득 불가능하다.
  - `Condition`을 지원한다.

**사용 기준**<br>
- 수정은 드물게 일어나고 조회가 빈번히 발생할 때 적합하다.
- 읽기 작업 시간이 너무 짧은 경우 `ReadWriteLock` 구현의 오버헤드가 증가하기 때문에 효율성이 떨어진다.

```java
public class SharedHashMap {
    Map<String, String> syncHashMap = new HashMap<>();
    ReadWriteLock lock = new ReentrantReadWriteLock();

    Lock writeLock = lock.writeLock();
    Lock readLock = lock.readLock();

    public void put(String key, String value) {
        try {
            writeLock.lock();
            syncHashMap.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }

    public String get(String key) {
        try {
            readLock.lock();
            return syncHashMap.get(key);
        } finally {
            readLock.unlock();
        }
    }
}
```

---
**Reference**<br>
- https://www.baeldung.com/java-concurrent-locks
- https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/package-summary.html
- https://ttl-blog.tistory.com/799
- https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Thread/java.util.concurrent.locks/ReentrantLock%EC%9D%B4%EB%9E%80%3F.md
- https://github.com/hyh1016/TIL/blob/main/java/ReentrantLock.md
- https://jaimemin.tistory.com/2411
