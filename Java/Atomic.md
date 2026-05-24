# Atomic 클래스
Java는 `java.util.concurrent.atomic` 패키지를 통해 **CAS** 연산을 지원하는 여러 Atomic 클래스를 제공한다.

> **CAS(Compare-And-Swap)**
> 
> 데이터의 조회-비교-교환의 과정을 하드웨어 수준에서 단일 명령어로 처리하여 원자성을 보장한다. 이를 통해 멀티스레드 환경에서 락(Lock) 없이도 데이터 일관성을 유지할 수 있다.

## 내부 동작 과정
`AtomicInteger`가 어떻게 락 없이 원자성을 보장하는지 내부 구현을 살펴보자.

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final Unsafe U = Unsafe.getUnsafe();
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");

    private volatile int value;

    public final int incrementAndGet() {
        return U.getAndAddInt(this, VALUE, 1) + 1;
    }
}
```
`incrementAndGet()`는 변수의 값을 1 올리는 메서드이다. 내부적으로 `Unsafe` 클래스의 `getAndAddInt()` 메서드를 호출한다.

```java
public final class Unsafe {
    @IntrinsicCandidate
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
    }
}
```
핵심 로직은 `getAndAddInt()` 메서드에 담겨 있다. `getIntVolatile()` 메서드를 통해 현재 메모리에 저장된 값을 읽어오고, `weakCompareAndSetInt()`를 호출하여 CAS 연산을 수행한다. 이를 CAS 연산이 성공할 때까지 반복한다.

```java
@IntrinsicCandidate
public final boolean weakCompareAndSetInt(Object o, long offset,
                                          int expected,
                                          int x) {
    return compareAndSetInt(o, offset, expected, x);
}
```
```java
/**
 * Atomically updates Java variable to {@code x} if it is currently
 * holding {@code expected}.
 *
 * <p>This operation has memory semantics of a {@code volatile} read
 * and write.  Corresponds to C11 atomic_compare_exchange_strong.
 *
 * @return {@code true} if successful
 */
@IntrinsicCandidate
public final native boolean compareAndSetInt(Object o, long offset,
                                             int expected,
                                             int x);
```
`weakCompareAndSetInt()`는 하드웨어 수준에서 CAS 연산을 실행하기 위해 native 메서드인 `compareAndSetInt()`를 호출한다.

## 예시
```java
public class Main {
    private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicCount = new AtomicInteger();

        Thread threadA = new Thread(() -> {
            IntStream.range(0, 100000).forEach(i -> {
                count++;
                atomicCount.incrementAndGet();
            });
        });

        Thread threadB = new Thread(() -> {
            IntStream.range(0, 100000).forEach(i -> {
                count++;
                atomicCount.incrementAndGet();
            });
        });

        threadA.start();
        threadB.start();

        Thread.sleep(1000);
        System.out.println("AtomicInteger: " + atomicCount.get());
        System.out.println("int: " + count);
    }
}
```
> 결과
```log
AtomicInteger: 200000
int: 190446
```

---
**Reference**<br>
- https://jenkov.com/tutorials/java-concurrency/compare-and-swap.html
- https://velog.io/@ksh98/CAS와-Atomic-타입
