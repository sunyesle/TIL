# Bulk Insert로 대량의 데이터 저장하기
사용 기술(Spring Data JPA, Spring Data JDBC)과 ID 생성 전략(IDENTITY, UUID)에 따른 Bulk Insert 방법을 알아보자.

## Bulk Insert 설정
### MySQL JDBC 드라이버 설정
> application.yml
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/my_db?rewriteBatchedStatements=true&profileSQL=false&logger=Slf4jLogger&maxQuerySizeToLog=999999
    username: user
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
```
JDBC URL에 파라미터를 추가한다.
- `rewriteBatchedStatements=true` 
   - 배치 처리를 위한 필수 파라미터. 여러 개의 `INSERT` 문을 `INSERT INTO ... VALUES (...), (...)` 형태로 합쳐준다.
- `profileSQL=true&logger=Slf4jLogger&maxQuerySizeToLog=999999`
   - 쿼리 재작성은 드라이버 내부에서 일어난다. 이 옵션을 통해 실제 DB로 전송되는 쿼리를 확인할 수 있다. (개발 환경용)

### JPA Bulk Insert 설정
> application.yml
```yml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        jdbc:
          batch_size: 1000
          batch_versioned_data: true
        order_inserts: true
        order_updates: true
```
- `batch_size`: 일괄 처리할 쿼리의 최대 개수
- `batch_versioned_data`: 버전이 지정된 데이터(Optimistic Lock)도 배치로 처리할지 여부
- `order_inserts`, `order_updates`: 엔티티별로 insert/update 순서 정렬하여 JDBC 레벨에서 배치를 묶기 쉽도록 한다. 단, 정렬로 인한 성능 저하가 발생할 수 있으므로 벤치마크가 필요하다.

## ID 생성 전략이 IDENTITY인 경우
예시 코드에서 사용할 테이블 정보와 엔티티이다. ID 생성 전략은 `IDENTITY`로, MySQL의 `AUTO_INCREMENT`를 사용한다.
```sql
CREATE TABLE team_identity (
    team_id BIGINT AUTO_INCREMENT,
    name VARCHAR(255),
    PRIMARY KEY (team_id)
);

CREATE TABLE member_identity (
    member_id BIGINT AUTO_INCREMENT,
    name VARCHAR(255),
    age INT,
    team_id BIGINT,
    PRIMARY KEY (member_id)
);
```

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(exclude = {"members"})
@Table(name = "team_identity")
public class TeamIdentity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team", cascade = CascadeType.PERSIST, fetch = FetchType.LAZY)
    List<MemberIdentity> members = new ArrayList<>();

    public TeamIdentity(String name) {
        this.name = name;
    }

    public void addMember(MemberIdentity member) {
        members.add(member);
        member.setTeam(this);
    }
}

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(exclude = {"team"})
@Table(name = "member_identity")
public class MemberIdentity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String name;
    private Integer age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private TeamIdentity team;

    public MemberIdentity(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    protected void setTeam(TeamIdentity team) {
        this.team = team;
    }
}
```

### JPA의 한계
IDENTITY 전략은 ID 생성을 DB에 위임하는 방식이다.
엔티티가 영속 상태가 되려면 ID가 있어야되는데 쿼리가 DB에서 실행되기 전까지는 ID를 알 수 없기 때문에, 쓰기 지연(Write-behind) 없이 쿼리가 실행된다.

ID 생성 전략이 IDENTITY인 경우 쓰기 지연이 불가능하기 때문에, INSERT 배치 처리가 불가능하다.

### JDBC로 단일 테이블 Bulk Insert
JDBC를 사용하면 Bulk Insert가 가능하다.
```java
@Autowired
NamedParameterJdbcTemplate jdbc;

@Test
void jdbc_identity_single() {
    List<TeamRequest> teamRequests = TeamFixture.createTeamRequest(1000, 0);

    SqlParameterSource[] batchArgs = teamRequests.stream()
            .map(m -> new MapSqlParameterSource()
                    .addValue("name", m.name()))
            .toArray(SqlParameterSource[]::new);

    jdbc.batchUpdate("INSERT INTO team_identity (name) VALUES (:name)", batchArgs);
}
```

### JDBC로 부모-자식 테이블 Bulk Insert
Team과 Member를 함께 넣어야 하는 상황이라면 복잡해진다.

`Team`의 ID를 가져와 `Member`의 FK로 넣어줘야 하는데, `Team`을 저장하기 전에는 ID를 알 수 없으므로 다음과 같은 과정이 필요하다.
1. 부모(`Team`) 데이터 세팅
2. 부모 데이터를 저장한다. 이때, 생성된 키들을 담을 `KeyHolder`를 인자로 전달한다.
3. 자식(`Member`) 데이터 세팅. 추출한 부모 ID를 자식 객체의 FK(`teamId`)로 설정한다.
4. 자식 데이터를 저장한다.

> `KeyHolder`는 자동 생성된 키 값을 가져올 때 사용하는 인터페이스이다. JDBC 드라이버 구현체에 따라 해당 기능을 지원하지 않을 수도 있다.

```java
@Test
void jdbc_identity_relation() {
    List<TeamRequest> teamRequests = TestFixture.createTeamRequest(100, 2);

    // Team 데이터 세팅
    SqlParameterSource[] teamBatchArgs = teamRequests.stream()
            .map(m -> new MapSqlParameterSource()
                    .addValue("name", m.name()))
            .toArray(SqlParameterSource[]::new);

    // AUTO_INCREMENT로 생성된 PK 값을 담기 위한 KeyHolder 준비
    KeyHolder keyHolder = new GeneratedKeyHolder();

    // Team 데이터 저장
    jdbcTemplate.batchUpdate("INSERT INTO team_identity (name) VALUES (:name)", teamBatchArgs, keyHolder);

    // 생성된 PK 값을 가져옴
    List<Map<String, Object>> keyList = keyHolder.getKeyList();

    // Member 데이터 세팅
    List<MapSqlParameterSource> memberParamList = new ArrayList<>();
    for (int i = 0; i < teamRequests.size(); i++) {
        Number key = (Number) keyList.get(i).values().iterator().next();
        long teamId = key.longValue();

        TeamRequest team = teamRequests.get(i);
        for (MemberRequest member : team.members()) {
            memberParamList.add(new MapSqlParameterSource()
                    .addValue("name", member.name())
                    .addValue("age", member.age())
                    .addValue("teamId", teamId));
        }
    }

    // Member 데이터 저장
    jdbcTemplate.batchUpdate("INSERT INTO member_identity (name, age, team_id) VALUES (:name, :age, :teamId)",
            memberParamList.toArray(SqlParameterSource[]::new)
    );
}
```

## ID 생성 전략이 UUID인 경우
예시 코드에서 사용할 테이블 정보와 엔티티이다. ID로 UUID를 사용하며 애플리케이션 단에서 ID를 부여한다.
```java
CREATE TABLE team_uuid (
    team_id BINARY(16) NOT NULL,
    name VARCHAR(255),
    PRIMARY KEY (team_id)
);

CREATE TABLE member_uuid (
    member_id BINARY(16) NOT NULL,
    name VARCHAR(255),
    age INT,
    team_id BINARY(16),
    PRIMARY KEY (member_id)
);
```
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(exclude = {"members", "isNew"})
@Table(name = "team_uuid")
public class TeamUuid implements Persistable<UUID> {
    @Id
    @Column(name = "team_id", columnDefinition = "BINARY(16)")
    private UUID id;

    private String name;

    @OneToMany(mappedBy = "team", cascade = CascadeType.PERSIST, fetch = FetchType.LAZY)
    List<MemberUuid> members = new ArrayList<>();

    @Transient
    private boolean isNew = true;

    @Override
    public boolean isNew() { return isNew; }

    @PostPersist
    @PostLoad
    protected void markNotNew() { this.isNew = false; }

    public TeamUuid(String name) {
        this.id = UuidUtil.generateUuid();
        this.name = name;
    }

    public void addMember(MemberUuid member) {
        members.add(member);
        member.setTeam(this);
    }
}

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(exclude = {"team", "isNew"})
@Table(name = "member_uuid")
public class MemberUuid implements Persistable<UUID> {
    @Id
    @Column(name = "member_id", columnDefinition = "BINARY(16)")
    private UUID id;

    private String name;
    private Integer age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private TeamUuid team;

    @Transient
    private boolean isNew = true;

    @Override
    public boolean isNew() { return isNew; }

    @PostPersist
    @PostLoad
    protected void markNotNew() { this.isNew = false; }

    public MemberUuid(String name, Integer age) {
        this.id = UuidUtil.generateUuid();
        this.name = name;
        this.age = age;
    }

    protected void setTeam(TeamUuid team) {
        this.team = team;
    }
}
```

MySQL에서 랜덤 방식 UUID(v4)를 PK로 사용할 경우, 인덱스 페이지 파편화로 인해 성능 저하가 발생한다.
따라서, 시간 기반 UUID(v7)를 사용하여 데이터들이 순차적으로 삽입될 수 있도록 하였다.

java 표준 라이브러리에서는 UUID v7 생성을 지원하지 않아서 다음 라이브러리를 추가하였다.
```gradle
dependencies {
    implementation 'com.fasterxml.uuid:java-uuid-generator:5.2.0'
}
```
```java
public class UuidUtil {
    // UUID v7 생성
    public static UUID generateUuid() {
        return Generators.timeBasedEpochGenerator().generate();
    }

    // UUID를 binary(16)으로 변환
    public static byte[] toBytes(UUID uuid) {
        ByteBuffer bb = ByteBuffer.wrap(new byte[16]);
        bb.putLong(uuid.getMostSignificantBits());
        bb.putLong(uuid.getLeastSignificantBits());
        return bb.array();
    }
}
```

### JPA로 단일 테이블 Bulk Insert
IDENTITY 전략이 아닌 경우, JPA로도 Bulk Insert가 가능하다.
```java
@Test
void jpa_uuid_single() {
    List<TeamRequest> teamRequests = TestFixture.createTeamRequest(100, 0);

    List<TeamUuid> teams = teamRequests.stream()
            .map(t -> new TeamUuid(t.name()))
            .collect(Collectors.toList());

    teamUuidRepository.saveAll(teams);
    teamUuidRepository.flush();
}
```

### JPA로 부모-자식 테이블 Bulk Insert
기본적으로 부모와 자식을 한 번에 `saveAll()` 할 때, hibernate에서 생성하는 쿼리 순서는 다음과 같다.
```sql
insert into 부모 values (1)
insert into 자식 values (1-1)
insert into 자식 values (1-2)
insert into 부모 values (2)
insert into 자식 values (2-1, 2)
```
문제는 쿼리의 종류가 달라지면 배치가 끊긴다는 점이다. 즉, 위 쿼리는 다음과 같은 형태로 묶이게 된다.
```sql
insert into 부모 values (1)
insert into 자식 values (1-1), (1-2)
insert into 부모 values (2)
insert into 자식 values (2-1)
```

이때, `spring.jpa.properties.hibernate.order_inserts=true` 옵션을 사용하면 hibernate가 insert 쿼리를 엔티티 별로 정렬하여, 효율적인 배치 형태로 묶일 수 있게 한다.
```sql
insert into 부모 values (1), (2)
insert into 자식 values (1-1), (1-2), (2-1)
```
다만, [공식 문서](https://docs.hibernate.org/orm/6.4/userguide/html_single/#batch-jdbcbatch)에 해당 옵션은 성능 저하가 발생할 수 있으므로 벤치마킹을 통해 실제로 성능 향상에 도움이 되는지를 확인해야 한다고 나와 있다.
정렬을 위해 모든 데이터를 메모리에 쌓아두는 방식으로 동작하므로, 한 번에 대량의 데이터를 저장하는 경우 주의가 필요하다. (`batch_size`와는 무관하다.)

```java
@Test
void jpa_uuid_relation() {
    List<TeamRequest> teamRequests = TestFixture.createTeamRequest(100, 2);

    List<TeamUuid> teams = teamRequests.stream()
            .map(request -> {
                // Team 생성
                TeamUuid team = new TeamUuid(request.name());

                // Member 생성 및 연관 관계 설정
                request.members().forEach(m -> {
                    MemberUuid member = new MemberUuid(m.name(), m.age());
                    team.addMember(member); // Team 내부 리스트에 추가
                });

                return team;
            })
            .collect(Collectors.toList());

    teamUuidRepository.saveAll(teams);
    teamUuidRepository.flush();
}
```

### JDBC로 단일 테이블 bulk insert
```java
@Test
void jdbc_uuid_single() {
    List<TeamRequest> teamRequests = TestFixture.createTeamRequest(100, 0);

    SqlParameterSource[] batchArgs = teamRequests.stream()
            .map(m -> new MapSqlParameterSource()
                    .addValue("teamId", UuidUtil.toBytes(UuidUtil.generateUuid()))
                    .addValue("name", m.name()))
            .toArray(SqlParameterSource[]::new);

    jdbcTemplate.batchUpdate("INSERT INTO team_uuid (team_id, name) VALUES (:teamId, :name)", batchArgs);
}
```

### JDBC로 부모-자식 테이블 bulk insert
IDENTITY 전략에서는 부모 데이터 저장 후 `KeyHolder`를 통해 ID를 받아오는 로직이 필수적이었으나,
UUID 방식을 사용하면 ID를 애플리케이션에서 직접 생성하기 때문에 부모와 자식 연관 관계를 미리 확정 지을 수 있다.
```java
@Test
void jdbc_uuid_relation() {
    List<TeamRequest> teamRequests = TestFixture.createTeamRequest(100, 2);

    List<MapSqlParameterSource> teamParamList = new ArrayList<>();
    List<MapSqlParameterSource> memberParamList = new ArrayList<>();

    for (TeamRequest teamDto : teamRequests) {
        byte[] teamId = UuidUtil.toBytes(UuidUtil.generateUuid());

        teamParamList.add(new MapSqlParameterSource()
                .addValue("teamId", teamId)
                .addValue("name", teamDto.name())
        );

        for (MemberRequest memberDto : teamDto.members()) {
            memberParamList.add(new MapSqlParameterSource()
                    .addValue("memberId", UuidUtil.toBytes(UuidUtil.generateUuid()))
                    .addValue("age", memberDto.age())
                    .addValue("name", memberDto.name())
                    .addValue("teamId", teamId)
            );
        }
    }

    jdbcTemplate.batchUpdate("INSERT INTO team_uuid (team_id, name) VALUES (:teamId, :name)",
            teamParamList.toArray(SqlParameterSource[]::new)
    );

    jdbcTemplate.batchUpdate("INSERT INTO member_uuid (member_id, name, age, team_id) VALUES (:memberId, :name, :age, :teamId)",
            memberParamList.toArray(SqlParameterSource[]::new)
    );
}
```

## 정리
### JPA
- 엔티티의 비즈니스 로직을 활용할 수 있어서 객체지향적 개발과 유지보수에 유리하다.
- JDBC와 비교했을 때 상대적으로 느리다.
- ID 생성 전략이 IDENTITY인 경우 배치 처리가 불가능하다.

### JDBC
- 속도가 빠르다.
- SQL을 직접 작성하므로 쿼리 튜닝에 유리하다.
- SQL 작성, 매개변수 바인딩, 결과 매핑 등을 수동으로 처리해야 하므로, 생산성이 낮고 유지보수가 어렵다.

### 결론
- 엔티티 상태 변경 작업인 경우 기존 비즈니스 로직을 활용할 수 있는 JPA를 사용한다. (IDENTITY 전략일 때는 제외)
- 대량 배치 처리와 같이 성능 최적화가 필요한 경우 JDBC를 사용한다.
- 경우에 따라 데이터 조회에는 Querydsl을 사용해 유지보수성을 높이고, 저장에는 JDBC를 사용하는 식으로 혼합하여 사용하는 것도 가능해 보인다.

---
**Reference**
- https://hstory0208.tistory.com/entry/Batch-Insert-설정-이해하기-그리고-JPA-JDBC-성능-비교
- https://helloworld.kurly.com/blog/bulk-performance-tuning/
- https://techblog.woowahan.com/2695/
- https://kwonnam.pe.kr/wiki/java/hibernate/batch
- https://cheese10yun.github.io/jpa-batch-insert/
- https://www.nextree.io/jpavsjdbc/
- https://www.borntodare.me/mysql_uuid
