# CAS 알고리즘과 Atomic 클래스

## CAS(Compare and Swap)
동시성 프로그래밍에서 사용하는 _락-프리(lock-free)_ 알고리즘 중 하나이다.

### CAS 작동 원리
다음 세 가지 값을 사용하여 연산을 수행한다.
- **메모리 주소**: 변경하고자 하는 메모리 위치
- **기댓값**: 해당 메모리 주소에 저장되어 있어야 하는 예상값
- **새로운 값**: 메모리 주소에 저장하고자 하는 값

CAS 연산은 다음과 같이 동작한다.
- CPU가 메모리 주소에 저장된 값과 기댓값이 일치하는지 비교한다.
- 일치하는 경우 해당 메모리 위치에 새로운 값을 원자적으로 교환한다.
- 일치하지 않는 경우 처음으로 돌아가 성공할 때까지 루프를 돌며 재시도한다.

#### 예시
다음과 같은 상황을 가정해 보자.

<img width="500" alt="CAS_1" src="https://github.com/user-attachments/assets/4d587a4c-416e-4c17-9968-bd6f305c5a04" />

두 개의 스레드가 동시에 `balance`에 접근해서 값을 변경하려고 한다.

<br>

<img width="500" alt="CAS_2" src="https://github.com/user-attachments/assets/0ed2549f-5c80-495b-9818-fc37034e16c4" />

`Thread1`이 `balance`에 접근해서 메모리값과 기댓값을 비교한다.<br>
기댓값 1000, 메모리 값 1000으로 일치하므로 `balance` 값을 900으로 변경한다.

이후 `Thread2`가 `balance`에 접근해서 메모리값과 기댓값을 비교한다.<br>
기댓값 1000, 메모리값 900으로 일치하지 않으므로 값을 변경하지 않는다.

### 특징
락을 사용하지 않기 때문에 락으로 인한 성능저하가 없으며 데드락 위험도 없다.<br>
하지만 충돌이 잦을 경우 반복적인 비교와 업데이트로 인해 CPU 자원을 소비할 수 있다.

다음과 같은 상황에 효과적이다.
- 충돌이 적은 경우
- 작업 수행 시간이 짧은 경우

## Atomic 클래스
Java는 `java.util.concurrent.atomic` 패키지를 통해 CAS 연산을 지원하는 여러 Atomic 클래스를 제공한다.

### 내부 동작 과정
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

### 예제
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
- https://velog.io/@appti/CASCompare-And-Set
- https://velog.io/@ksh98/CAS%EC%99%80-Atomic-%ED%83%80%EC%9E%85
- https://blog.arong.info/c%23/2022/07/05/C-Lock-Free,-Interlocked-%EC%82%AC%EC%9A%A9-(CAS).html
- https://f-lab.kr/insight/java-synchronization-and-cas
- https://dalichoi.tistory.com/entry/Lock-Free-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0CAS-Volatile-Java-Atomic-Variables
