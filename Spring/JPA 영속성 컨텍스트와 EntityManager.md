# JPA 영속성 컨텍스트와 EntityManager
## EntityManagerFactory
`EntityManager`를 만드는 공장으로 Thread-safety 하게 설계되었다.

생성 비용이 크기 때문에, 하나의 DB당 하나의 `EntityManagerFactory`를 생성하고 이를 애플리케이션 전체에 공유하여 사용한다.
만약, 멀티 데이터베이스 환경이라면 `DataSource`마다 연결되는 `EntityManagerFactory`를 각각 설정해야 한다.

## EntityManager
특정 트랜잭션 단위에서 엔티티에 대한 작업을 수행하는 데 사용하며, Thread-safety 하지 않다.

`EntityManager`를 통해 **영속성 컨텍스트에 접근하여 엔티티를 관리**할 수 있다.

### 예시
```java
void main() {
    // [엔티티 매니저 팩토리] - 생성
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa_test");

    // [엔티티 매니저] - 생성
    EntityManager em = emf.createEntityManager();

    // [트랜잭션] - 획득
    EntityTransaction tx = em.getTransaction();

    try {
        tx.begin();    // [트랜잭션] - 시작

        // 비즈니스 로직 실행
        logic(em);     

        tx.commit();   // [트랜잭션] - 커밋
    } catch (Exception e) {
        tx.rollback(); // [트랜잭션] - 롤백
    } finally {
        em.close(); // [엔티티 매니저] - 종료
    }

    emf.close(); // [엔티티 매니저 팩토리] - 종료
}

// EntityManager를 통해 엔티티를 관리하고 CRUD 작업을 수행한다.
private void logic(EntityManager em) {
    Member member = new Member();
    member.setId(1L);
    member.setName("test");
    member.setAge(25);

    // 등록
    em.persist(member);

    // 수정
    member.setAge(26);

    // 단 건 조회
    Member findMember = em.find(Member.class, 1L);

    // 목록 조회
    List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

    // 삭제
    em.remove(member);
}
```

## 영속성 컨텍스트 (Persistence Context)
**논리적인 개념**으로 엔티티를 영구 저장하는 환경을 말한다.
애플리케이션과 데이터베이스 사이에서 객체를 보관하는 **가상의 데이터베이스 역할**을 한다.

<img width="521" height="301" alt="EntityManager" src="https://github.com/user-attachments/assets/9041507c-b459-43d0-9fc9-9a86ac1eab17" />

### 엔티티의 생명주기
다음 4가지 생명주기를 바탕으로 `EntityManager`를 통해 엔티티를 관리한다.

<img width="521" height="361" alt="JPA 엔티티 생명주기" src="https://github.com/user-attachments/assets/1879e523-02e3-4b54-aab8-029ee65174aa" />

1. **비영속 상태(New)**
   - 엔티티가 생성은 되었지만, 영속성 컨텍스트와 관련이 없는 상태이다.
2. **영속 상태(Managed)**
   - 비영속 상태에서 `persist()`를 통해 엔티티가 영속성 컨텍스트에 들어와 `EntityManager`에 의해 관리되고 있는 상태이다.
3. **준영속 상태(Detached)**
   - 엔티티가 영속성 컨텍스트에 의해 관리되고 있다가, `detach()`, `clear()`, `close()` 등으로 관리 대상에서 분리된 상태이다. `merge()`를 통해 다시 영속 상태로 변경할 수 있다.
4. **삭제된 상태(Removed)**
   - 엔티티가 영속성 컨텍스트에 의해 관리되고 있다가, `remove()`를 통해 삭제된 상태이다.

### 영속성 컨텍스트의 주요 기능

<img width="611" height="381" alt="영속성 컨텍스트" src="https://github.com/user-attachments/assets/2b2536a8-c81a-405f-9bdc-9aace90ee191" />

**- 1차 캐시**<br>
영속성 컨텍스트는 내부에 캐시를 가지고 있으며, 이를 1차 캐시라고 한다. 영속 상태의 엔티티는 `Map<K, V>` 형태로 모두 여기에 저장된다.
- **Key**: `@Id`로 매핑한 식별자 값
- **Value**: Entity 인스턴스

엔티티를 식별자 값으로 구분하므로, 영속 상태는 반드시 식별자 값이 있어야 한다.

`find()` 호출 시 영속성 컨텍스트의 1차 캐시를 먼저 조회한다.
엔티티가 1차 캐시에 존재하지 않으면, 데이터베이스를 조회하여 엔티티 생성 후 1차 캐시에 저장한 후에 영속 상태의 엔티티를 반환한다.

**- 동일성 보장**<br>
영속성 컨텍스트는 1차 캐시에 있는 같은 엔티티 인스턴스를 반환하므로, 엔티티의 동일성(identity)을 보장한다.

**- 트랜잭션을 지원하는 쓰기 지연** (**Transactional write-behind**)<br>
영속성 컨텍스트는 엔티티의 변경 사항을 즉시 데이터베이스에 반영하지 않고,
쓰기 지연 SQL 저장소에 모아두었다가 커밋되는 시점에 모아둔 SQL을 데이터베이스에 전달한다.

**- 변경 감지** (**dirty checking**)<br>
엔티티를 영속성 컨텍스트에 보관할 때 최초 상태를 복사해서 저장해둔다. (스냅샷)
엔티티와 스냅샷을 비교하여 변경된 내용이 있다면 자동으로 UPDATE SQL을 생성한다.

### Flush
**flush**는 **영속성 컨텍스트의 변경사항을 데이터베이스에 반영**(동기화)하는 과정이다.

`flush()` 호출 시, 쓰기 지연 SQL 저장소에 모아두었던 쿼리를 데이터베이스에 전달한다. DB 트랜잭션 커밋과는 별개의 작업이며, 영속성 컨텍스트(1차 캐시)도 그대로 유지된다.

Hibernate 기본값인 `FlushModeType.AUTO` 기준으로 `flush()`가 자동 호출되는 상황은 다음과 같다.
- **트랜잭션을 커밋하기 전**: 트랜잭션을 끝내기 전에 영속성 컨텍스트의 변경 사항을 모두 반영하기 위해 `flush()` 호출
- **JPQL 실행 전**: 영속성 컨텍스트에 대기중인 작업(`INSERT`, `UPDATE`, `DELETE`)과 JPQL로 조회하려는 대상이 겹치는 경우 `flush()` 호출
- **Native SQL 실행 전**: 항상 `flush()` 호출

---
**Reference**
- https://dahye-jeong.gitbook.io/spring/spring/2020-04-11-jpa-basic/2022-03-12-persistence-context
- https://psvm.kr/posts/tutorials/jpa/3-em-and-persistence-context
- https://docs.hibernate.org/orm/7.2/userguide/html_single/#flushing-auto
