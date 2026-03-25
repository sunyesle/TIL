# Spring Boot Graceful Shutdown

## application.yml
```yml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```
| 속성                                          | 기본값   | 설명                                             |
|-----------------------------------------------|----------|--------------------------------------------------|
| `server.shutdown`                             | graceful | graceful shutdown 사용 여부 (graceful, immediate) |
| `spring.lifecycle.timeout-per-shutdown-phase` | 30s      | 종료 페이즈마다 허용되는 최대 대기 시간             |

> 적절한 `SIGTERM` 신호를 보내지 않는 경우, graceful shutdown 설정을 했더라도 즉시 종료될 수 있다.

## SmartLifecycle과 Graceful Shutdown
스프링은 `SmartLifecycle` 인터페이스를 통해 빈의 생명주기를 관리한다.

`SmartLifecycle` 인터페이스는 `ApplicationContext`가 시작되거나 종료될 때 각 빈이 `Phase`(순서)에 맞게 처리되도록 한다.
`Phase` 값은 빈이 시작되고 종료될 단계를 결정하며, 시작 시에는 값이 낮은 빈부터, 종료 시에는 값이 높은 빈부터 처리된다.

애플리케이션 종료 시 흐름은 다음과 같다.
```
Tomcat/Web Server (Phase 2147482623):
→ stop() 호출
→ 최대 30초 대기
→ 30초 지나면 다음 Phase로

TaskExecutor/Scheduler (Phase 1073741823):
→ stop() 호출
→ 최대 30초 대기
→ 30초 지나면 다음 Phase로

일반 Bean (Phase 0):
→ stop() 호출
→ 최대 30초 대기
```

### Tomcat/Web Server Shutdown
톰캣은 Phase 값이 `Integer.MAX_VALUE - 1024`로 높게 설정되어 있어 가장 먼저 종료된다.<br>
(관련 클래스: `WebServerGracefulShutdownLifecycle`)

종료 신호를 받으면 즉시 새로운 요청 수락을 중단하고, 기존에 처리 중인 요청들이 완료될 때까지 대기한다.

이때, 설정한 `spring.lifecycle.timeout-per-shutdown-phase`동안 완료되지 않았다면, 해당 페이즈를 강제 종료하고 다음 단계로 넘어간다.

**요청 처리 시간을 고려하여 적절한 `spring.lifecycle.timeout-per-shutdown-phase`을 설정해야 한다.**

### TaskExecutor/Scheduler Shutdown
TaskExecutor는 Phase 값이 `Integer.MAX_VALUE / 2`로 일반적인 빈(Phase 0)보다 먼저 종료된다.<br>
(관련 클래스: `ExecutorConfigurationSupport`)

기본적으로는 종료 신호를 받으면 새 작업을 거부하고, 실행 중인 작업이 즉시 중단된다.

`TaskExecutor`의 `waitForTasksToCompleteOnShutdown`를 `true`로 설정하면 새 작업은 거부하지만, 실행 중인 작업과 큐에 들어온 작업이 완료될 때까지 대기하며, 최대 대기 시간은 `awaitTerminationMillis`로 지정할 수 있다.

이때, 설정한 `awaitTerminationMillis`가 `spring.lifecycle.timeout-per-shutdown-phase`보다 길 경우, 작업이 완료되기 전에 다음 페이즈로 넘어가고 빈들이 정리되기 시작한다.
`TaskExecutor`의 작업은 여전히 백그라운드에서 돌고 있지만, 의존하던 빈이 제거되면서 에러가 발생할 수 있다.

**작업이 다른 빈에 의존하는 경우 `awaitTerminationMillis`를 `spring.lifecycle.timeout-per-shutdown-phase`보다 작게 설정해야 한다.**

---
**Reference**
- https://docs.spring.io/spring-boot/reference/web/graceful-shutdown.html
- https://www.baeldung.com/spring-boot-graceful-shutdown
- https://dkswhdgur246.tistory.com/102
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/SmartLifecycle.html
