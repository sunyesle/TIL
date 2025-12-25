# Spring Boot 4 API Versioning
Spring Framework 7, Spring Boot 4부터 API 버전 관리 기능을 공식적으로 지원한다.
이를 통해 애플리케이션 전체에 일관된 버저닝 방식을 설정할 수 있다. 

## 버전 관리 전략
기본 제공 방식은 다음 네 가지이다. 일관성을 유지하기 위해 애플리케이션당 하나의 전략만 사용하는 것이 좋으며, Path Segment 전략은 다른 전략과 혼합해 사용할 수 없다.
| 방식 | 예시 | 특징 | 활용 대상 |
|------|------|------|-----------|
| **Path Segment** | `/api/v1/hello`<br>`/api/v2/hello` | RESTful 구조,<br>캐시 친화적 | 공개 API |
| **Request Header** | `X-API-Version: 1`<br>`X-API-Version: 2` | URL 구조 유지 | 내부 API, 마이크로서비스 간 통신 |
| **Request Parameter** | `/api/hello?version=1`<br>`/api/hello?version=2` | 디버깅 용이 | 개발 및 테스트 환경 |
| **Media Type Parameter** | `Accept: application/json; version=1`<br>`Accept: application/json; version=2` | 콘텐츠 협상 표준 준수 | 엔터프라이즈 API |

## 설정 방법
### 1. Properties-Based Config
> application.properties
```properties
spring.mvc.apiversion.supported=1.0,2.0
spring.mvc.apiversion.default=1.0

spring.mvc.apiversion.use.path-segment=1
spring.mvc.apiversion.use.header=X-API-Version
spring.mvc.apiversion.use.query-parameter=version
spring.mvc.apiversion.use.media-type-parameter[application/json]=version
```

> application.yml
```yml
spring:
  mvc:
    apiversion:
      supported: 1.0,2.0
      default: 1.0
      use:
        path-segment: 1
        header: X-API-Version
        query-parameter: version
        media-type-parameter:
          "[application/json]": version
```

### 2. `WebMvcConfigurer`를 이용한 Java Config
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureApiVersioning(ApiVersionConfigurer configurer) {
        configurer
                .addSupportedVersions("1.0", "2.0")
                .setDefaultVersion("1.0")
                .usePathSegment(1)
                .useRequestHeader("X-API-Version")
                .useQueryParam("version")
                .useMediaTypeParameter(MediaType.APPLICATION_JSON, "version");
    }
}
```

## 예시
### Path Segment 기반 API 버전 관리
프로퍼티 기반 설정과 자바 설정 중 하나를 선택해 사용하면 된다.
```properties
spring.mvc.apiversion.supported=1.0,2.0
spring.mvc.apiversion.use.path-segment=1
```
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureApiVersioning(ApiVersionConfigurer configurer) {
        configurer
                .addSupportedVersions("1.0", "2.0")
                .usePathSegment(1);
    }
}
```

`@RequestMapping`, `@GetMapping`, `@PostMapping` 등의 어노테이션에서 `version` 옵션으로 버전을 지정할 수 있다. 
```java
@RestController
public class UserController {

    @GetMapping(path = "/api/v{version}/users", version = "1.0")
    public List<UserDTOv1> getAllV1() {
    }

    @GetMapping(path = "/api/v{version}/users", version = "2.0")
    public List<UserDTOv2> getAllV2() {
    }
}
```

**요청 URL 예시**<br>
URL 경로의 버전 값(`v{version}`)에 따라 서로 다른 컨트롤러 메서드가 매핑된다.
- v1 API 호출: `GET /api/v1.0/users`
- v2 API 호출: `GET /api/v2.0/users`

Path Segment 방식에서는 API 버전이 URL 경로에 포함되므로, 반드시 `/api/{version}`과 같이 URI 변수로 선언해야 한다. 변수명은 자유롭게 지정 가능하다.

공통 Path Prefix를 설정으로 분리해 관리하는 방식도 고려할 수 있다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    ...

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer.addPathPrefix("/api/v{version}", HandlerTypePredicate.forAnnotation(RestController.class));
    }
}
```
위와 같이 설정하면 Controller에서 접두사를 반복적으로 선언하지 않아도 된다.
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping(version = "1.0")
    public List<UserDTOv1> getAllV1() {
    }

    @GetMapping(version = "2.0")
    public List<UserDTOv2> getAllV2() {
    }
}
```

---
**Reference**<br>
- https://spring.io/blog/2025/09/16/api-versioning-in-spring
- https://www.baeldung.com/spring-boot-4-spring-framework-7
- https://www.danvega.dev/blog/spring-boot-4-api-versioning?ref=apisyouwonthate.com
