# 일반 Join, Fetch Join 차이점

## 요약
- **일반 Join**: Join 조건을 제외하고 실제 질의하는 대상 Entity에 대한 컬럼만 Select
- **Fetch Join**: 실제 질의하는 대상 Entity와 Fetch Join이 걸려있는 Entity를 포함한 컬럼을 함께 Select

## Join, Fetch Join 차이점 테스트
예제와 함께 일반 Join과 Fetch Join의 차이를 알아보자.

Team과 Member는 1:N 관계이다.

```java
@Entity
@Getter
@ToString
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY, cascade = CascadeType.PERSIST)
    private List<Member> members = new ArrayList<>();

    public Team() {
    }

    public Team(String name) {
        this.name = name;
    }

    public void addMember(Member member) {
        member.setTeam(this);
        members.add(member);
    }
}
```

```java
@Entity
@Getter
@ToString(exclude = "team")
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member() {
    }

    public Member(String name) {
        this.name = name;
    }

    public void setTeam(Team team) {
        this.team = team;
    }
}
```

## FROM절에 1인 Team 사용

### 일반 Join
```java
public interface TeamRepository extends JpaRepository<Team, Long> {
    @Query("SELECT t FROM Team t JOIN t.members")
    List<Team> findAllWithMemberUsingJoin();
}
```
Team의 컬럼만 Select 한다.
```
select
    t1_0.id,
    t1_0.name
from
    team t1_0
join
    member m1_0
        on t1_0.id=m1_0.team_id
```
Member 조회 시 추가 쿼리가 발생한다.
```
select
    m1_0.team_id,
    m1_0.id,
    m1_0.name
from
    member m1_0
where
    m1_0.team_id=3
```

### Fetch Join
```java
public interface TeamRepository extends JpaRepository<Team, Long> {
    @Query("SELECT t FROM Team t JOIN FETCH t.members")
    List<Team> findAllWithMemberUsingFetchJoin();
}
```
Team과 Member의 컬럼을 모두 Select 한다.
```
select
    t1_0.id,
    m1_0.team_id,
    m1_0.id,
    m1_0.name,
    t1_0.name
from
    team t1_0
join
    member m1_0
        on t1_0.id=m1_0.team_id
```

## FROM절에 N인 Member 사용
### 일반 Join + Member만 Select
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("SELECT m FROM Member m JOIN m.team t")
    List<Member> findAllWithTeamUsingJoin();
}
```
Member의 컬럼만 Select 한다.
```
select
    m1_0.id,
    m1_0.name,
    m1_0.team_id
from
    member m1_0
join
    team t1_0
        on t1_0.id=m1_0.team_id
```
Team 조회 시 추가 쿼리가 발생한다.
```
select
    t1_0.id,
    t1_0.name
from
    team t1_0
where
    t1_0.id=9
```
### 일반 Join + Member와 Team Select
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("SELECT m, t FROM Member m JOIN m.team t")
    List<Member> findAllWithTeamUsingJoinSelect();
}
```
Join 대상이 1인 Team이기 때문에 한번의 쿼리로 조회해 올 수 있다.
```
select
    m1_0.id,
    m1_0.name,
    m1_0.team_id,
    t1_0.id,
    t1_0.name
from
    member m1_0
join
    team t1_0
        on t1_0.id=m1_0.team_id
```

### Fetch Join
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("SELECT m FROM Member m JOIN FETCH m.team t")
    List<Member> findAllWithTeamUsingFetchJoin();
}
```
Member와 Team의 컬럼을 모두 Select 한다.
```
select
    m1_0.id,
    m1_0.name,
    t1_0.id,
    t1_0.name
from
    member m1_0
join
    team t1_0
        on t1_0.id=m1_0.team_id
```

---
**Reference**
- https://escapefromcoding.tistory.com/709#-from%EC%A0%88%EC%97%90-n%EC%9D%B8-book-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0
- https://cobbybb.tistory.com/18
