# Oracle Cloud 웹 서버 구축 (5) - Spring Boot - MySQL 연동

## Spring Boot 프로젝트 MySQL 연동 로직 추가
게시글을 조회, 저장하는 로직을 작성할 것이다.

**build.gradle**<br>
Spring Data JPA와 MySQL Driver 그리고 로컬 테스트를 위한 H2 Database 의존성을 추가한다.
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    runtimeOnly 'com.mysql:mysql-connector-j'
    ...
}
```

**application.yml**
```yml
spring:
  profiles:
    active: local

---
spring:
  config:
    activate:
      on-profile: local
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  h2:
    console:
      enabled: true

---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

**JPA Auditing 활성화**
```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

**Entity**
```java
@Entity
@Getter
@AllArgsConstructor
@NoArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String title;

    @Column(nullable = false, length = 3000)
    private String content;

    @CreatedDate
    @Column(updatable = false)
    protected LocalDateTime createdAt;

    public Post(String title, String content) {
        this.title = title;
        this.content = content;
    }
}
```

**DTO**
```java
public record PostRequest(String title, String content) {
}

public record PostResponse(Long id, String title, String content, LocalDateTime createdAt) {
}
```

**Controller**
```java
@RestController
@RequestMapping("/api/posts")
@RequiredArgsConstructor
public class PostController {
    private final PostService postService;

    @GetMapping
    public ResponseEntity<List<PostResponse>> getAll() {
        return ResponseEntity.ok(postService.getAll());
    }

    @PostMapping
    public ResponseEntity<Void> save(@RequestBody PostRequest postRequest) {
        postService.save(postRequest);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

**Service**
```java
@Service
@RequiredArgsConstructor
public class PostService {
    private final PostRepository postRepository;

    public List<PostResponse> getAll() {
        return postRepository.findAll().stream()
                .map(m -> new PostResponse(m.getId(), m.getTitle(), m.getContent(), m.getCreatedAt())
                ).toList();
    }

    public void save(PostRequest postRequest) {
        postRepository.save(new Post(postRequest.title(), postRequest.content()));
    }
}
```

**Repository**
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

## MySQL 테이블 생성
DB에 접속해서 애플리케이션에서 사용할 테이블을 생성한다.
```sql
CREATE TABLE post (
  id BIGINT NOT NULL AUTO_INCREMENT,
  title VARCHAR(100) NOT NULL,
  content VARCHAR(3000) NOT NULL,
  created_at TIMESTAMP(6) NOT NULL,
  PRIMARY KEY (id)
)
```

## 웹 서버 환경변수 설정
`start.sh` 스크립트에 다음 내용을 추가한다.
```bash
# 환경변수 설정
export DB_URL="jdbc:mysql://<DB 서버 private IP>:3306/mydb?serverTimezone=Asia/Seoul"
export DB_USERNAME="testuser"
export DB_PASSWORD="password1111"
```

## JAR 파일 전송
빌드된 JAR 파일을 서버의 배포 경로 `/home/ubuntu/app/current`로 옮긴다.

## 애플리케이션 실행 및 확인
`start.sh` 스크립트로 애플리케이션을 실행하고, 요청을 보내 정상적으로 작동하는지 확인한다.
```bash
# 애플리케이션 실행
/home/ubuntu/app/start.sh

# 로그 확인
tail -f /home/ubuntu/app/logs/app.out

# 애플리케이션 종료
/home/ubuntu/app/stop.sh
```
