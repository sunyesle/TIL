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

---
**Reference**
- https://docs.spring.io/spring-boot/reference/web/graceful-shutdown.html
- https://www.baeldung.com/spring-boot-graceful-shutdown
- https://dkswhdgur246.tistory.com/102
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/SmartLifecycle.html
