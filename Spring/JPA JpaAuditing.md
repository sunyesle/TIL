# JPA JpaAuditing
Auditing 기능을 통해 엔티티를 생성/수정 시점과 작업자를 자동으로 기록할 수 있다.

## Auditing 활성화
`@Configuration` 클래스에 `@EnableJpaAuditing` 어노테이션을 추가하여, Auditing 기능을 활성화한다.
```java
@Configuration
@EnableJpaAuditing
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<Long> userAuditorAware() {
        return new UserAuditorAware();
    }
}
```

## AuditorAware
생성자/수정자 추적을 사용하는 경우, 현재 사용자를 인식할 수 있도록 `AuditorAware<T>`를 구현해야 한다. `T`는 `@CreatedBy`, `@LastModifiedBy` 어노테이션이 달린 필드의 타입이다.
```java
public class UserAuditorAware implements AuditorAware<Long> {

    @Override
    public Optional<Long> getCurrentAuditor() {
        return Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(authentication -> !authentication.getPrincipal().equals("anonymousUser"))
                .map(Authentication::getPrincipal)
                .map(CustomUserDetails.class::cast)
                .map(CustomUserDetails::getId);
    }
}
```

## Entity
`@EntityListeners`는 엔티티의 생명주기 이벤트가 발생했을 때, 콜백을 처리할 수 있게 해주는 어노테이션이다.
Auditing 기능을 적용하려면 `AuditingEntityListener`를 리스너로 등록해야 한다.
```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime modifiedAt;

    @CreatedBy
    @Column(updatable = false)
    private Long createdBy;

    @LastModifiedBy
    private Long modifiedBy;
}
```

## @DataJpaTest 테스트 시 주의점
`@DataJpaTest` 테스트 시에 Auditing 기능을 활성화하려면 다음과 같이 설정 클래스를 import해야 한다.
```java
@DataJpaTest
@Import(JpaAuditingConfig.class)
public class RepositoryTest {
}
```

## 비즈니스 로직에서 Auditing 필드에 의존하지 말자
### NPE 위험
Auditing 필드는 JPA의 생명주기 콜백(`@PrePersist`, `@PreUpdate`)에 의존한다.
따라서 영속화 이전에는 해당 필드가 `null`이며, 비즈니스 로직에서 이를 사용할 경우 `NullPointerException`이 발생할 위험이 있다.

### 단위 테스트 작성의 어려움
Auditing 필드는 JPA 영속성 컨텍스트가 활성화되어야만 값이 채워진다.
비즈니스 로직 단위 테스트 시에 해당 필드를 사용하려면, 관련 JPA 빈을 모킹하거나 통합 테스트를 수행해야 하며, 이는 테스트 코드의 작성과 유지보수를 까다롭게 만든다.

### 해결 방안
핵심 로직에서 필요하다면 Auditing 필드에 의존하지 말고, 별도의 필드를 추가하는 것이 좋다.
```java
@Entity
public class Order extends BaseEntity {
    private LocalDateTime orderedAt; 

    public void placeOrder(LocalDateTime now) {
        this.orderedAt = now;
        // ...
    }
}
```

## 관련 오류
`@WebMvcTest`로 컨트롤러 테스트 진행 중 오류가 발생했다.
```
Error creating bean with name 'jpaAuditingHandler': Cannot resolve reference to bean 'jpaMappingContext' while setting constructor argument*
```

`@SpringBootApplication` 클래스는 슬라이스 테스트의 기본 구성으로 사용된다. 만약 다음과 같이 `@SpringBootApplication` 클래스에 `@EnableJpaAuditing`을 추가하는 경우 슬라이스 테스트 시에도 JPA 관련 빈을 필요로 하는 상태가 된다.

```java
@EnableJpaAuditing
@SpringBootApplication
public class ExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

이는 의도하는 동작이 아니며, JPA 관련 빈을 로드하지 않아 오류가 발생할 수 있기 때문에 `@EnableJpaAuditing`를 별도의 `@Configuration` 클래스로 분리하는 것이 권장된다. [(공식문서)](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.user-configuration-and-slicing)   

---
**Reference**
- https://docs.spring.io/spring-data/jpa/reference/auditing.html
- https://stackoverflow.com/questions/51467132/spring-webmvctest-with-enablejpa-annotation
- https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.user-configuration-and-slicing
- https://hellorennon.tistory.com/9
- https://kdohyeon.tistory.com/51
- https://gist.github.com/dlxotn216/94c34a2debf848396cf82a7f21a32abe
