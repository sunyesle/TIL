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

---
**Reference**<br>
- https://incheol-jung.gitbook.io/docs/q-and-a/spring/resttemplate
- https://minkwon4.tistory.com/216
- https://moonsiri.tistory.com/202
- https://hc.apache.org/httpcomponents-client-5.4.x/migration-guide/preparation.html
