# REST Assured

## Gradle 설정
```gradle
dependencies {
	testImplementation 'io.rest-assured:rest-assured:5.5.0'
}
```

## Default Configuration
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public abstract class BaseAcceptanceTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    public void setUp() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = port;
    }
}
```

## GET 요청
```java
@DisplayName("게시글 목록을 조회한다")
@Test
void getBoards() {
    // when
    ExtractableResponse<Response> response = RestAssured
            .given()
            .when()
                .get("/api/v1/boards")
            .then()
                .extract();

    // then
    assertThat(response.statusCode()).isEqualTo(HttpStatus.OK.value());
}
```

## POST 요청
```java
@DisplayName("게시글을 작성한다")
@Test
void saveBoard() throws JsonProcessingException {
    // given
    String title = "테스트 게시글";
    String content = "테스트 내용";
    BoardRequest boardRequest = new BoardRequest(title, content);

    // when
    ExtractableResponse<Response> response = RestAssured
            .given()
                .contentType(ContentType.JSON)
                .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
                .body(objectMapper.writeValueAsString(boardRequest))
            .when()
                .post("/api/v1/boards")
            .then()
                .extract();

    // then
    assertThat(response.statusCode()).isEqualTo(HttpStatus.CREATED.value());
}
```

# 사용법
> 자세한 사용법은 [공식문서](https://github.com/rest-assured/rest-assured/wiki/Usage)에서 확인할 수 있다.

## 요청
### Parameters
```java
given()
    .param("param", "value1")
    .param("myList", "value1", "value2") // 다중값 매개변수
.when()
    .get("/something");
```
REST Assured는 HTTP 메서드에 따라 parameter 유형을 자동으로 판별한다. GET의 경우 query parameter, POST의 경우 form parameter가 사용된다.

다음과 같이 유형을 직접 지정할 수도 있다.
```java
given()
    .formParam("formParamName", "value1")
    .queryParam("queryParamName", "value2")
.when()
    .post("/something");
```

### Path Param
```java
post("/reserve/{hotelId}/{roomNumber}", "My Hotel", 23);
```
또는
```java
given()
    .pathParam("hotelId", "My Hotel")
    .pathParam("roomNumber", 23)
.when()
    .post("/reserve/{hotelId}/{roomNumber}")
```

### Header
```java
given()
    .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
    .header("headerName", "value1", "value2") // 다중값 헤더
```

### ContentType
```java
given().contentType(ContentType.JSON)
given().noContentType()
```

### Body
```java
Map<String, String> params = new HashMap<>();
    params.put("id", "1");
    params.put("name", "value1");

given().body(params)
```

---
**Reference**<br>
- https://github.com/rest-assured/rest-assured/wiki/Usage


```java
RequestDTO request = new RequestDto(1, "value1");

given().body(request)
```

