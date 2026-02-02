# JPA 쓰기 지연의 동작 순서와 주의 사항
**쓰기 지연**이란 영속성 컨텍스트에 변경이 발생했을 때, 즉시 DB에 반영하지 않고 모아두었다가 flush 시점에 한꺼번에 반영하는 기능이다.

## IDENTITY 쓰기 지연 예외
예외적으로 ID 생성 전략이 `IDENTITY`인 엔티티를 저장하는 경우 쓰기 지연이 동작하지 않는다.

`IDENTITY`는 DB에 ID 생성을 위임한 전략으로 DB에 데이터 넣어야만 ID를 알 수 있기 때문에, 영속성 컨텍스트 관리를 위해 쿼리를 즉시 실행하게 된다.
([공식 문서](https://docs.hibernate.org/orm/7.2/userguide/html_single/#portability-idgen))

## 쓰기 지연의 동작 순서
Hibernate는 지연된 쿼리들을 `ActionQueue`에 저장해둔다.

`ActionQueue`의 작업은 다음 순서대로 실행된다.
([공식 문서](https://docs.hibernate.org/orm/7.2/userguide/html_single/#flushing-order))

- `OrphanRemovalAction`: 고아 객체 제거
- `EntityInsertAction`: 엔티티 삽입
- `EntityUpdateAction`: 엔티티 수정
- `QueuedOperationCollectionAction`: 지연된 컬렉션 작업
- `CollectionRemoveAction`: 컬렉션 요소 삭제
- `CollectionUpdateAction`: 컬렉션 요소 수정
- `CollectionRecreateAction`: 컬렉션 재생성
- `EntityDeleteAction`: 엔티티 삭제

## 주의 사항
이러한 쿼리 순서 변경으로 인해 문제가 발생하는 대표적인 케이스는,
**unique** 필드를 가진 레코드를 수정할 때 update 대신 **delete 후 다시 insert** 하는 방식을 사용하는 경우이다.

'**삭제 → 삽입**' 순으로 코드를 작성하더라도 실제 쿼리는 Hibernate의 우선순위에 따라 '**삽입 → 삭제**' 순으로 실행된다.

기존 데이터가 삭제되기 전에 동일한 unique 값을 가진 데이터가 삽입되어 **unique 위반으로 인한 오류가 발생**할 수 있다.

### 예시
상품의 태그 정보를 저장하는 엔티티가 있다. `productId`와 `name`에 대한 복합 unique key를 설정했다.
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(uniqueConstraints = {@UniqueConstraint(columnNames = {"productId", "name"})})
public class Tag {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private Long productId;

    @Column(nullable = false)
    private String name;

    public Tag(Long productId, String name) {
        this.productId = productId;
        this.name = name;
    }
}
```

`updateTags()` 메서드는 태그 목록을 교체하기 위해, 상품의 모든 태그를 삭제하고 새로운 태그를 저장한다.
```java
@Service
@RequiredArgsConstructor
public class TagService {
    private final TagRepository tagRepository;

    @Transactional
    public void updateTags(long productId, List<String> newTagNames) {
        System.out.println("----- [delete] -----");
        tagRepository.deleteByProductId(productId);

        System.out.println("----- [ save ] -----");
        for (String tagName : newTagNames) {
            tagRepository.save(new Tag(productId, tagName));
        }
    }
}
```

아래 테스트 코드를 실행하면 어떤 결과가 나올까?
```java
@Test
void test() {
    // 기존에 "Spring" 태그가 존재한다.
    tagRepository.save(new Tag(1L, "Spring"));
    tagRepository.flush();

    // "Spring" 태그를 유지하면서 "JPA" 태그를 추가한다.
    System.out.println("===== [start] =====");
    tagService.updateTags(1L, List.of("Spring", "JPA"));
    System.out.println("===== [ end ] =====");
}
```

> 실행 결과
```log
Hibernate: insert into tag (name,product_id) values (?,?)
===== [start] =====
----- [delete] -----
Hibernate: select t1_0.id,t1_0.name,t1_0.product_id from tag t1_0 where t1_0.product_id=?
----- [ save ] -----
Hibernate: insert into tag (name,product_id) values (?,?)
2026-02-02T22:38:59.800+09:00  WARN 16728 --- [spring-boot-jpa] [    Test worker] org.hibernate.orm.jdbc.error             : HHH000247: ErrorCode: 1062, SQLState: 23000
2026-02-02T22:38:59.800+09:00  WARN 16728 --- [spring-boot-jpa] [    Test worker] org.hibernate.orm.jdbc.error             : Duplicate entry '1-Spring' for key 'tag.UK_tag_product_name'
```
로그를 보면 `deleteByProductId()` 호출 시점에 실제 delete 쿼리는 나가지 않고, 엔티티 조회를 위한 select 쿼리만 찍힌 것을 확인할 수 있다.
delete 쿼리가 `ActionQueue`에 담겨 지연되었기 때문이다. 

이후, `save()`는 IDENTITY 전략 때문에 호출 즉시 insert 쿼리가 실행된다.

결국, 기존 데이터가 삭제되기도 전에 동일한 유니크 값을 가진 새로운 데이터가 삽입되면서 제약 조건 위반 오류가 발생하게 된다.

### 해결 방안
- 기존 데이터를 조회하여 적절히 `save`/`update`/`delete`를 진행한다.
- `delete`와 `save` 사이에 명시적으로 `flush()`를 호출하여 쌓여있는 쿼리를 실행한다.
- `delete`와 `save`의 트랜잭션을 분리한다.

---
**Reference**
- https://popo-se.tistory.com/27
- https://code-boki.tistory.com/266
