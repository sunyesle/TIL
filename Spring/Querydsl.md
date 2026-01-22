# Querydsl
Querydsl은 JPQL 쿼리를 Java 코드로 Type-safe하게 작성할 수 있도록 도와주는 라이브러리이다.

## 의존성
Spirng Boot 4.0.1, Gradle 9.2.1 기준으로 작성되었다. Querydsl 설정 방법은 Gradle 및 IntelliJ 버전에 따라 다를 수 있다.
> build.gradle
```gradle
ext {
	set('queryDslVersion', "5.1.0")
}

dependencies {
	implementation "com.querydsl:querydsl-jpa:${queryDslVersion}:jakarta"
	annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}:jakarta"
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"
}

// Querydsl generated 경로 설정
def querydslDir = 'build/generated/querydsl'

tasks.withType(JavaCompile) {
	options.generatedSourceOutputDirectory = file(querydslDir)
}
```

## 예제 도메인 모델
<img width="381" height="171" alt="예제 도메인 모델" src="https://github.com/user-attachments/assets/a4799a1a-f451-4599-b7bd-990f0b322f00" />

```java
@Entity
@Table(name = "member")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(exclude = {"team"})
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String name;
    private Integer age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String name, Integer age, Team team) {
        this.name = name;
        this.age = age;
        this.team = team;
    }
}
```
```java
@Entity
@Table(name = "team")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(exclude = {"members"})
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }
}
```

## Q 클래스 생성
빌드를 진행하면 `build/generated/querydsl` 디렉토리 하위에 `Entity`로 등록한 클래스들이 `Q`라는 접두사가 붙은 형태로 생성된 것을 확인할 수 있다.

이러한 클래스들을 **Q 클래스** 또는 **Q 타입**이라고 하며, 이를 통해 Type-safe하게 쿼리를 작성할 수 있다.

## JPAQueryFactory 빈 등록
`JPAQueryFactory`는 Querydsl의 핵심 클래스로 이를 통해 쿼리를 생성할 수 있다.

빈으로 등록하여 필요한 곳에서 주입받아 사용하면 편리하다.
```java
@Configuration
public class QuerydslConfig {
    @PersistenceContext
    private EntityManager em;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(em);
    }
}
```

## 테스트용 JPAQueryFactory 및 커스텀 어노테이션
`@DataJpaTest`는 Querydsl의 `JPAQueryFactory`를 빈으로 등록해 주지 않는다.

테스트용 설정 클래스와 커스텀 어노테이션을 활용하여,
`@DataJpaQuerydslTest` 어노테이션 선언만으로 Querydsl 테스트 환경을 구축할 수 있다.
```java
@TestConfiguration
public class TestQuerydslConfig {
    @PersistenceContext
    private EntityManager em;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(em);
    }
}
```
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@DataJpaTest
@Import(TestQuerydslConfig.class)
public @interface DataJpaQuerydslTest {
}
```

## JPQL vs Querydsl
### JPQL
- 문자열 기반으로 쿼리를 작성한다.
- 오타나 잘못된 쿼리 작성으로 인해 런타임 오류가 발생할 수 있다.
- 복잡한 동적 쿼리 작성이 어렵다.
```java
List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
        .setParameter("name", "member1")
        .getResultList();
```

### Querydsl
- SQL과 유사한 메서드 체이닝 방식으로 쿼리를 작성한다.
- 컴파일 타임에 오류를 감지할 수 있다.
- IDE의 자동 완성 기능을 활용할 수 있다.
- 효율적으로 동적 쿼리를 작성할 수 있다.
- 엔티티 필드 변경 시 Q 클래스를 통해 관련 쿼리에도 반영되므로 유지보수가 편리하다.
```java
QMember member = QMember.member;

List<Member> result = queryFactory
        .select(member)
        .from(member)
        .where(member.name.eq("member1"))
        .fetch();
```

## Q 클래스 사용 방법
### 기본 인스턴스 사용
기본 인스턴스를 `static Import` 하는 방식이 가장 많이 사용된다.
```java
import static com.sunyesle.spring_boot_jpa_querydsl.entity.QMember.member;
```

다음과 같이 변수에 할당하여 사용할 수도 있다.
```java
QMember m = QMember.member;
```

### 별칭 직접 지정
셀프 조인이나 서브쿼리 등 하나의 쿼리 내에서 같은 테이블을 여러 번 불러와야 하는 상황에 사용된다.
```java
QMember m1 = new QMember("m");
```

## 조회
- `select()`: 조회할 컬럼
- `from()`: 조회 대상 엔티티

`select()`와 `from()`의 내용이 동일할 경우 `selectFrom()`으로 축약할 수 있다.

## 검색 조건
`where()`
- 파라미터로 여러 개의 조건을 받을 수 있다. 각 조건은 **AND**로 결합된다.
- 파라미터로 전달된 조건 중 **null 값은 무시**되며, 이 특징을 이용해 동적 쿼리를 깔끔하게 작성할 수 있다.
```java
List<Member> result = queryFactory
        .selectFrom(member)
        .where(
                member.name.eq("member1"),
                member.age.eq(10)
        )
        .fetch();
```

### 복합 조건
`and()`, `or()`를 이용하여 조건을 조합할 수 있다.
```java
List<Member> result = queryFactory
        .selectFrom(member)
        .where(
                member.name.eq("member1").or(member.name.eq("member2"))
        )
        .fetch();
```

### 검색 조건 목록
```java
member.name.eq("member"),       // name = 'member'
member.name.ne("member"),       // name != 'member'
member.name.eq("member").not(), // name != 'member'

member.name.isNull(),    // name is null
member.name.isNotNull(), // name is not null

member.age.in(10, 20),    // age in (10, 20)
member.age.notIn(10, 20), // age not in (10, 20)

member.age.between(10, 30), // age between 10 and 30

member.age.goe(30), // age >= 30
member.age.gt(30),  // age > 30
member.age.loe(30), // age <= 30
member.age.lt(30),  // age < 30

member.name.like("member%"),      // name like 'member%'
member.name.contains("member"),   // name like '%member%'
member.name.startsWith("member"), // name like 'member%'
member.name.endsWith("member")    // name like '%member'
```

## 결과 조회
### fetch()
리스트 조회. 결과가 없으면 빈 리스트를 반환한다.
```java
List<Member> result = queryFactory
        .selectFrom(member)
        .fetch();
```

### fetchOne()
단건 조회. 결과가 없으면 null을 반환하고, 결과가 여러 건이면 `NonUniqueResultException`이 발생한다.
```java
Member result = queryFactory
        .selectFrom(member)
        .where(member.name.eq("member3"))
        .fetchOne();
```

### fetchFirst()
가장 상위 데이터 조회. `limit(1).fetchOne()`과 동일하다.
```java
Member result = queryFactory
        .selectFrom(member)
        .fetchFirst();
```

## 정렬
- `orderBy()`: 해당 컬럼을 정렬조건에 따라 정렬 (기본값 asc)
- `asc()`, `desc()`: 정렬 순서
- `nullsLast()`, `nullsFirst()`: null 데이터 순서
```java
List<Member> result = queryFactory
        .selectFrom(member)
        .orderBy(
                member.age.desc(),
                member.name.asc().nullsFirst()
        )
        .fetch();
```

## 페이징
- `offset()`: 조회할 데이터의 시작 지점 (0부터 시작)
- `limit()`: 조회할 데이터 양
```java
int offset = 0;
int limit = 10;

List<Member> content = queryFactory
        .selectFrom(member)
        .orderBy(member.name.desc())
        .offset(offset)
        .limit(limit)
        .fetch();

Long count = queryFactory
        .select(member.count())
        .from(member)
        .fetchOne();
```

## 집계함수
- `count()`: 행 갯수
- `sum()`: 합
- `avg()`: 평균
- `max()`: 최대
- `min()`: 최소
```java
List<Tuple> result = queryFactory
        .select(member.count(),
                member.age.sum(),
                member.age.avg(),
                member.age.max(),
                member.age.min())
        .from(member)
        .fetch();
```
> `Tuple`은 Querydsl이 제공하는 클래스로 데이터 타입에 관계없이 결과값을 받을 수 있다.<br>
> 실무에서는 `Tuple` 대신 DTO로 받아 사용하는 것을 권장한다.(뒷 부분에서 다룰 예정)

## 그룹화
- `groupBy()`: 해당 컬럼을 기준으로 그룹화한다.
- `having()`: 특정 조건을 만족하는 그룹을 필터링한다.
```java
List<Tuple> result = queryFactory
        .select(member.count(),
                member.age.sum(),
                member.age.avg(),
                member.age.max(),
                member.age.min())
        .from(member)
        .join(member.team, team)
        .groupBy(team.name)
        .having(member.age.avg().gt(30))
        .fetch();
```

## 조인
`.join(조인 대상, 별칭)`

### innerJoin()
```java
List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .join(member.team, team) // 또는 .innerJoin()
        .fetch();
```

### leftJoin()
```java
List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(member.team, team)
        .fetch();
```

### rightJoin()
```java
List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .rightJoin(member.team, team)
        .fetch();
```

## 조인 - ON 절
`on()`은 다음과 같은 상황에 사용한다.
### 조인 대상을 필터링할 때
외부 조인 시에는 on 절에 필터링 조건을 넣으면 조인 대상 테이블의 데이터만 걸러지고, 기준 테이블은 유지된다.
내부 조인 시에는 where 절에서 필터링하는 것과 동일하므로, 가독성을 위해 where을 선호한다.
```java
List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(member.team, team).on(team.name.eq("TeamA")) // 외부조인. 특정 조건을 가진 데이터만 조인.
        .fetch();
```
### 연관 관계가 없는 엔티티 간의 조인 (Theta Join)
```java
List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(team).on(member.name.eq(team.name))
        .fetch();
```

## 조인 - Fetch Join
SQL 조인을 활용해서 연관된 엔티티 혹은 컬렉션들을 한번에 조회하는 기능이다.

`join()`, `leftJoin()` 등의 조인 기능 뒤에 `fetchJoin()`을 추가하면 된다.
```java
List<Member> result = queryFactory
        .selectFrom(member)
        .join(member.team, team)
        .fetchJoin()
        .fetch();
```

## 서브쿼리
`JPAExpressions`을 사용하여 서브쿼리를 작성할 수 있다.
### 중첩된 서브쿼리(Nested Subquery) : WHERE 절, HAVING 절의 서브쿼리
```java
QMember memberSub = new QMember("memberSub");

// eq
List<Member> result1 = queryFactory
        .selectFrom(member)
        .where(member.age.eq(
                        JPAExpressions
                                .select(memberSub.age.max())
                                .from(memberSub)
                )
        )
        .fetch();

// in
List<Member> result2 = queryFactory
        .selectFrom(member)
        .where(member.age.in(
                        JPAExpressions
                                .select(memberSub.age)
                                .from(memberSub)
                                .where(memberSub.age.gt(10))
                )
        )
        .fetch();

// exists
List<Member> result3 = queryFactory
        .selectFrom(member)
        .where(
                JPAExpressions
                        .selectFrom(memberSub)
                        .where(
                                memberSub.id.eq(member.id),
                                memberSub.team.name.eq("TeamA"))
                        .exists()
        )
        .fetch();

```

### 스칼라 서브쿼리(Scalar Subquery) : SELECT 절의 서브쿼리
```java
QMember memberSub = new QMember("memberSub");

List<Tuple> result = queryFactory
        .select(
                member.name,
                ExpressionUtils.as(
                        JPAExpressions
                                .select(memberSub.age.avg())
                                .from(memberSub)
                        , "avg")
        )
        .from(member)
        .fetch();
```

### 인라인 뷰(Inline View) : FROM 절의 서브쿼리
Querydsl은 기본적으로 JPQL의 표준 사양을 따르기 때문에 인라인뷰를 지원하지 않는다.

**대안**
- 서브쿼리를 조인으로 변경한다.
- 애플리케이션에서 쿼리를 2번으로 나누어 실행한다.
- Native Query를 사용한다.
