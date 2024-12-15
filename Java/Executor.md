# Executor

## 관련 개념
- `Runnable`: 결과를 반환하지 않는 작업
- `Callable`: 결과를 반환하는 작업
- `Future`: 미래에 완료된 Callable의 실행 결과를 받기 위해 사용된다.

## Executor 인터페이스 계층 구조
![Executor](https://github.com/user-attachments/assets/52738237-0548-49e5-b19c-168890e06a68)

- `Executor`: 작업 실행을 위한 인터페이스
- `ExecutorService`: 작업 등록과 실행을 위한 인터페이스
- `ScheduledExecutorService`: 특정 시간 이후 또는 주기적으로 작업을 실행할 수 있는 ExecutorService
- `Executors`: ExecutorService와 ScheduledExecutorService의 팩토리의 메서드 제공

## ExecutorService
`ExecutorService` 인터페이스는 `Executor`를 상속받는 인터페이스로 작업 실행뿐만 아니라 작업 등록의 책임도 갖는다.

대표적인 구현체로 `ThreadPoolExecutor`가 있다.
`ThreadPoolExecutor`는 내부의 `BlockingQueue`에 작업을 등록해둔다.
각각의 스레드는 작업을 할당받아 처리하고, 사용할 수 있는 스레드가 없다면 작업은 큐에서 대기하게 된다.

`ExecutorService`가 제공하는 퍼블릭 메서드는 크게 두 가지로 분류할 수 있다.
- 라이프사이클 관리를 위한 기능
- 비동기 작업을 위한 기능

### 라이프사이클 관리를 위한 기능
- `shutdown`: 새로운 작업을 더 이상 받아들이지 않는다. 호출 전에 제출된 작업은 실행이 끝나고 종료된다.
- `shutdownNow`: shutdown 기능에 더해 제출된 작업을 인터럽트 시킨다. 실행을 위해 대기 중인 작업 목록을 반환한다.
- `isShutdown`: `Executor`의 shutdown 여부를 반환한다.
- `isTerminated`: shutdown 실행 후 모든 작업의 종료 여부를 반환한다.
- `awaitTermination`: shutdown 실행 후 지정한 시간 동안 모든 작업일 종료될 때까지 대기한다. 지정한 시간 내에 모든 작업이 종료되었는지 여부를 반환한다.

```java
@Test
void testShutdown() {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    Runnable runnable = () -> System.out.println("Thread: " + Thread.currentThread().getName());
    executorService.execute(runnable);

    executorService.shutdown();

    // shutdown 호출 이후에는 새로운 작업을 받을 수 없다. 에러 발생
    assertThrowsExactly(RejectedExecutionException.class, () -> executorService.execute(runnable));
}
```

### 비동기 작업을 위한 기능
- `submit`: 작업을 제출하고, 작업의 상태와 결과를 포함하는 `Future`를 반환한다. `Future`의 get 메서드를 호출하면 작업이 끝날 때까지 블로킹된다. 성공적으로 작업이 완료된 후 결과를 얻을 수 있다.
- `invokeAll`: 모든 결과가 나올 때까지 대기하는 블로킹 방식의 요청이다. 주어진 작업을 실행하고 전부 완료되면 `List<Future>`를 반환한다.
- `invokeAny`: 가장 빨리 실행된 결과가 나올 때까지 대기하는 블로킹 방식의 요청이다. 주어진 작업을 실행하고 가장 빨리 완료된 작업의 `Future`를 반환한다.
```java
@Test
void testInvokeAll() throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    long start = System.nanoTime();

    Callable<String> hello = () -> {
        Thread.sleep(1000L);
        String result = "Hello";
        System.out.println("result: " + result);
        return result;
    };
    Callable<String> world = () -> {
        Thread.sleep(2000L);
        String result = "World";
        System.out.println("result: " + result);
        return result;
    };

    List<Future<String>> futures = executorService.invokeAll(Arrays.asList(hello, world));
    for (Future<String> f : futures) {
        System.out.println(f.get());
    }

    System.out.println("duration: " + TimeUnit.SECONDS.convert(System.nanoTime() - start, TimeUnit.NANOSECONDS));
    executorService.shutdown();
}
// duration: 2
```
```java
@Test
void testInvokAny() throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    long start = System.nanoTime();

    Callable<String> hello = () -> {
        Thread.sleep(1000L);
        String result = "Hello";
        System.out.println("result: " + result);
        return result;
    };
    Callable<String> world = () -> {
        Thread.sleep(2000L);
        String result = "World";
        System.out.println("result: " + result);
        return result;
    };

    String future = executorService.invokeAny(Arrays.asList(hello, world));
    System.out.println(future);

    System.out.println("duration: " + TimeUnit.SECONDS.convert(System.nanoTime() - start, TimeUnit.NANOSECONDS));
    executorService.shutdown();
}
// duration: 1
```

## ScheduledExecutorService
`ExecutorService`를 상속받는 인터페이스로 특정 시간 이후 또는 주기적으로 작업을 실행시키는 메서드가 추가되었다.
- `schedule`: 지정한 지연시간 이후 작업을 실행시킨다.
- `scheduleAtFixedRate`: 지정한 지연시간 이후, 고정된 간격으로 반복적으로 작업을 실행시킨다. 어떤 이유로 실행이 지연되는 경우 뒤처진 시간을 따라잡기 위해 두 개 이상의 작업이 연속해서 실행된다.
- `scheduleWithFixedDelay`: 지정한 지연시간 이후, 고정된 지연으로 반복적으로 작업을 실행시킨다.

```java
@Test
void testExecutors() throws InterruptedException {
    CountDownLatch lock = new CountDownLatch(1);
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

    ScheduledFuture<?> future = executor.scheduleAtFixedRate(() -> {
        System.out.println("start task: " + new Date());
    }, 0, 1000, TimeUnit.MILLISECONDS);

    executor.schedule(() -> {
        System.out.println("start slow task: " + new Date());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("end slow task: " + new Date());
    }, 1000, TimeUnit.MILLISECONDS);

    lock.await(7000, TimeUnit.MILLISECONDS);
    future.cancel(true);
    executor.shutdown();
}
```
slow task 실행으로 인해 뒤처진 시간을 따라잡기 위해 연속으로 실행되는 것을 확인할 수 있다.
```log
start task: Sun Dec 15 23:15:29 KST 2024
start task: Sun Dec 15 23:15:30 KST 2024
start slow task: Sun Dec 15 23:15:30 KST 2024
end slow task: Sun Dec 15 23:15:33 KST 2024
start task: Sun Dec 15 23:15:33 KST 2024
start task: Sun Dec 15 23:15:33 KST 2024
start task: Sun Dec 15 23:15:33 KST 2024
start task: Sun Dec 15 23:15:34 KST 2024
start task: Sun Dec 15 23:15:35 KST 2024
```

## Executors
스레드 풀을 손쉽게 생성할 수 있는 팩토리 메서드를 제공한다.
- `newFixedThreadPool`: 고정된 크기의 스레드 풀을 생성한다.
- `newCachedThreadPool`: 필요에 따라 새 스레드를 만드는 스레드 풀을 생성한다.
- `newScheduledThreadPool`: 일정 시간 뒤 혹은 주기적으로 실행되어야 하는 작업을 위한, 고정된 크기의 스레드 풀을 생성한다.
- `newSingleThreadExecutor`, `newSingleThreadScheduledExecutor`: 1개의 스레드만을 갖는 스레드 풀을 생성한다.

---
**Reference**<br>
- https://mangkyu.tistory.com/259
- https://yangbox.tistory.com/28
- https://brunch.co.kr/@mystoryg/37
- https://jizard.tistory.com/569
