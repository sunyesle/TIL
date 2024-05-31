# Spring Security static resource 설정

### Spring Security 5.7 이전
```java
@Bean
public WebSecurityCustomizer ignore() {
    return web -> web.ignoring().requestMatchers("/static/**");
}

@Bean
SecurityFilterChain app(HttpSecurity http) {
    http
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().authenticated()
            )
    // ...
    ;
    return http.build();
}
```

### Spring Security 5.7 이후

Spring Security 5.7 이후 버전에서 WebSecurity.ignoring()을 사용할 경우 경고메시지가 뜬다.

``You are asking Spring Security to ignore Ant [pattern='...']. This is not recommended -- please use permitAll via HttpSecurity#authorizeHttpRequests instead.``

[관련 Github Issue](https://github.com/spring-projects/spring-security/issues/10913)

[공식문서](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html#favor-permitall)

Spring Security 5.7부터 SecurityContext 조회 지연 기능을 지원한다.
과거에는 모든 요청에 대해 SecurityContext 조회 작업이 발생하여 불필요한 오버헤드가 있었고, 이를 해결하기 위해 SecurityFilterChain에서 제외시키는 ignoring()을 사용했다.
이제 Spring Security는 인증이 필요한 시점까지 SecurityContext 조회를 지연하며, permitAll() 설정을 한 경우 인증이 수행되지 않기 때문에 SecurityContext 조회 작업도 발생하지 않는다.

또한, ignoring()를 사용하면 Spring Security가 해당 엔드포인트에 보안 헤더나 기타 보호 조치를 제공할 수 없다.

따라서 ignoring()보다 permitAll()을 사용하는 것이 권장된다.

다음과 같이 정적 리소스를 위한 별도의 필터체인을 추가한다.
```java
@Bean
@Order(0)
public SecurityFilterChain ignore(HttpSecurity http) throws Exception {
    http
            .securityMatcher("/static/**")
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().permitAll()
            )
            .csrf(AbstractHttpConfigurer::disable)
            .requestCache(AbstractHttpConfigurer::disable)
            .securityContext(AbstractHttpConfigurer::disable)
            .sessionManagement(AbstractHttpConfigurer::disable)
    ;
    return http.build();
}

@Bean
@Order(1)
public SecurityFilterChain app(HttpSecurity http) throws Exception {
    http
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().authenticated()
            )
    // ...
    ;
    return http.build();
}
```

---
### Reference

https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-multi-securityfilterchain-figure

https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html#security-matchers

https://github.com/spring-projects/spring-security/issues/10938

https://github.com/spring-projects/spring-boot/issues/32622

https://docs.spring.io/spring-security/reference/5.8/migration/servlet/config.html#use-new-security-matchers

https://kdev.ing/threadlocal/

https://bombo96.tistory.com/100
