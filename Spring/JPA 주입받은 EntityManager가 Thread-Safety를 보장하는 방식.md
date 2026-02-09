# JPA 주입받은 EntityManager가 Thread-Safety를 보장하는 방식
Spring Data JPA를 사용하다 보면, 테스트 코드나 QueryDsl 설정 등을 위해 `EntityManager`를 주입받아 사용하는 경우가 있다.
```java
@Autowired
EntityManager em;
```

하지만, Hibernate의 [공식 문서](https://docs.hibernate.org/orm/7.2/userguide/html_single/#session-per-application)를 보면 `EntityManager`는 thread-safe 하지 않으며 여러 스레드에서 공유하면 문제가 발생할 수 있다고 나와있다.
> The Hibernate `Session`, like the Jakarta Persistence `EntityManager`, is not a thread-safe object and it is intended to be confined to a single thread at once.
> If the `Session` is shared among multiple threads, there will be race conditions as well as visibility issues, so beware of this.

그런데 어떻게 Spring에서는 `EntityManager`를 빈으로 등록하고 여러 코드에서 주입받아 사용할 수 있는 걸까?

## 내부 동작 과정
주입받은 `EntityManager`를 디버깅해 보면 **Proxy 객체**가 주입된 것을 확인할 수 있다.

주입된 객체는 `SharedEntityManagerCreator`의 중첩 클래스인 `SharedEntityManagerInvocationHandler`로
메서드 호출 시점마다 현재 트랜잭션의 `EntityManager`를 가져와서 실제 작업을 위임하는 역할을 한다.
```java

public abstract class SharedEntityManagerCreator {

    private static class SharedEntityManagerInvocationHandler implements InvocationHandler, Serializable {

        public @Nullable Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // ...

            // 현재 트랜잭션의 EntityManager를 가져온다.
            EntityManager target = EntityManagerFactoryUtils.doGetTransactionalEntityManager(this.targetFactory, this.properties, this.synchronizedWithTransaction);

            try {
                // 실제 EntityManager의 메서드를 호출한다.
                Object result = method.invoke(target, args);
            } catch (InvocationTargetException ex) {
                throw ex.getTargetException();
            }

            // ...
        }
    }
}
```

프록시 객체가 호출한 `EntityManagerFactoryUtils`는 현재 트랜잭션에서 사용할 `EntityManager`를 찾는 역할을 한다.
```java
public abstract class EntityManagerFactoryUtils {
    public static @Nullable EntityManager doGetTransactionalEntityManager(EntityManagerFactory emf, @Nullable Map<?, ?> properties, boolean synchronizedWithTransaction) throws PersistenceException {
        // 현재 스레드에 바인딩 된 EntityManagerHolder를 가져온다.
        EntityManagerHolder emHolder = (EntityManagerHolder)TransactionSynchronizationManager.getResource(emf);
        
        // ... EntityManagerHolder가 없으면 새로 생성 ...

        // EntityManager를 추출하여 반환한다.
        return emHolder.getEntityManager();
    }
}
```

`TransactionSynchronizationManager`는 `ThreadLocal`을 이용해 스레드별 트랜잭션 리소스를 관리한다.

트랜잭션이 시작되면 `JpaTransactionManager`는 `EntityManagerFactory`를 key로 `EntityManagerHolder`를 `ThreadLocal`에 바인딩해 둔다.
따라서 같은 스레드 내에서는 항상 동일한 `EntityManager`가 조회된다.
```java
public abstract class TransactionSynchronizationManager {
    // ThreadLocal을 사용하여 스레드별 트랜잭션 리소스를 관리한다.
    private static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal("Transactional resources");

    public static @Nullable Object getResource(Object key) {
        Object actualKey = TransactionSynchronizationUtils.unwrapResourceIfNecessary(key);
        return doGetResource(actualKey);
    }

    private static @Nullable Object doGetResource(Object actualKey) {
        // 현재 스레드의 리소스를 가져온다.
        Map<Object, Object> map = (Map)resources.get();
        if (map == null) {
            return null;
        } else {
            // EntityManagerFactory를 key로 EntityManagerHolder를 조회한다.
            Object value = map.get(actualKey);
            
            // ...

            return value;
        }
    }
}
```

## 결론
즉, 프록시는 메서드 호출 시점마다 `ThreadLocal`에 바인딩된 현재 트랜잭션의 `EntityManager`를 찾아 위임한다.
이 구조 덕분에 `EntityManager`는 싱글톤처럼 사용되지만 스레드 간 충돌 없이 안전하게 동작한다.

---
**Reference**
- https://woodcock.tistory.com/35
- https://medium.com/@SlackBeck/spring-container는-jpa-entitymanager의-thread-safety를-어떻게-보장할까-1650473eeb64
- https://medium.com/@taesulee93/spring-data-jpa에서-entitymanager의-주입-방법과-주의점-autowired도-이젠-사용-가능하다-f85925b7ff9a
