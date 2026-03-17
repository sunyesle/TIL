# TaskScheduler 스케줄링
스프링 부트는 주기적인 작업을 수행할 수 있도록하는 스케줄링 기능을 제공한다.

## application.yml
```yml
spring:
  task:
    scheduling:
      thread-name-prefix: scheduling-
      pool:
        size: 1
      shutdown:
        await-termination: true
        await-termination-period: 30s
```

| 속성                                                        | 기본값         | 설명                                                |
|------------------------------------------------------------|---------------|-----------------------------------------------------|
| `spring.task.scheduling.pool.size`                         | `1`           | 스케줄러가 사용하는 스레드 풀의 크기                      |
| `spring.task.scheduling.shutdown.await-termination`        | `false`       | 애플리케이션 종료시 수행중인 scheduled task 완료 대기 여부 |
| `spring.task.scheduling.shutdown.await-termination-period` | `0`           | 애플리케이션 종료시 수행중인 scheduled task 완료 대기 시간 |
| `spring.task.scheduling.thread-name-prefix`                | `scheduling-` | 스레드 이름 접두사                                     |


## 스케줄링 활성화
`@Configuration` 클래스에 `@EnableScheduling` 어노테이션을 추가하여, `@Scheduled` 어노테이션을 활성화한다.
```java
@Configuration
@EnableScheduling
public class SchedulingConfig {
}
```

## @Scheduled
메서드에 `@Scheduled` 어노테이션을 추가하여 스케줄링 될 메서드를 지정할 수 있다.
메서드는 반환 값이 `void`여야 하고, 파라미터를 받아서는 안 된다. 

### FixedDelay
고정된 지연 시간으로 호출된다. (작업이 *끝난* 시점부터 지정된 시간만큼 기다린 후 다음 작업 실행)
```java
@Scheduled(fixedDelay = 5000)
public void doSomething() {
	// ...
}
```
> 다음과 같이 표현할 수도 있다.<br>
> `@Scheduled(fixedDelay = 5, timeUnit = TimeUnit.SECONDS) // TimeUnit 지정`

### fixedRate
고정된 속도로 호출된다. (작업이 *시작된* 시점부터 지정된 시간만큼 기다린 후 다음 작업 실행)
```java
@Scheduled(fixedRate = 5000)
public void doSomething() {
	// ...
}
```
### fixedRate
메서드의 첫 번째 실행 전에 대기할 시간을 지정한다.

`fixedDelay`, `fixedRate`와 함께 사용할 수 있으며, 일회성 작업의 경우 다음과 같이 초기 지연 시간을 지정할 수 있다.
```java
@Scheduled(initialDelay = 1000)
public void doSomething() {
	// something that should run only once
}
```

### cron
cron 표현식으로 실행 간격을 지정한다.
```java
@Scheduled(cron="*/5 * * * * MON-FRI")
public void doSomething() {
	// something that should run on weekdays only
}
```

---
**Reference**
- https://docs.spring.io/spring-framework/reference/integration/scheduling.html
- https://devel-repository.tistory.com/47
