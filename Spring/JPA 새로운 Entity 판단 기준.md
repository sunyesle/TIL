# 새로운 Entity 판단 기준
## 요약
| 구분                            | 새로운 엔티티 판단 기준                  |
|-------------------------------|--------------------------------|
| `Persistable` 구현체             | 엔티티의 `isNew()` 반환값이 `true`인 경우 |
| reference 타입 `@Version` 필드 존재 | `@Version` 필드 값이 null인 경우      |
| `@Id` (reference 타입)          | `@Id` 필드 값이 `null`인 경우         |
| `@Id` (Number 하위 타입)          | `@Id` 필드 값이 `0`인 경우            |

## 동작 과정
`SimpleJpaRepository`의 `save()` 메서드를 확인해 보자. `EntityInformation`의 `isNew()` 메서드를 통해 새로운 엔티티인지 아닌지 판단한다.
```java
@Override
@Transactional
public <S extends T> S save(S entity) {

    Assert.notNull(entity, ENTITY_MUST_NOT_BE_NULL);

    if (entityInformation.isNew(entity)) {
        entityManager.persist(entity);
        return entity;
    } else {
        return entityManager.merge(entity);
    }
}
```

### EntityInformation 구현체 선택
어떤 `EntityInformation` 구현체가 사용되는지는 `JpaEntityInformationSupport` 클래스를 확인하면 알 수 있다.

일반적으로는 `JpaMetamodelEntityInformation` 객체를, 엔티티가 `Persistable` 인터페이스를 구현했다면 `JpaPersistableEntityInformation` 객체를 반환한다.
```java
public static <T> JpaEntityInformation<T, ?> getEntityInformation(Class<T> domainClass, Metamodel metamodel,
        PersistenceUnitUtil persistenceUnitUtil) {

    Assert.notNull(domainClass, "Domain class must not be null");
    Assert.notNull(metamodel, "Metamodel must not be null");

    ManagedType<T> type = metamodel.managedType(domainClass);

    if (type instanceof EntityType<T> entityType) {
        if (Persistable.class.isAssignableFrom(domainClass)) {
            return new JpaPersistableEntityInformation(entityType, metamodel, persistenceUnitUtil);
        } else {
            return new JpaMetamodelEntityInformation(entityType, metamodel, persistenceUnitUtil);
        }
    }

    if (Persistable.class.isAssignableFrom(domainClass)) {
        return new JpaPersistableEntityInformation(domainClass, metamodel, persistenceUnitUtil);
    } else {
        return new JpaMetamodelEntityInformation(domainClass, metamodel, persistenceUnitUtil);
    }
}
```

구현체별 `isNew()` 동작을 확인해 보자.

### JpaMetamodelEntityInformation
`JpaMetamodelEntityInformation`의 `isNew()` 메서드는 다음과 같이 구현되어 있다.
```java
@Override
public boolean isNew(T entity) {

    if (versionAttribute.isEmpty()
            || versionAttribute.map(Attribute::getJavaType).map(Class::isPrimitive).orElse(false)) {
        return super.isNew(entity);
    }

    BeanWrapper wrapper = new DirectFieldAccessFallbackBeanWrapper(entity);

    return versionAttribute.map(it -> wrapper.getPropertyValue(it.getName()) == null).orElse(true);
}
```

엔티티에 `@Version` 필드가 없거나, `@Version` 필드가 primitive 타입인 경우, `super.isNew()`로 `AbstractEntityInformation`의 `isNew()` 메서드를 호출한다.

`@Version` 필드가 reference 타입인 경우, `version` 필드의 `null` 여부로 새로운 엔티티인지 판단한다.

`AbstractEntityInformation`의 `isNew()`는 다음과 같이 구현되어 있다.
```java
@Override
public boolean isNew(T entity) {

    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;
    }

    if (id instanceof Number n) {
        return n.longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s", idType));
}
```
`@Id` 필드가 primitive 타입이 아니라면 `null` 여부로, `Number`의 하위 타입이라면 `0`인지 여부로 새로운 엔티티인지 판단한다.

`JpaMetamodelEntityInformation`의 `isNew()` 동작을 요약하자면 다음과 같다.
- `@Version` 필드가 없거나, primitive 타입인 경우
  - `@Id` 필드를 기반으로 새로운 엔티티인지 판단한다.
    - `@Id` 필드가 reference 타입이면 `null` 여부로 판단
    - `@Id` 필드가 `Number`의 하위 타입이면 `0`인지 여부로 판단
- `@Version` 필드가 reference 타입인 경우
  - `@Version` 필드의 null 여부로 새로운 엔티티인지 판단한다.

### JpaPersistableEntityInformation
`JpaPersistableEntityInformation`의 `isNew()` 메서드는 다음과 같이 구현되어 있다.
```java
@Override
public boolean isNew(T entity) {
    return entity.isNew();
}
```
엔티티의 `isNew()` 메서드를 호출한다.
즉, 엔티티가 `Persistable` 인터페이스를 구현하면 새로운 엔티티 여부를 판단하는 기준을 직접 정의할 수 있다.

---
**Reference**
- https://ttl-blog.tistory.com/852
- https://ttl-blog.tistory.com/807
- https://github.com/2024-woowacourse-study/level-interview/discussions/127
