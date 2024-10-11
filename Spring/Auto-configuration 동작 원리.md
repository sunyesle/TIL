# Spring Boot Auto-configuration 동작 원리

> 이 글은 Spring Boot 3.3.4 기준으로 작성되었다.

**자동 구성**(Auto-configuration)은 스프링 부트의 핵심 기능 중 하나이다.

AutoConfiguration은 spring-boot-project의 하위 모듈인 [spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/tree/v3.3.4/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure)에서 구현했다.

스프링 부트는 web, jdbc, jpa 등 수많은 자동 구성을 제공한다.
사용할 라이브러리의 의존성만 추가하면 필요한 빈들을 자동으로 등록되어, 애플리케이션을 더 쉽게 설정하고 관리할 수 있게 돕는다.

## @EnableAutoConfiguration
@Configuration 클래스에 @EnableAutoConfiguration 어노테이션을 추가하여 자동 구성 기능을 사용할 수 있다.
@EnableAutoConfiguration 어노테이션이 어떻게 동작하는지 확인해 보자.

@EnableAutoConfiguration 어노테이션은 다음과 같이 @SpringBootApplication 내부에 있다.
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // ...
}
```

이어서 @EnableAutoConfiguration 내부를 살펴보면 @Import를 통해 AutoConfigurationImportSelector 클래스를 import 하고 있다.
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    // ...
}
```

## AutoConfigurationImportSelector
AutoConfigurationImportSelector는 ImportSelector 인터페이스를 구현하고 있다.
ImportSelector의 selectImport() 메서드는 String 배열을 리턴하는데, 이 배열은 설정으로 사용할 클래스 이름을 값으로 갖는다.

`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일에 정의되어 있는 AutoConfiguration 클래스들을 읽어온다.
그 뒤, exclude 할 class와 filter 조건을 보면서 실제로 주입할 설정 정보를 결정한다.

---
**Reference**
- https://docs.spring.io/spring-boot/reference/features/developing-auto-configuration.html
- https://sup2is.github.io/2020/11/16/how-spring-auto-configuration-works.html
- https://wildeveloperetrain.tistory.com/292
- https://jh-labs.tistory.com/337
