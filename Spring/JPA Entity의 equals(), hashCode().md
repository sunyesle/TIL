# JPA Entity의 equals(), hashCode()

엔티티를 단일 트랜잭션 내에서만 사용하는 경우 `equals()`, `hashCode()`를 재정의할 필요는 없다.
영속성 컨텍스트에 의해 엔티티의 동일성(Identity)이 보장되기 때문이다.

단, 반드시 다음 규칙을 준수해야 한다.
- 서로 다른 트랜잭션에서 조회한 엔티티 객체 간 비교를 위해 `equals()`를 호출하지 않는다.
- 해시 기반 컬렉션을 사용하는 경우 트랜잭션 범위 내에서만 사용한다.

`equals()`는 기본적으로 참조 동등성, 즉 두 객체가 메모리상에서 동일한 인스턴스인지 비교한다.
**DB 상에서 동일한 레코드인지 비교**하고 싶다면, 반드시 **`equals()`, `hashCode()` 메서드를 오버라이드**해야 한다.

## 애플리케이션 생성 ID 기반
ID 생성을 애플리케이션 단에서 처리하거나 불변이고 `null`이 아닌 비즈니스 키가 존재하는 경우, 이를 기준으로 동일 레코드인지 판별할 수 있다.
```java
@Override
public final boolean equals(Object o) {
    if (this == o) return true; // 주소 비교
    if (!(o instanceof Student obj)) return false; // null 체크 + 타입 체크
    return getId() != null && Objects.equals(getId(), obj.getId()); // ID 기반 비교
}

@Override
public final int hashCode() {
    return Objects.hashCode(getId());
}
```

## DB 생성 ID 기반
ID 생성을 DB에게 위임하는 경우 영속화 이전에 ID가 `null`인 경우를 고려해야 한다.

`Objects.hashCode(getId())`를 사용하면 영속화 이후 ID가 할당되면서 `hashCode` 값이 변하게 되고,
이미 해당 객체를 담고 있던 해시 기반 컬렉션(`HashSet`, `HashMap` 등)에서 데이터를 찾지 못하는 버그가 발생한다.

이를 방지하기 위해 `getClass().hashCode()`을 사용하여 영속화 이전과 이후의 일관성을 보장한다.

모든 동일 타입 엔티티가 동일한 버킷에 저장되어 성능이 저하되지만 일반적으로 한 컬렉션에 수 만건의 데이터를 담는 경우는 드물기 때문에, 성능 저하의 오버헤드보다 객체 유실 버그를 막는 이득이 크다.
```java
// equals() 동일

@Override
public final int hashCode() {
    return getClass().hashCode();
}
```

---
**Reference**
- https://www.baeldung.com/jpa-entity-equality
- https://jpa-buddy.com/blog/hopefully-the-final-article-about-equals-and-hashcode-for-jpa-entities-with-db-generated-ids/
