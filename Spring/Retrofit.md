# Spring Boot에서 Retrofit 사용하기

## Retrofit이란

> https://square.github.io/retrofit/

Retrofit은 HTTP 통신을 도와주는 라이브러리로 Interface에 선언된 HTTP API 명세를 기반으로 구현체를 생성해 준다.

## 사용법

### 의존성 추가
```gradle
implementation 'com.squareup.retrofit2:retrofit:2.11.0'
implementation 'com.squareup.retrofit2:converter-jackson:2.10.0' // converter-jackson
implementation 'com.squareup.okhttp3:logging-interceptor:4.12.0' // 로깅 인터셉터
```
예시에서는 jackson 컨버터를 사용했다. 그 외에도 gson, simpleXML 등 다양한 컨버터를 지원한다. 

### Interface에 Http API 기술
```java
public interface UserAPI {
    @GET("users")
    Call<UserListResponse> getUserList(@Query("page") Integer page);

    @GET("users/{userId}")
    Call<UserResponse> getUser(@Path("userId") Integer userId);

    @POST("users")
    Call<UserCreateResponse> saveUser(@Body UserCreateRequest request);
}
```

### 설정 및 구현체 생성
```java
@Configuration
public class RetrofitConfig {
    private static final String BASE_URL = "https://reqres.in/";

    @Bean
    public HttpLoggingInterceptor httpLoggingInterceptor() {
        HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
        httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        return httpLoggingInterceptor;
    }

    @Bean
    public OkHttpClient okHttpClient(Interceptor httpLoggingInterceptor) {
        return new OkHttpClient.Builder()
                .addInterceptor(httpLoggingInterceptor) // 로깅 인터셉터
                .connectTimeout(20, TimeUnit.SECONDS)
                .writeTimeout(60, TimeUnit.SECONDS)
                .readTimeout(60, TimeUnit.SECONDS)
                .build();
    }

    @Bean
    public Retrofit retrofit(OkHttpClient client) {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new JavaTimeModule());
        objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);

        String baseURL = BASE_URL + "api/";
        return new Retrofit.Builder().baseUrl(baseURL)
                .addConverterFactory(JacksonConverterFactory.create(objectMapper)) // 컨버터
                .client(client)
                .build();
    }

    @Bean
    public UserAPI userAPI(Retrofit retrofit) {
        return retrofit.create(UserAPI.class); // Http API 명세가 담긴 Interface의 구현체를 반환한다.
    }
}
```

---
**Reference**<br>
- https://galid1.tistory.com/617
- https://medium.com/myrealtrip-product/retrofit2-%EC%99%80-resilience4j-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B9%A0%EB%A5%B8-rest-api-client-%EA%B5%AC%ED%98%84-81c10dcfb6
- https://github.com/HwangEunmi/Retrofit-Sample?tab=readme-ov-file#retrofit%EC%9D%98-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98
