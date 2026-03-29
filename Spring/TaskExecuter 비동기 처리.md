# TaskExecuter 비동기 처리
스프링 부트는 `@Async` 어노테이션을 이용한 비동기 처리를 지원한다.

## application.yml
```yml
spring:
  task:
    execution:
      thread-name-prefix: task-
      pool:
        core-size: 5
        max-size: 25
        queue-capacity: 100
        allow-core-thread-timeout: true
        keep-alive: 60s
      shutdown:
        await-termination: true
        await-termination-period: 30s
```

| 속성                                                       | 기본값            | 설명                                         |
|-----------------------------------------------------------|-------------------|---------------------------------------------|
| `spring.task.execution.pool.core-size`                    | 8                 | 코어 스레드 수                                |
| `spring.task.execution.pool.max-size`                     | Integer.MAX_VALUE | 최대 스레드 수                                |
| `spring.task.execution.pool.queue-capacity`               | Integer.MAX_VALUE | 작업 대기 큐의 용량                            |
| `spring.task.execution.pool.allow-core-thread-timeout`    | true              | 코어 스레드 타임아웃 허용 여부                   |
| `spring.task.execution.pool.keep-alive`                   | 60s               | 코어 사이즈를 초과한 스레드가 종료 전 대기하는 시간 |
| `spring.task.execution.shutdown.await-termination`        | false             | 애플리케이션 종료시 수행중인 작업 완료 대기 여부   |
| `spring.task.execution.shutdown.await-termination-period` |                   | 애플리케이션 종료시 수행중인 작업 완료 대기 시간   |
| `spring.task.execution.thread-name-prefix`                | task-             | 스레드 이름 접두사                             |

## Java Config 설정
작업 성격에 따라 스레드 풀을 분리해야된다면 Java Config를 통해 여러개의 빈을 직접 등록해야 한다.
```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "apiExecutor")
    public Executor apiExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("api-async-");

        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());

        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);

        executor.initialize();
        return executor;
    }

    @Bean(name = "mailExecutor")
    public Executor mailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("mail-async-");

        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(30);
        
        executor.initialize();
        return executor;
    }
}
```

### RejectedExecutionHandler
`RejectedExecutionHandler`는 스레드 풀에서 더 이상 작업을 받을 수 없을 때 호출된다.

`maxPoolSize`까지 스레드가 생성되어 작업을 처리하고 있고, 작업 큐도 `queueCapacity`만큼 가득 찬 상황에서 새로운 작업이 들어오면 `RejectedExecutionHandler`가 동작하게 된다.

**거부 정책 종류**
- **AbortPolicy** (기본값)
  - `RejectedExecutionException` 예외가 발생한다.
  - 작업 누락을 즉시 알아야 하는 경우
- **CallerRunsPolicy**
  - 요청한 스레드에서 직접 실행한다.
  - 작업의 누락 없이 모두 처리해야하는 경우
- **DiscardPolicy**
  - 거부된 작업이 버려진다.
  - 중요도가 낮고 작업이 누락되어도 괜찮은 경우
- **DiscardOldestPolicy**
  - 큐에서 가장 오래된 작업이 버려진다.
  - 중요도가 낮고 작업이 누락되어도 괜찮은 경우. 최신 데이터가 더 중요한 경우


### AsyncUncaughtExceptionHandler
`AsyncUncaughtExceptionHandler`는 반환 값이 없는(`void`) 비동기 메서드에서 에러가 발생했을 경우 호출된다.
기본적으로는 `SimpleAsyncUncaughtExceptionHandler`가 동작하여 에러 로그를 남긴다.

다음과 같이 커스텀 핸들러를 지정할 수 있다.
```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new CustomAsyncExceptionHandler();
    }
}
```

```java
public class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(Throwable ex, Method method, @Nullable Object... params) {
        // 예외 처리
    }
}
```

## 비동기 기능 활성화
`@Configuration` 클래스에 `@EnableAsync` 어노테이션을 추가하여, `@Async` 어노테이션을 활성화한다.
```java
@Configuration
@EnableAsync
public class AsyncConfig {
}
```

## @Async
`@Async` 주석을 추가하여 비동기적으로 실행될 메서드를 지정할 수 있다.
```java
@Async
public void doSomething() {
	// ...
}
```

비동기 작업의 결과를 받아야 하는 경우 리턴 타입이 `Future` 계열이어야한다.
```java
@Async
public Future<String> returnSomething(int i) {
	// ...
}
```

## 동작 과정
- `@Async` 어노테이션이 붙은 메서드가 호출되면, 스프링은 해당 호출을 가로채서 비동기 실행을 처리하기 위한 프록시 객체를 생성한다.
- `TaskExecutor`에 의해 스레드 풀에 작업으로 등록된다.
- 작업은 별도의 스레드에서 처리되며, 호출자 메서드는 블로킹되지 않고 즉시 리턴된다.

## 예시
```java
@Slf4j
@Service
public class AsyncService {

    @Async
    public void doSomething(int number) {
        log.info("[doSomething] START");
        try {
            Thread.sleep(2000);
            if (number == -1) {
                throw new RuntimeException();
            }
            log.info("[doSomething] END");
        } catch (InterruptedException e) {
            log.error("[doSomething] ERROR", e);
            throw new RuntimeException(e);
        }
    }

    @Async
    public CompletableFuture<String> returnSomething(int number) {
        log.info("[returnSomething] START");
        try {
            Thread.sleep(2000);
            if (number == -1) {
                throw new RuntimeException();
            }
            log.info("[returnSomething] END");
            return CompletableFuture.completedFuture("(" + number + ")");
        } catch (InterruptedException e) {
            log.error("[returnSomething] ERROR", e);
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

```java
@SpringBootTest
public class AsyncTest {
    private static final Log log = LogFactory.getLog(AsyncTest.class);

    @Autowired
    AsyncService asyncService;

    @Test
    void test() throws InterruptedException {
        log.info("======= START =======");

        asyncService.doSomething(1);

        log.info("======= END =======");
        Thread.sleep(3000);
    }

    @Test
    void testFuture() throws InterruptedException {
        log.info("======= START =======");

        asyncService.returnSomething(1)
                .thenAccept(result -> {
                    // 성공 시
                    log.info("Success: " + result);
                })
                .exceptionally(ex -> {
                    // 예외 발생 시
                    log.error("Error: " + ex.getMessage());
                    return null;
                });

        log.info("======= END =======");
        Thread.sleep(3000);
    }
}
```
---
**Reference**
- https://docs.spring.io/spring-framework/reference/integration/scheduling.html
- https://fvor001.tistory.com/140
