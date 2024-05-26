# @EnableJpaAuditing

엔티티 객체가 생성되거나 변경되었을 때 @EnableJpaAuditing 어노테이션을 활용해서 자동으로 값을 등록할 수 있다.

### Entity
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
``@EntityListeners(AuditingEntityListener.class)`` : ``@EntityListener``는 JpaEntity에 특정 이벤트가 발생했을 때 콜백을 요청할 수 있는 어노테이션이다. Auditing을 적용하기 위해 ``AuditingEntityListener``를 사용하도록 한다.


### AuditorAware
``@CreatedBy``또는 ``@LastModifiedBy``를 사용하는 경우 현재 사용자를 인식할 수 있도록 ``AuditorAware<T>``를 구현해야한다.
``T``는 ``@CreatedBy``, ``@LastModifiedBy`` 주석이 달린 필드의 타입으로 정의한다.
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

### Auditing 설정
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
``@EnableJpaAuditing``: JpaAuditing 기능을 활성화한다.

``AuditorAware``타입의 Bean이 존재할 경우 자동으로 사용한다. 여러개의 ``AuditorAware`` Bean이 있다면 ``@EnableJpaAuditing``의 ``auditorAwareRef`` 속성에 Bean 이름을 명시하여 사용할 것을 선택할 수 있다.

## 테스트시 주의사항

``@WebMvcTest``로 Controller 테스트 진행 중 오류가 발생했다.

***발생했던 에러** : Error creating bean with name 'jpaAuditingHandler': Cannot resolve reference to bean 'jpaMappingContext' while setting constructor argument*

``@SpringBootApplication`` 클래스는 슬라이스 테스트의 기본 구성으로 사용된다. 만약 ``@SpringBootApplication`` 클래스에 ``@EnableJpaAuditing``을 추가할 경우 슬라이스 테스트 시에도 JPA관련 Bean을 필요로 하는 상태가 된다.

```java
@EnableJpaAuditing
@SpringBootApplication
public class ExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

의도하는 동작이 아니며, JPA와 관련 Bean을 로드하지 않아 오류가 발생할 수 있기 때문에 ``@EnableJpaAuditing``를 별도의 ``@Configuration`` 클래스로 분리하는것이 권장된다. [(공식문서)](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.user-configuration-and-slicing)   

```java
@Configuration
@EnableJpaAuditing
public class JpaAuditingConfig {
    ...
}
```

### Repository 테스트 시 주의점

위와 같이 설정 클래스를 분리했다면 ``@DataJpaTest`` 테스트 시에 JPA Auditing 기능 활성화를 위해 설정 클래스를 import 해주어야한다.

```java
@DataJpaTest
@Import(JpaAuditingConfig.class)
public class RepositoryTest {
    ...
}
```

---

### Reference

https://docs.spring.io/spring-data/jpa/reference/auditing.html

https://stackoverflow.com/questions/51467132/spring-webmvctest-with-enablejpa-annotation

https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.user-configuration-and-slicing

https://hellorennon.tistory.com/9

https://kdohyeon.tistory.com/51

https://gist.github.com/dlxotn216/94c34a2debf848396cf82a7f21a32abe
