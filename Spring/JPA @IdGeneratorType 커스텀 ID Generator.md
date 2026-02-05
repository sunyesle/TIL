# JPA @IdGeneratorType 커스텀 ID Generator
`@GenericGenerator`는 Hibernate 6.5 버전부터 deprecated 되었다. 대신 `@IdGeneratorType`를 이용해 커스텀 어노테이션으로 래핑하여 사용하는 것이 권장된다.
```log
'org.hibernate.annotations.GenericGenerator' is deprecated since version 6.5 and marked for removal
```

## 예시
`@IdGeneratorType`를 이용해 UUID v7 ID Generator를 구현해 보았다.

### Generator 구현
`IdentifierGenerator` 인터페이스를 구현한다. `generate()` 메서드는 ID를 반환한다.
```java
public class UuidV7IdGenerator implements IdentifierGenerator {
    @Override
    public Object generate(SharedSessionContractImplementor sharedSessionContractImplementor, Object o) {
        return Generators.timeBasedEpochGenerator().generate();
    }
}
```
> UUID 생성에는 `java-uuid-generator` 라이브러리를 사용하였다.

### 커스텀 어노테이션 정의
구현한 Generator를 엔티티에서 사용하기 위해 커스텀 어노테이션을 정의한다. 이때 `@IdGeneratorType`에 구현한 Generator 클래스를 지정한다.
```java
@IdGeneratorType(UuidV7IdGenerator.class)
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD})
public @interface UuidV7Generator {
}
```

### 적용
ID 필드에 어노테이션을 추가하여 적용할 수 있다. 
```java
@Entity
public class Member {
    @Id
    @UuidV7Generator
    @Column(name = "member_id", columnDefinition = "BINARY(16)")
    private UUID id;

    // ...
}
```

---
**Reference**
- https://medium.com/@nipunasan/how-to-use-idgeneratortype-in-hibernate-6-5-spring-boot-3-3-for-custom-id-generation-403be9b51d6f
