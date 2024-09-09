# @EntityGraph

일반적으로 JPA에서는 지연 로딩(lazy loading) 방식을 사용하여 연관된 엔티티 객체를 필요할 때 가져온다.
그러나 이 경우에는 연관된 엔티티 객체를 가져오는 쿼리가 지연 로딩 수행 시 매번 실행되어 성능상의 이슈가 발생할 수 있다.

엔티티 그래프는 이러한 성능상의 이슈를 해결하기 위해,
한 번의 쿼리로 필요한 모든 엔티티 객체를 함께 가져오기 위한 방법을 제공한다.

## 예제 코드
게시글과 댓글은 1:N관계이다. 게시글을 조회할 때 게시글의 댓글도 함께 가져올 수 있도록 해보자.
```java
@Entity
@Getter
@ToString
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String content;

    @OneToMany(mappedBy = "post", fetch = FetchType.LAZY, cascade = CascadeType.PERSIST)
    private List<Comment> comments = new ArrayList<>();

    protected Post() {
    }

    public Post(String title, String content) {
        this.title = title;
        this.content = content;
    }

    public void addComment(Comment comment) {
        comment.setPost(this);
        comments.add(comment);
    }
}
```

```java
@Entity
@Getter
@ToString(exclude = "post")
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id")
    private Post post;

    protected Comment() {
    }

    public Comment(String content) {
        this.content = content;
    }

    public void setPost(Post post) {
        this.post = post;
    }
}
```
테스트 코드는 다음과 같다.
```java
@DataJpaTest
class PostRepositoryTest {
    @Autowired
    TestEntityManager entityManager;

    @Autowired
    PostRepository postRepository;

    @BeforeEach
    void setUp() {
        Post post1 = new Post("Post1", "..."); // comment 2개
        Post post2 = new Post("Post2", "..."); // comment 1개
        Post post3 = new Post("Post3", "..."); // comment 0개

        Comment comment1 = new Comment("Post1 Comment1");
        Comment comment2 = new Comment("Post1 Comment2");
        Comment comment3 = new Comment("Post2 Comment2");

        post1.addComment(comment1);
        post1.addComment(comment2);
        post2.addComment(comment3);

        postRepository.save(post1);
        postRepository.save(post2);
        postRepository.save(post3);

        entityManager.flush();
        entityManager.clear();
    }

    @Test
    void findAllTest() {
        // when
        List<Post> posts = postRepository.findAll();

        // then
        posts.forEach(p -> System.out.println(p.toString()));
        assertThat(posts).hasSize(3);
    }
}
```

## FetchType 전략을 사용하여 관련 엔티티 로드
FetchType을 EAGER로 변경하여 Post 엔티티를 로드할 때 Comment 엔티티도 함께 로드하도록 변경할 수 있다.
```java
@OneToMany(mappedBy = "post", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST)
private List<Comment> comments = new ArrayList<>();
```

실행되는 쿼리를 확인해 보면 Comment 엔티티를 가져오지만, 댓글의 수만큼 추가 조회 쿼리가 나간다.
```
select p1_0.id,p1_0.content,p1_0.title from post p1_0
select c1_0.post_id,c1_0.id,c1_0.content from comment c1_0 where c1_0.post_id=1
select c1_0.post_id,c1_0.id,c1_0.content from comment c1_0 where c1_0.post_id=2
select c1_0.post_id,c1_0.id,c1_0.content from comment c1_0 where c1_0.post_id=3
```

## @EntityGraph
@EntityGraph를 사용하여 N+1 문제를 해결해보자.

PostRepository의 findAll 메서드을 오버라이딩하여 @EntityGraph 어노테이션을 추가한다.
### @EntityGraph 속성
- `attributePaths` : fetch join 할 엔티티 필드명
- `type`
  - `EntityGraph.EntityGraphType.FETCH` : (기본값) entity graph attribute는 EAGER로 패치하고, 나머지는 LAZY로 패치한다.
  - `EntityGraph.EntityGraphType.LOAD` : entity graph attribute는 EAGER로 패치하고, 나머지는 엔티티에 명시한 FetchType 이나 default FetchType으로 패치한다.

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    @Override
    @EntityGraph(attributePaths = "comments")
    List<Post> findAll();
}
```
실행되는 쿼리를 확인해 보면 left join으로 댓글 정보를 함께 가져오고 있다.
```
select
    p1_0.id,
    c1_0.post_id,
    c1_0.id,
    c1_0.content,
    p1_0.content,
    p1_0.title 
from
    post p1_0 
left join
    comment c1_0 
        on p1_0.id=c1_0.post_id
```
@EntityGraph는 쿼리메서드, jpql과도 함께 사용할 수 있다.
