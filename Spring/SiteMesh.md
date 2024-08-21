# SiteMesh

Spring MVC에서 Spring Boot로 마이그레이션 작업 중 Spring Boot에 SiteMesh를 적용한 부분을 정리해 보려고 한다.

SiteMesh 3도 나왔지만, 기존에 사용하던 설정 파일을 그대로 사용하기 위해 2.4.1 버전을 사용했다.

## SiteMesh 필터 추가
web.xml에서 정의하고 있던 SiteMesh 필터를 자바 설정으로 변경했다.
2.4 버전부터 PageFilter가 deprecated 되었다고 하여 SiteMeshFilter으로 변경하였다.

**변경 전**
```xml
<filter>
  <filter-name>sitemesh</filter-name>
  <filter-class>
    com.opensymphony.module.sitemesh.filter.PageFilter
  </filter-class>
</filter>

<filter-mapping>
  <filter-name>sitemesh</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

**변경 후**
```java
@Configuration
public class SitemeshConfig {

    @Bean
    public FilterRegistrationBean<SiteMeshFilter> siteMeshFilter() {
        FilterRegistrationBean<SiteMeshFilter> filter = new FilterRegistrationBean<>();
        filter.setFilter(new SiteMeshFilter());
        return filter;
    }
}
```

## SiteMesh 설정 파일
sitemesh.xml 파일은 src/main/webapp/WEB-INF 하위에 위치해야 한다.

---
**Reference**<br>
- https://github.com/sitemesh/sitemesh2
