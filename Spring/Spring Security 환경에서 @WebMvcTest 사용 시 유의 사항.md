# Spring Security 환경에서 @WebMvcTest 사용 시 유의 사항

## 오류 상황
`@WebMvcTest` 어노테이션으로 컨트롤러 단위 테스트 중 문제가 발생하였다.

**오류 상황**<br>
- GET 테스트에서 401 상태 코드를 반환
- GET 이외의 HTTP Method 테스트에서 403 상태 코드를 반환

오류가 발생한 테스트 코드는 다음과 같다.
```java
@WebMvcTest(BoardController.class)
class BoardControllerTest {

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BoardService boardService;

    @Test
    void testSaveBoard() throws Exception {
        CreateResponse response = new CreateResponse(1L);
        given(boardService.saveBoard(any(), any())).willReturn(response);

        BoardRequest boardRequest = new BoardRequest("제목", "내용");
        mockMvc.perform(post("/api/v1/boards")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(boardRequest)))
                .andDo(print())
                .andExpect(status().isCreated());
    }

    @Test
    void testGetBoard() throws Exception {
        BoardDetailResponse response = new BoardDetailResponse(1L, "제목", "내용", LocalDateTime.of(2024, 11, 10, 10, 0), LocalDateTime.of(2024, 11, 20, 10, 0), 1L, "작성자 이름");
        given(boardService.getBoard(any())).willReturn(response);

        mockMvc.perform(get("/api/v1/boards/{id}", 1)
                        .accept(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

## 발생 원인
결론부터 말하자면 Spring Security 설정으로 인한 문제였다.

[공식 문서](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html)에 따르면, `@WebMvcTest` 어노테이션에는 기본적으로 **Spring Security가 자동 설정** 된다고 나와있다.
> By default, tests annotated with @WebMvcTest will also auto-configure Spring Security and MockMvc (include support for HtmlUnit WebClient and Selenium WebDriver). 

<br>

Spring Security 자동 설정에 대해 찾아보자.

`org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration` 클래스의 일부이다.
```java
// ...
@Import({SpringBootWebSecurityConfiguration.class, SecurityDataConfiguration.class})
public class SecurityAutoConfiguration {
    // ...
}
```

@Import에 있는 `SpringBootWebSecurityConfiguration` 클래스를 확인해보면 defaultSecurityFilterChain 설정 로직을 확인할 수 있다.
```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
class SpringBootWebSecurityConfiguration {
    // ...

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnDefaultWebSecurity
    static class SecurityFilterChainConfiguration {
        SecurityFilterChainConfiguration() {
        }

        @Bean
        @Order(2147483642)
        SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
            http.authorizeHttpRequests((requests) -> {
                ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)requests.anyRequest()).authenticated();
            });
            http.formLogin(Customizer.withDefaults());
            http.httpBasic(Customizer.withDefaults());
            return (SecurityFilterChain)http.build();
        }
    }
}
```

<br>

GET 테스트에서 401 상태 코드를 반환<br>
-> defaultSecurityFilterChain에서는 모든 경로에 대해 인증을 요구하기 때문에 401이 반환된 것이었다.

GET 이외의 HTTP Method 테스트에서 403 상태 코드를 반환<br>
-> defaultSecurityFilterChain에서는 CSRF 보호가 활성화되기 때문에 403이 반환된 것이었다.

## 해결 방법
해결 방법을 찾아본 결과 다음과 같은 방식이 있었다.

### 1번: Spring Security 자동 구성 제외 처리
Spring Security 자동 설정을 진행하지 않도록 제외처리한다.
```java
@WebMvcTest(value = BoardController.class, excludeAutoConfiguration = SecurityAutoConfiguration.class)
class BoardControllerTest {
    // ...
}
```

### 2번: 테스트 환경용 SecurityFilterChain 정의
테스트 환경에서 사용할 SecurityFilterChain 빈을 정의한다.
```java
@TestConfiguration
public class MockSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(authorize -> authorize.anyRequest().permitAll())
                .build();
    }
}
```

테스트 클래스에서 해당 설정을 임포트한다.
```java
@Import(MockSecurityConfig.class)
@WebMvcTest(BoardController.class)
class BoardControllerTest {
    // ...
}
```

### 3번: 에러가 발생하지 않도록 테스트 코드 수정
CSRF 토큰 누락으로 인한 403 에러가 발생하지 않도록, GET 요청 이외의 테스트 코드에 `.with(csrf())`를 추가한다.

인증 정보 누락으로 인한 401 에러가 발생하지 않도록, 모든 테스트에 `@WithMockUser`로 인증 정보를 설정한다.

> 만약 커스텀 UserDetails를 구현해서 사용 중이라면 @WithMockUser 어노테이션를 사용할 수 없다.
> 해당 문제에 대한 해결 방법은 별도의 글로 작성할 예정이다.

```java
@WebMvcTest(BoardController.class)
class BoardControllerTest {

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BoardService boardService;

    @WithMockUser // 추가
    @Test
    void testSaveBoard() throws Exception {
        CreateResponse response = new CreateResponse(1L);
        given(boardService.saveBoard(any(), any())).willReturn(response);

        BoardRequest boardRequest = new BoardRequest("제목", "내용");
        mockMvc.perform(post("/api/v1/boards")
                        .contentType(MediaType.APPLICATION_JSON)
                        .with(csrf()) // 추가
                        .content(objectMapper.writeValueAsString(boardRequest)))
                .andDo(print())
                .andExpect(status().isCreated());
    }

    @WithMockUser // 추가
    @Test
    void testGetBoard() throws Exception {
        BoardDetailResponse response = new BoardDetailResponse(1L, "제목", "내용", LocalDateTime.of(2024, 11, 10, 10, 0), LocalDateTime.of(2024, 11, 20, 10, 0), 1L, "작성자 이름");
        given(boardService.getBoard(any())).willReturn(response);

        mockMvc.perform(get("/api/v1/boards/{id}", 1)
                        .accept(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

---
**Reference**<br>
- https://jiwondev.tistory.com/270
- https://stir.tistory.com/407
- https://hello-woody.medium.com/spring-mvc-%EC%97%90%EC%84%9C-controller-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C%EC%97%90%EC%84%9C%EB%8A%94-%EB%AD%98-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C-d398d5b4a35f
