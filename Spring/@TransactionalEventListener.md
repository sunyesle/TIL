# @TransactionalEventListener
`@TransactionalEventListener`는 트랜잭션과 연동되어 실행되는 이벤트 리스너를 지정하는 어노테이션이다.

## TransactionPhase
`phase` 옵션을 통해 이벤트를 처리할 시점을 지정할 수 있다.
- `TransactionPhase.BEFORE_COMMIT`
  - 트랜잭션이 commit 되기 전에 이벤트를 실행한다. 이벤트를 발행한 쪽의 DB 트랜잭션 범위에 포함된다.
- `TransactionPhase.AFTER_COMMIT`
  - 트랜잭션이 commit 되었을 때 이벤트를 실행한다. (default)
- `TransactionPhase.AFTER_ROLLBACK`
  - 트랜잭션이 rollback 되었을 때 이벤트를 실행한다.
- `TransactionPhase.AFTER_COMPLETION`
  - 트랜잭션이 completion(commit 또는 rollback) 되었을 때 이벤트 실행한다.

## 내부 동작 과정
> Spring Boot 4.0.3 기준으로 작성되었다.

트랜잭션을 커밋/롤백할 때 `AbstractPlatformTransactionManager`의 메서드를 호출한다.
```java
public abstract class AbstractPlatformTransactionManager
		implements PlatformTransactionManager, ConfigurableTransactionManager, Serializable {

    private void processCommit(DefaultTransactionStatus status) throws TransactionException {
        // 커밋 준비
        prepareForCommit(status);

        // BEFORE_COMMIT
        triggerBeforeCommit(status);

        // 실제 DB 커밋 수행
        doCommit(status);

        // AFTER_COMMIT, AFTER_COMPLETION
        triggerAfterCompletion(status, TransactionSynchronization.STATUS_COMMITTED);

        // 트랜잭션 정리
        cleanupAfterCompletion(status);
    }

    private void processRollback(DefaultTransactionStatus status, boolean unexpected) {
        // 실제 DB 롤백 수행
        doRollback(status);
        
        // AFTER_ROLLBACK, AFTER_COMPLETION
        triggerAfterCompletion(status, TransactionSynchronization.STATUS_ROLLED_BACK);

        // 트랜잭션 정리
        cleanupAfterCompletion(status);
    }
}
```

`trigger...()` 메서드는 등록된 `TransactionSynchronization`를 순회하며 콜백 메서드를 호출한다.
```java
public abstract class TransactionSynchronizationUtils {

	public static void triggerBeforeCommit(boolean readOnly) {
		for (TransactionSynchronization synchronization : TransactionSynchronizationManager.getSynchronizations()) {
			synchronization.beforeCommit(readOnly);
		}
	}

	public static void invokeAfterCompletion(@Nullable List<TransactionSynchronization> synchronizations,
			int completionStatus) {

		if (synchronizations != null) {
			for (TransactionSynchronization synchronization : synchronizations) {
				try {
					synchronization.afterCompletion(completionStatus);
				}
				catch (Throwable ex) {
					logger.error("TransactionSynchronization.afterCompletion threw exception", ex);
				}
			}
		}
	}
}
```

최종적으로 `TransactionSynchronization` 구현체 `TransactionalApplicationListenerSynchronization`에서 각 이벤트 리스너의 호출이 이루어진다.
```java
abstract class TransactionalApplicationListenerSynchronization<E extends ApplicationEvent> implements Ordered {

	private static class PlatformSynchronization<AE extends ApplicationEvent>
			extends TransactionalApplicationListenerSynchronization<AE>
			implements org.springframework.transaction.support.TransactionSynchronization {

		@Override
		public void beforeCommit(boolean readOnly) {
			if (getTransactionPhase() == TransactionPhase.BEFORE_COMMIT) {
				processEventWithCallbacks();
			}
		}

		@Override
		public void afterCompletion(int status) {
			TransactionPhase phase = getTransactionPhase();
			if (phase == TransactionPhase.AFTER_COMMIT && status == STATUS_COMMITTED) {
				processEventWithCallbacks();
			}
			else if (phase == TransactionPhase.AFTER_ROLLBACK && status == STATUS_ROLLED_BACK) {
				processEventWithCallbacks();
			}
			else if (phase == TransactionPhase.AFTER_COMPLETION) {
				processEventWithCallbacks();
			}
		}
	}
}
```

## 주의 사항
### 이벤트 페이즈별 실행 순서
`TransactionalApplicationListenerSynchronization`의 `afterCompletion()` 메서드에서 `AFTER_COMMIT`, `AFTER_ROLLBACK`, `AFTER_COMPLETION` 페이즈를 모두 처리한다.
이로인해, `AFTER` 계열 이벤트의 호출 순서는 등록된 `TransactionSynchronization` 목록의 순서에 따라 정해지게 된다.

이벤트 리스너간의 순서가 중요한 경우, 이벤트 리스너에 `@Order`를 사용해서 **명시적으로 우선순위를 지정**해야 한다.

### 리스너 내 DB 작업
`AFTER_COMMIT`, `AFTER_ROLLBACK`, `AFTER_COMPLETION` 메서드가 실행되는 시점은 DB 트랜잭션은 종료되었지만, 스프링 트랜잭션 컨텍스트는 아직 유지되고 있는 상태이다.

이 시점에는 새로운 DB 작업을 수행해도 실제 DB에 반영되지 않으며, 필요하다면 **명시적으로 새로운 스프링 트랜잭션 컨텍스트를 요청**해야 한다.

- **트랜잭션 전파 속성 `@Transactional(propagation = Propagation.REQUIRES_NEW)`**: 항상 새로운 트랜잭션을 시작하도록 명시한다.
- **이벤트 리스너 `@Async`**: 이벤트를 비동기로 다른 스레드에서 실행하여 새로운 트랜잭션을 시작하도록 한다.

### 리스너 예외 처리
`TransactionSynchronizationUtils.invokeAfterCompletion()` 메서드를 확인해보면 `try-catch`로 예외를 잡아 로깅만 수행하는 것을 확인할 수 있다.

Spring Boot 3.2 (Spring Framework 6.1.0) 전 버전에서는 로그 레벨이 `error`가 아니라 `debug`로 설정되어있다. ([관련 커밋](https://github.com/spring-projects/spring-framework/commit/3f65b8506bbeda0f321408e95dbc92e2b00eca04))

이전 버전의 스프링을 사용하는 경우, 에러가 묻히지 않도록 이벤트 리스너 내에서 `try-catch`하는 것이 좋다.


---
**Reference**
- https://github.com/spring-projects/spring-framework/blob/main/spring-tx/src/main/java/org/springframework/transaction/support/AbstractPlatformTransactionManager.java
- https://lenditkr.github.io/spring/transactional-event-listener/
- https://curiousjinan.tistory.com/entry/fixing-spring-transactionaleventlistener-after-commit-update-issue
- https://dev.to/haraf/understanding-transactioneventlistener-in-spring-boot-use-cases-real-time-examples-and-4aof
