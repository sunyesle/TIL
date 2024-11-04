# RestTemplate

RestTemplate은 Spring에서 제공하는 HTTP client이다.

## 설정
```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        HttpClientConnectionManager connectionManager = PoolingHttpClientConnectionManagerBuilder.create()
                .setDefaultConnectionConfig(ConnectionConfig.custom()
                        .setSocketTimeout(Timeout.ofSeconds(5)) // 읽기 최대 시간
                        .setConnectTimeout(Timeout.ofSeconds(3)) // 커넥션 최대 시간
                        .setTimeToLive(Timeout.ofSeconds(5)).build()) // 커넥션 만료 시간
                .setMaxConnTotal(100) // 최대로 연결할 수 있는 커넥션 스레드 수
                .setMaxConnPerRoute(10) // (IP + PORT) 당 커넥션 스레드 수
                .build();

        HttpClient httpClient = HttpClientBuilder.create()
                .setConnectionManager(connectionManager)
                .evictIdleConnections(TimeValue.of(60, TimeUnit.SECONDS)) // 최대 연결 유효시간
                .evictExpiredConnections() // 설정한 최대 연결 유효시간이 만료되면 커넥션을 해제한다
                .build();

        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setHttpClient(httpClient);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new JavaTimeModule());
        objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);

        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        converter.setObjectMapper(objectMapper);

        RestTemplate restTemplate = new RestTemplate(factory);
        restTemplate.getMessageConverters().add(0, converter); // 커스텀 메시지 컨버터 추가
        return restTemplate;
    }
}
```

## RestTemplate 메서드

| 메서드            | HTTP    | 설명                                                               |
|-------------------|---------|--------------------------------------------------------------------|
| getForObject      | GET     | 주어진 URL 주소로 GET 메서드로 객체로 결과를 반환                    |
| getForEntity      | GET     | 주어진 URL 주소로 GET 메서드로 객체로 결과를 ResponseEntity로 반환     |
| postForLocation   | POST    | POST 요청을 보내고 결과로 헤더에 저장된 URI를 결과로 반환              |
| postForObject     | POST    | POST 요청을 보내고 객체로 결과를 반환                               |
| postForEntity     | POST    | POST 요청을 보내고 결과를 ResponseEntity로 반환                     |
| delete            | DELETE  | 주어진 URL 주소로 DELETE 메서드 실행                               |
| headForHeaders    | HEAD    | 헤더의 모든 정보를 얻을 수 있으면 HTTP HEAD 메서드 사용              |
| put               | PUT     | 주어진 URL 주소로 PUT 메서드 실행                                  |
| patchForObject    | PATCH   | 주어진 URL 주소로 PATCH 메서드 실행                                |
| optionsForAllow   | OPTIONS | 주어진 URL 주소에서 지원하는 HTTP 메서드 조회                        |
| exchange          | ANY     | HTTP 헤더를 새로 만들 수 있고 어떤 HTTP 메서드도 사용 가능           |
| execute           | ANY     | Request/Response 콜백 수정 가능                                    |

## 사용 예시
```java
@Component
@RequiredArgsConstructor
public class UserCaller {
    private final RestTemplate restTemplate;

    public UserResponse getUser(String userId) {
        String url = "https://reqres.in/api/users/" + userId;
        ResponseEntity<UserResponse> response = restTemplate.getForEntity(url, UserResponse.class);
        return response.getBody();
    }

    public UserResponse createUser(UserCreateResponse request) {
        String url = "https://reqres.in/api/users";
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(url, request, UserResponse.class);
        return response.getBody();
    }

    public UserResponse updateUser(UserUpdateRequest request) {
        String url = "https://reqres.in/api/users";
        HttpEntity<UserUpdateRequest> requestHttpEntity = new HttpEntity<>(request);
        ResponseEntity<UserResponse> response = restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, UserResponse.class);
        return response.getBody();
    }
}

```



---
**Reference**<br>
- https://incheol-jung.gitbook.io/docs/q-and-a/spring/resttemplate
- https://minkwon4.tistory.com/216
- https://moonsiri.tistory.com/202
- https://hc.apache.org/httpcomponents-client-5.4.x/migration-guide/preparation.html
- https://recordsoflife.tistory.com/360
