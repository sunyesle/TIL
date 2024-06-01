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
## Spring Security 5.7 변경 사항

[공식문서](https://docs.spring.io/spring-security/reference/5.7/servlet/authentication/persistence.html)

Spring Security 5.7부터 SecurityContextPersistenceFilter가 SecurityContextHolderFilter로 대체되어 사용된다. 두 필터의 차이점을 알아보자.

### SecurityContextPersistenceFilter
![securitycontextpersistencefilter](https://github.com/sunyesle/TIL/assets/45172865/706d440d-0eb2-411a-b42e-e709d9fa77ab)

> [SecurityContextPersistenceFilter.java](https://github.com/spring-projects/spring-security/blob/main/web/src/main/java/org/springframework/security/web/context/SecurityContextPersistenceFilter.java)

```java
private SecurityContextRepository repo;

private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    // ensure that filter is only applied once per request
    if (request.getAttribute(FILTER_APPLIED) != null) {
        chain.doFilter(request, response);
        return;
    }
    request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
    if (this.forceEagerSessionCreation) {
        HttpSession session = request.getSession();
        if (this.logger.isDebugEnabled() && session.isNew()) {
            this.logger.debug(LogMessage.format("Created session %s eagerly", session.getId()));
        }
    }
    HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request, response);
    SecurityContext contextBeforeChainExecution = this.repo.loadContext(holder); // ※조회
    try {
        this.securityContextHolderStrategy.setContext(contextBeforeChainExecution);
        if (contextBeforeChainExecution.getAuthentication() == null) {
            logger.debug("Set SecurityContextHolder to empty SecurityContext");
        }
        else {
            if (this.logger.isDebugEnabled()) {
                this.logger
                    .debug(LogMessage.format("Set SecurityContextHolder to %s", contextBeforeChainExecution));
            }
        }
        chain.doFilter(holder.getRequest(), holder.getResponse());
    }
    finally {
        SecurityContext contextAfterChainExecution = this.securityContextHolderStrategy.getContext();
        // Crucial removal of SecurityContextHolder contents before anything else.
        this.securityContextHolderStrategy.clearContext();
        this.repo.saveContext(contextAfterChainExecution, holder.getRequest(), holder.getResponse()); // ※저장
        request.removeAttribute(FILTER_APPLIED);
        this.logger.debug("Cleared SecurityContextHolder to complete request");
    }
}
```
항상 SecurityContext를 조회하고 저장해주고 있다.

### SecurityContextHolderFilter
![securitycontextholderfilter](https://github.com/sunyesle/TIL/assets/45172865/3376b53e-dc52-447d-a6b6-8c88a0f20afa)

> [SecurityContextHolderFilter.java](https://github.com/spring-projects/spring-security/blob/main/web/src/main/java/org/springframework/security/web/context/SecurityContextHolderFilter.java)
```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
        throws ServletException, IOException {
    if (request.getAttribute(FILTER_APPLIED) != null) {
        chain.doFilter(request, response);
        return;
    }
    request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
    Supplier<SecurityContext> deferredContext = this.securityContextRepository.loadDeferredContext(request); // ※조회
    try {
        this.securityContextHolderStrategy.setDeferredContext(deferredContext);
        chain.doFilter(request, response);
    }
    finally {
        this.securityContextHolderStrategy.clearContext();
        request.removeAttribute(FILTER_APPLIED);
    }
}
```
SecurityContextPersistenceFilter와 다르게 loadContext()가 아닌 loadDeferredContext() 메서드를 호출하고 있다.

SecurityContextRepository의 loadDeferredContext() 메서드를 확인해 보자.
```java
default DeferredSecurityContext loadDeferredContext(HttpServletRequest request) {
    Supplier<SecurityContext> supplier = () -> {
        return this.loadContext(new HttpRequestResponseHolder(request, (HttpServletResponse)null));
    };
    return new SupplierDeferredSecurityContext(SingletonSupplier.of(supplier), SecurityContextHolder.getContextHolderStrategy());
}
```
Supplier 인터페이스를 활용하여 SecurityContext 조회를 지연 처리하는 방식으로 변경되었다.

또한, SecurityContext를 자동으로 저장해주지 않는다. 필요하다면 명시적으로 SecurityContext를 저장해야 한다.

### 요약
- SecurityContext 조회를 지연 처리하는 방식으로 변경하였다.
- 사용자가 SecurityContext를 저장할 것인지 아닌지를 선택할 수 있게 되었다.

---
### Reference

https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-multi-securityfilterchain-figure

https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html#security-matchers

https://github.com/spring-projects/spring-security/issues/10938

https://github.com/spring-projects/spring-boot/issues/32622

https://docs.spring.io/spring-security/reference/5.8/migration/servlet/config.html#use-new-security-matchers

https://kdev.ing/threadlocal/

https://bombo96.tistory.com/100

https://velog.io/@goniieee/Spring-Security%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9D%B8%EC%A6%9D%EB%90%9C-%EC%82%AC%EC%9A%A9%EC%9E%90%EB%A5%BC-%EA%B8%B0%EC%96%B5%ED%95%A0%EA%B9%8C
