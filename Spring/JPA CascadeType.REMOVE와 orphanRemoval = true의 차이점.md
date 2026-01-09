# CascadeType.REMOVE와 orphanRemoval = true의 차이점

## 요약
| 상황                 | `CascadeType.REMOVE` | `orphanRemoval = true` |
|--------------------|---------------------|-----------------------|
| 부모 삭제              | 자식 삭제               | 자식 삭제                 |
| 부모와 자식 사이의 연관관계 제거 | 아무 일도 안 함           | 자식 삭제                 |

## 예시
Parent와 Child 엔티티를 바탕으로 두 개념의 차이점을 알아보자.
```java
@Entity
@Getter
public class Parent {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "parent", fetch = FetchType.LAZY, cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<>();

    public Parent() {
    }

    public void addChild(Child child){
        child.setParent(this);
        children.add(child);
    }

    public void removeChild(Child child){
        child.setParent(null);
        children.remove(child);
    }
}
```
```java
@Entity
@Getter
public class Child {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Parent parent;

    public Child() {
    }

    public void setParent(Parent parent) {
        this.parent = parent;
    }
}
```

## CascadeType.REMOVE
Cascade 옵션은 부모 엔티티에서 자식 엔티티로 상태 전환(영속성 이벤트)을 전파시킬지 선택하는 옵션이다.

CascadeType.REMOVE를 사용할 경우 부모 엔티티가 삭제되었을 때 상태가 전파되어 자식 엔티티도 함께 삭제된다.

테스트를 통해 CascadeType.REMOVE 옵션에 대해 알아보자.<br>
Team 엔티티의 cascade 옵션에 CascadeType.REMOVE를 추가한다.
```java
@OneToMany(mappedBy = "parent", fetch = FetchType.LAZY, cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
private List<Child> children = new ArrayList<>();
```

### 부모 엔티티 삭제
```java
@Test
void cascadeTypeRemove_removeParent_test(){
    // given
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();

    parent.addChild(child1);
    parent.addChild(child2);

    parentRepository.save(parent);

    entityManager.flush();

    // when
    parentRepository.delete(parent);

    // then
    List<Parent> parents = parentRepository.findAll();
    List<Child> children = childRepository.findAll();

    assertThat(parents).isEmpty();
    assertThat(children).isEmpty();
}
```
실행되는 SQL은 다음과 같다.
```
delete from child where id=1
delete from child where id=2
delete from parent where id=1
```
부모 엔티티를 삭제하면, 자식 엔티티도 함께 삭제된다.

### 부모 엔티티와 자식 엔티티 사이의 연관관계 제거
```java
@Test
void cascadeTypeRemove_removeChild_test(){
    // given
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();

    parent.addChild(child1);
    parent.addChild(child2);

    parentRepository.save(parent);

    entityManager.flush();

    // when
    parent.removeChild(child);
    parent.removeChild(child2);

    // then
    List<Parent> parents = parentRepository.findAll();
    List<Child> children = childRepository.findAll();

    assertThat(parents).isNotEmpty();
    assertThat(children).isNotEmpty();
}
```
실행되는 SQL은 다음과 같다.
```
select p1_0.id from parent p1_0
update child set parent_id=NULL where id=1
update child set parent_id=NULL where id=2
```
부모 엔티티와 자식 엔티티 사이의 연관 관계를 제거하면, 자식 엔티티의 외래키 값이 NULL로 변경된다.

연관 관계가 끊어진 것이 부모 엔티티의 상태가 변화한 것은 아니기 때문에 자식 엔티티가 삭제되지는 않는다.

## orphanRemoval = true
orphanRemoval 옵션은 **고아 객체**를 DB에서 자동으로 삭제할지 선택하는 옵션이다.

**고아 객체**<br>
고아 객체란 부모 엔티티와의 연관 관계가 끊어진 자식 엔티티이다.<br>
다음과 같은 작업으로 인해 고아 객체가 될 수 있다.
- 부모 엔티티를 삭제하는 경우
- 부모 엔티티와 자식 엔티티 사이의 연관 관계를 제거하는 경우

테스트를 통해 orphanRemoval = true 옵션에 대해 알아보자.<br>
Team 엔티티에 orphanRemoval = true를 추가한다.
```java
@OneToMany(mappedBy = "parent", fetch = FetchType.LAZY, cascade = CascadeType.PERSIST, orphanRemoval = true)
private List<Child2> children = new ArrayList<>();
```

### 부모 엔티티 삭제
```java
@Test
void orphanRemovalTrue_removeParent_test(){
    // given
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();

    parent.addChild(child1);
    parent.addChild(child2);

    parentRepository.save(parent);

    entityManager.flush();

    // when
    parentRepository.delete(parent);

    // then
    List<Parent> parents = parentRepository.findAll();
    List<Child> children = childRepository.findAll();

    assertThat(parents).isEmpty();
    assertThat(children).isEmpty();
}
```
실행되는 SQL은 다음과 같다.
```
delete from child where id=1
delete from child where id=2
delete from parent where id=1
```
부모 엔티티를 삭제하면, 자식 엔티티가 고아 객체로 취급되어 삭제된다.

### 부모 엔티티와 자식 엔티티 사이의 연관관계 제거
```java
@Test
void orphanRemovalTrue_removeChild_test(){
    // given
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();

    parent.addChild(child1);
    parent.addChild(child2);

    parentRepository.save(parent);

    entityManager.flush();

    // when
    parent.removeChild(child1);
    parent.removeChild(child2);

    // then
    List<Parent> parents = parentRepository.findAll();
    List<Child> children = childRepository.findAll();

    assertThat(parents).isNotEmpty();
    assertThat(children).isEmpty();
}
```
실행되는 SQL은 다음과 같다.
```
select p1_0.id from parent p1_0
delete from child where id=1
delete from child where id=2
```
부모 엔티티와 자식 엔티티 사이의 연관 관계를 제거하면, 자식 엔티티가 고아 객체로 취급되어 삭제된다.

---
**Reference**<br>
- https://tecoble.techcourse.co.kr/post/2021-08-15-jpa-cascadetype-remove-vs-orphanremoval-true
- https://velog.io/@yuseogi0218/JPA-CascadeType.REMOVE-vs-orphanRemoval-true#orphanremoval--true
