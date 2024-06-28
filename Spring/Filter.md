# Filter
필터는 리소스에 대한 요청과 응답에 대해 필터링 작업을 수행하는 객체이다.
서블릿에서 제공하는 기능으로 Servlet 컨테이너(WAS)에서 관리한다.

```java
public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {
    }

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    default void destroy() {
    }
}
```
- `init` : 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출한다.
- `doFilter` : 요청이 올 떄마다 해당 메서드가 호출한다.
- `destroy` : 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출한다.

## 필터 등록 방법

### 1. @Component
`Filter` 인터페이스를 구현하고 빈으로 등록한다. 정의한 필터가 여러 개 존재하는 경우, `@Order` 어노테이션을 통해 순서를 지정할 수 있다.
```java
@Component
@Order(1)
public class MyFilter implements Filter {
   // ...
}
```

### 2. FilterRegistrationBean
필터를 특정 URL 패턴에만 적용하는 등의 상세한 설정이 필요한 경우 `FilterRegistrationBean`을 사용하여 필터를 등록한다.
```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean<MyFilter> reqResLoggingFilterBean() {
        FilterRegistrationBean<MyFilter> filterReg = new FilterRegistrationBean<>();
        filterReg.setFilter(new MyFilter());
        filterReg.addUrlPatterns("/api/*");
        filterReg.setOrder(1);
        return filterReg;
    }
}
```

### 3. @ServletComponentScan + @WebFilter
`Filter` 인터페이스를 구현한 클래스에 `@WebFilter`를 추가한다.
`@WebFilter`의 value 나 urlPatterns 파라미터를 이용해 URL 패턴을 지정할 수 있다.
```java
//@WebFilter("/api/*")
@WebFilter(urlPatterns = "/api/*")
public class MyFilter implements Filter {
   // ...
}
```
<br>

**@ServletComponentScan**<br>
서블릿 컴포넌트(`@WebFilter`, `@WebServlet`, `@WebListener`)를 스캔해서 빈으로 등록한다. 패키지를 지정하지 않을 경우 어노테이션이 달린 클래스의 패키지에서 스캔한다.
<br>

`@Configuration` 클래스에 `@ServletComponentScan`을 추가한다.
> Main Application 클래스에 추가
```java
@SpringBootApplication // @Configuration이 포함되어 있다.
@ServletComponentScan
public class ApiApplication {

	public static void main(String[] args) {
		SpringApplication.run(ApiApplication.class, args);
	}
}
```
> 별도의 Configuration 클래스에 추가
```java
@Configuration
@ServletComponentScan(basePackages = "com.example")
public class ServletConfig {

}
```

---
**#Reference**
- https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.add-servlet-filter-listener
- https://jronin.tistory.com/124
- https://velog.io/@roycewon/Filter%EC%99%80-Interceptor
