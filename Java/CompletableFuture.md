# CompletableFuture

## Future의 한계와 단점
`Future`는 Java 5에 비동기 연산을 위해 추가되었지만 몇 가지 문제점을 가지고 있다.
- 외부에서 완료시킬 수 없고, get의 타임아웃으로만 완료가 가능하다.
- 블로킹 코드(get)을 통해서만 이후의 결과를 처리할 수 있다.
- 여러 작업을 결합하기 힘들다.
- 비동기 처리 중에 발생하는 예외를 처리하기 힘들다.

이러한 `Future` 인터페이스의 문제를 개선한 `CompletableFuture`가 Java 8에 추가되었다.

## CompletableFuture
`CompletableFuture`는 `Future`와 `CompletionStage` 인터페이스를 구현하고 있다.
`CompletionStage`는 작업을 중첩시키거나 완료 후 콜백을 위해 추가되었다.

`CompletableFuture`의 기능은 크게 다음과 같이 나뉜다.
- 비동기 작업 실행
- 작업 콜백
- 작업 조합
- 예외 처리
- 결과 반환

### 비동기 작업 실행
`runAcync`와 `supplyAsync`는 기본적으로 `ForkJoinPool.commonPool()`을 사용해 작업을 실행할 스레드를 얻는다.
- `runAsync`: 결과값이 없는 작업을 비동기로 실행한다.
- `supplyAsync`: 결과값이 있는 작업을 비동기로 실행한다.
```java
@Test
void testRunAsync() {
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> System.out.println("hello"));
    future.join();
}
```
```java
@Test
void testSupplyAsync() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello");
    System.out.println(future.join());
}
```

### 작업 콜백
메서드 체이닝을 통해 다양한 함수형 인터페이스들을 콜백으로 등록할 수 있다.
- `thenApply`: 결과값을 받아서 다른 값을 반환한다. (`Function`)
- `thenAccept`: 결과값을 받아서 처리하고 값을 반환하지 않는다. (`Consumer`)
- `thenRun`: 결과값을 받지 않고 작업 실행한다. (`Runnable`)
- `whenComplete`: 결과값 또는 예외를 받지만, 결과를 변경하지 않고 그대로 다음 단계로 넘긴다. 주로 로그 출력이나 리소스 정리와 같은 부수적인 작업에 사용된다. (`BiConsumer`)
```java
@Test
void testThenApply() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello")
            .thenApply(String::toUpperCase);
    System.out.println(future.join());
}
```
```java
@Test
void testThenAccept() {
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> "hello")
            .thenAccept(System.out::println);
    future.join();
}
```
```java
@Test
void testThenRun() {
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> "hello")
            .thenRun(() -> System.out.println("Hi"));
    future.join();
}
```
```java
@Test
void testWhenComplete() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello")
            .whenComplete((result, e) -> {
                if (e == null) {
                    System.out.println("Log: " + result);
                }
            });

    System.out.println(future.join());
}
```

### 작업 조합
- `thenCompose`: 두 작업이 이어서 실행되도록 조합한다.
- `thenCombine`: 두 작업을 독립적으로 실행하고, 둘 다 완료되었을 때 콜백을 실행한다.
- `allOf`: 작업을 동시에 실행하고, 모든 작업 결과에 콜백을 실행한다.
- `anyOf`: 작업을 동시에 실행하고, 가장 먼저 완료된 작업 결과에 콜백을 실행한다.
```java
@Test
void testThenCompose() {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> "hello");

    CompletableFuture<String> future = hello.thenCompose(message -> CompletableFuture.supplyAsync(() -> "send: " + message));
    System.out.println(future.join());
}
```
```java
@Test
void testThenCombine() {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> "hello");
    CompletableFuture<String> hi = CompletableFuture.supplyAsync(() -> "hi");

    CompletableFuture<String> future = hello.thenCombine(hi, (a, b) -> a + " " + b);
    System.out.println(future.join());
}
```
```java
@Test
void testAllOf() {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> "hello");
    CompletableFuture<String> hi = CompletableFuture.supplyAsync(() -> "hi");

    List<CompletableFuture<String>> futures = List.of(hello, hi);

    CompletableFuture<List<String>> result = CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                    .map(CompletableFuture::join)
                    .toList());

    System.out.println(result.join());
}
```
```java
@Test
void testAnyOf() {
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "hello";
    });
    CompletableFuture<String> hi = CompletableFuture.supplyAsync(() -> "hi");

    CompletableFuture<String> result = CompletableFuture.anyOf(hello, hi)
            .thenApply(Object::toString);

    System.out.println(result.join());
}
```

### 예외 처리
- `exceptionally`: 발생한 에러를 받아서 예외를 처리한다. (`Function`)
- `handle`: (결과값, 에러)를 받아서 처리한다. (`BiFunction`)
```java
@ParameterizedTest
@ValueSource(booleans = {true, false})
void testExceptionally(boolean doThrow) {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        if (doThrow) {
            throw new RuntimeException("exception");
        }
        return "success";
    }).exceptionally(Throwable::getMessage);

    System.out.println(future.join());
}
```
```java
@ParameterizedTest
@ValueSource(booleans = {true, false})
void testHandle(boolean doThrow) {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        if (doThrow) {
            throw new RuntimeException("exception");
        }
        return "success";
    }).handle((result, e) -> e == null ? result : e.getMessage());

    System.out.println(future.join());
}
```
> 실행 결과(두 예제 모두 동일하다)
```log
doThrow = true 일때: java.lang.RuntimeException: exception
doThrow = false 일떄: success
```

### 결과 반환
두 메서드 모두 작업이 완료될 때까지 대기(Blocking)한 후 결과를 반환하지만, 예외 처리 방식이 다르다.
- `join()`: 작업 실행 중 예외 발생 시 Unchecked Exception을 던진다.
- `get()`: 작업 실행 중 예외 발생 시 Checked Exception을 던진다. 예외를 명시적으로 처리해야 된다.
```java
@Test
public void testJoin() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello");
    System.out.println(future.join());

    CompletableFuture<String> exceptionFuture = CompletableFuture.failedFuture(new RuntimeException("exception"));
    assertThrows(CompletionException.class, exceptionFuture::join);
}
```
```java
@Test
public void testGet() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello");
    try {
        System.out.println(future.get());
    } catch (ExecutionException | InterruptedException e) {
        throw new RuntimeException(e);
    }

    CompletableFuture<String> exceptionFuture = CompletableFuture.failedFuture(new RuntimeException("exception"));
    assertThrows(ExecutionException.class, exceptionFuture::get);
}
```

## Java 9 CompletableFuture 개선 사항
Java 9부터 `CompletableFuture`에 지연 및 시간 초과를 지원하는 메서드가 추가되었다.

### 시간 초과
- `orTimeout`: 지정된 시간 내에 완료되지 않으면 `TimeoutException`을 발생시켜 `CompletableFuture`를 예외적으로 완료한다.
- `completeOnTimeout`: 지정된 시간 내에 완료되지 않으면 기본값을 반환한다.
```java
@Test
void testTimeout() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                return "hello";
            })
            .orTimeout(1, TimeUnit.SECONDS);

    CompletionException exception = assertThrows(CompletionException.class, future::join);
    exception.printStackTrace();
}
```
```java
@Test
void testComplateOnTimeout() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                return "hello";
            })
            .completeOnTimeout("timeout", 1, TimeUnit.SECONDS);

    System.out.println(future.join());
}
```

### 실행 지연
- `delayedExecutor`: 지정된 지연 후에 작업을 실행하는 `Executor`를 반환한다.
```java
@Test
void testDelayedExecutor() {
    long start = System.nanoTime();
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello", CompletableFuture.delayedExecutor(500, TimeUnit.MILLISECONDS));

    future.join();
    System.out.println("duration: " + TimeUnit.MILLISECONDS.convert(System.nanoTime() - start, TimeUnit.NANOSECONDS) + "ms");
}
```

---
**Reference**<br>
- https://devidea.tistory.com/59
- https://11st-tech.github.io/2024/01/04/completablefuture/
- https://www.baeldung.com/java-completablefuture
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html
- https://www.baeldung.com/java-completablefuture-join-vs-get
