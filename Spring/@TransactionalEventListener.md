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

---
**Reference**
- https://github.com/spring-projects/spring-framework/blob/main/spring-tx/src/main/java/org/springframework/transaction/support/AbstractPlatformTransactionManager.java
- https://lenditkr.github.io/spring/transactional-event-listener/
- https://curiousjinan.tistory.com/entry/fixing-spring-transactionaleventlistener-after-commit-update-issue
- https://dev.to/haraf/understanding-transactioneventlistener-in-spring-boot-use-cases-real-time-examples-and-4aof
