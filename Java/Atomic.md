# Atomic
Java는 `java.util.concurrent.atomic` 패키지를 통해 CAS 연산을 지원하는 여러 Atomic 클래스를 제공한다.

## 내부 동작 과정
`AtomicInteger` 클래스의 일부이다.
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
`AtomicInteger` 변수의 값을 1 올리는 메서드 `incrementAndGet`이 있다.<br>
이 메서드는 내부적으로 `Unsafe` 클래스의 `getAndAddInt` 메서드를 호출한다.

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
`getIntVolatile` 메서드를 통해 메모리의 값을 읽어오고,<br>
`weakCompareAndSetInt` 메서드 내부에서 CAS 연산을 수행한다.<br>
현재값이 기댓값과 동일하면 메모리에 저장된 값을 교체하고 true를 반환하여 while문을 빠져나오게 된다.

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
최종적으로 native 메서드를 호출한다.

## 예제
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
- https://velog.io/@ksh98/CAS%EC%99%80-Atomic-%ED%83%80%EC%9E%85
