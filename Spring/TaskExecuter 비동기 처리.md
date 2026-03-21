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

---
**Reference**
- https://docs.spring.io/spring-framework/reference/integration/scheduling.html
- https://fvor001.tistory.com/140
