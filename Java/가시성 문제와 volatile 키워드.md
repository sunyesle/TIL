# 가시성 문제와 volatile 키워드

동시성 프로그래밍에서는 CPU 캐시와 메인 메모리(RAM)의 동작 방식으로 인해 다수의 스레드가 공유 자원에 접근할 때 문제가 발생할 수 있다.

그 중 가시성 문제에 대해 알아보려고 한다.

## 가시성 문제
가시성 문제란 한 스레드에서 공유 자원을 수정한 결과가 다른 스레드에서는 보이지 않는 문제를 말한다.<br>
다음 코드를 살펴보자.
```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread threadA = new Thread(() -> {
            while (!stopRequested) {
            }
            System.out.println("threadA stopped");
        });

        Thread threadB = new Thread(() -> {
            stopRequested = true;
            System.out.println("threadB: stopRequested = true");
        });

        threadA.start();
        Thread.sleep(1000); // 1초의 딜레이
        threadB.start();
    }
}
```
`threadB`가 1초 후 `stopRequested`를 `true`로 설정하면 `threadA`가 반복문을 빠져나올 것으로 보인다.<br>
하지만 실행시켜 보면 반복문을 오랜 시간 빠져나오지 못하거나, 영원히 빠져나오지 못할 수도 있다.

다음 그림을 살펴보자.

<img width="500" alt="visibility_problem_1" src="https://github.com/user-attachments/assets/be0e45c4-38e0-476a-95a5-4e7cc585b4af" />

CPU1에서 수행된 스레드를 `threadA`, CPU2에서 수행된 스레드를 `threadB` 하자.<br>
`threadB`는 캐시 메모리와 메인 메모리에 공유 변수인 `stopRequested` 쓰기 작업을 완료했으나,<br>
`threadA`는 계속 자신의 캐시 메모리를 참조하고 있기 때문에 무한루프를 수행하게 된다.

즉, `threadB`가 수정한 값을 `threadA`가 언제 보게 될지 보장할 수 없다. 이러한 문제점을 **가시성 문제**라고 한다.

## volatile 키워드
이 문제를 해결하기 위해서는 `stopRequested` 변수를 `volatile`로 선언하면 된다.<br>
`volatile`은 변수의 읽기와 쓰기를 메인 메모리에 수행하도록 보장하는 키워드로, 변수의 변경 사항이 모든 스레드에 즉시 가시적이도록 보장한다.

<img width="500" alt="visibility_problem_2" src="https://github.com/user-attachments/assets/7676fc75-790c-408e-8688-80b818ef80b3" />

위 코드에서 stopRequested 변수에 volatile를 추가한다.
```java
private static volatile boolean stopRequested;
```
코드를 실행시켜보면 1초 후 프로그램이 종료되는 것을 확인할 수 있다.

---
**Reference**<br>
- https://jchong00.github.io/2019-09-05-concurrent-java-visibility/
- https://badcandy.github.io/2019/01/14/concurrency-02/
