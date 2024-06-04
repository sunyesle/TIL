# Spring Transaction

### JDBC 트랜잭션 관리
```java
@Service
public class UserService { 
    private String url = "jdbc:mysql://localhost:3306/mydatabase";
    private String dbUser = "myusername";
    private String dbPass = "mypassword";

    public Long registerUser(User user) {
        Connection conn = null;
        PreparedStatement pstmt = null;

        try {
            conn = DriverManager.getConnection(url, dbUser, dbPass);  // Connection 획득
            conn.setAutoCommit(false); // Transaction 시작
            // SQL 실행 ...
            conn.commit(); // 성공 시 커밋
            return id;

        } catch (SQLException e) {
            e.printStackTrace();
            if(conn != null) {
                try {
                    conn.rollback(); // 예외 발생 시 롤백
                } catch (SQLException ex) { ex.printStackTrace(); }
            }
        } finally {
            // 자원 해제
            if(pstmt != null) {
                try { pstmt.close(); } catch (SQLException e) { e.printStackTrace(); }
            }
            if (conn != null) {
                DataSourceUtils.releaseConnection(conn, dataSource);
            }
        }
    }
}
```
트랜잭션마다 커넥션 획득, 커밋, 롤백, 자원 해제 등의 트랜잭션 관리 로직이 반복적으로 사용된다.
비즈니스 로직의 가독성을 떨어뜨린다.

### 프로그래밍 방식 트랜잭션 관리

```java
@Service
public class UserService {

   @Autowired
   private TransactionTemplate template;

   public Long registerUser(User user) {
      Long id = template.execute(status -> {
         // SQL 실행 ...
         return id;
      });
   }
}
```
템플릿 콜백 패턴을 사용하여 트랜잭션 관련 로직이 반복되던 문제를 해결하였다.
하지만 여전히 비즈니스 로직에 트랜잭션 관리 로직이 침투되어 있다.

### 선언적 트랜잭션 관리

```java
@Service
public class UserService {

    @Transactional
    public Long registerUser(User user) {
        // SQL 실행...
        //userDao.save(user);
        return id;
    }
}
```
Spring은 트랜잭션 관리를 편리하게 수행할 수 있도록 **선언적 트랜잭션 관리**를 지원한다.

메서드나 클래스 레벨에 ``@Transactional`` 어노테이션을 추가하여 트랜잭션 전파, 고립 수준, 타임아웃, 읽기 전용 플래그 등 다양한 트랜잭션 속성을 설정할 수 있다.

``@Transactional``은 Spring AOP를 기반으로 만들어진 어노테이션이다.
Spring AOP는 기본적으로 프록시 방식으로 동작하며, CGLIB 또는 JDK Dynamic Proxy로 프록시 객체를 생성한다.
``@Transactional`` 어노테이션이 적용된 경우, Spring은 트랜잭션 프록시 객체를 생성하여 대상 객체를 감싼다.

![transactional](https://github.com/sunyesle/TIL/assets/45172865/c49de2d3-a504-4bc3-b787-d5e226fa28e7)

**[동작방식]**
- Spring 컨테이너는 빈을 생성할 때 @Transactional 어노테이션이 적용된 경우, 이 빈의 프록시 객체를 생성한다.
- 프록시 객체가 메서드 호출을 가로채서 트랜잭션을 시작한다.
- 트랜잭션 내에서 실제 객체 메서드를 호출하여 비즈니스 로직을 실행한다.
- 비즈니스 로직 실행 후, 트랜잭션을 커밋하거나 롤백한다.

---
### Reference

https://docs.spring.io/spring-framework/reference/data-access/transaction.html

https://www.marcobehler.com/guides/spring-transaction-management-transactional-in-depth

https://jiwondev.tistory.com/154

https://pjh3749.tistory.com/87

https://mingeun2154.github.io/spring/spring-transaction-management/

https://coding-business.tistory.com/82

https://hwannny.tistory.com/98
