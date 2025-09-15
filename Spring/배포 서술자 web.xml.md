# 배포 서술자 web.xml
## web.xml
- `web.xml` 파일은 웹 애플리케이션의 배포 서술자(Deployment Descriptor)로서 XML 형식의 파일이다.
- `/WEB-INF/web.xml`에 위치한다.
- 웹 애플리케이션이 시작되면 WAS가 `web.xml` 파일을 읽어 서블릿, 필터, 리스너 등의 컴포넌트를 초기화한다.

## 주요 구성 요소
- `<servlet>`, `<servlet-mapping>`: 서블릿을 정의하고 URL 패턴을 매핑한다.
- `<filter>`, `<filter-mapping>`: 필터를 정의하고 URL 패턴 또는 서블릿에 매핑한다.
- `<listener>`: 애플리케이션 이벤트를 처리하기 위한 리스너를 정의한다.
- `<context-param>`: 애플리케이션 전역에서 사용할 수 있는 초기화 파라미터를 정의한다.
- `<init-param>`: 특정 서블릿이나 필터에 대한 초기화 파라미터를 정의한다.
- `<welcome-file-list>`: 디렉토리 요청 시 기본적으로 제공할 파일을 정의한다.
- `<error-page>`: 특정 오류 코드나 예외에 대한 사용자 정의 에러 페이지를 정의한다.
- `<security-constraint>`: 애플리케이션의 보안 제약을 정의한다.
- `<session-config>`: 세션 구성 요소를 정의한다.

## 예시
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    
    <!-- 웹 애플리케이션 이름 -->
    <display-name>MyWebApp</display-name>
    
    <!-- Servlet 정의 -->
    <servlet>
        <servlet-name>exampleServlet</servlet-name>
        <servlet-class>com.example.ExampleServlet</servlet-class>
        <!-- Servlet 초기화 파라미터 -->
        <init-param>
            <param-name>exampleParam</param-name>
            <param-value>paramValue</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <!-- Servlet 매핑 -->
    <servlet-mapping>
        <servlet-name>exampleServlet</servlet-name>
        <url-pattern>/example</url-pattern>
    </servlet-mapping>

    <!-- Filter 정의 -->
    <filter>
        <filter-name>exampleFilter</filter-name>
        <filter-class>com.example.ExampleFilter</filter-class>
    </filter>

    <!-- Filter 매핑 -->
    <filter-mapping>
        <filter-name>exampleFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- Listener 정의 -->
    <listener>
        <listener-class>com.example.ExampleListener</listener-class>
    </listener>

    <!-- Context Parameter 정의 -->
    <context-param>
        <param-name>configFile</param-name>
        <param-value>/WEB-INF/config.properties</param-value>
    </context-param>
    <context-param>
        <param-name>appName</param-name>
        <param-value>MyWebApplication</param-value>
    </context-param>

    <!-- Welcome File List 정의 -->
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

    <!-- Error Page 정의 -->
    <error-page>
        <error-code>404</error-code>
        <location>/error/404.html</location>
    </error-page>

    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/error/defaultError.html</location>
    </error-page>

    <!-- Security Constraint 정의 -->
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Protected Area</web-resource-name>
            <url-pattern>/secure/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>USER</role-name>
        </auth-constraint>
    </security-constraint>

    <!-- Session Configuration 정의 -->
    <session-config>
        <session-timeout>30</session-timeout> <!-- 세션 타임아웃을 30분으로 설정 -->
    </session-config>

</web-app>
```

---
**Reference**
- https://play-with.tistory.com/311
- https://tedock.tistory.com/100
- https://nobacking.tistory.com/19#2.%20주요%20태그%20정리-1-1
