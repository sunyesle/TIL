# ObjectMapper
`ObjectMapper`는 Jackson 라이브러리의 핵심 클래스 중 하나로, 자바 객체를 JSON으로 변환(직렬화)하거나 JSON을 자바 객체로 변환(역직렬화)하는 기능을 제공한다.

## ObjectMapper를 이용한 직렬화(Serialize)
![Serialize](https://github.com/user-attachments/assets/d1c05a8d-814b-4053-a63e-d2942b1d1dcf)

- 자바 객체 -> JSON
- 기본적으로 public 필드 또는 public 형태의 getter를 사용한다.

```java
@Test
void testSerialize() throws JsonProcessingException {
    String json = """
            {"name": "apple", "price": "2000"}
            """;
    Product product = objectMapper.readValue(json, Product.class);

    System.out.println(product); // Product(name=apple, price=2000)
}
```

> visibility 옵션을 통해 어떤 접근 제한자를 이용할지 변경할 수 있다.
> ```java
> ObjectMapper objectMapper = new ObjectMapper();
> objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
> ```

## ObjectMapper를 이용한 역직렬화(Deserialize)
![Deserialize](https://github.com/user-attachments/assets/619784b3-af58-4097-8067-b7f55562eb4b)

- JSON -> 자바 객체
- 기본적으로 기본 생성자와 public 필드 또는 public 형태의 getter, setter를 사용한다.

기본 생성자로 객체를 생성하고, 필드 값을 찾아서 값을 바인딩한다.

```java
@Test
void testDeserialize() throws JsonProcessingException {
    Product product = new Product("apple", 2000);
    String json = objectMapper.writeValueAsString(product);

    System.out.println(json); // {"name":"apple","price":2000}
}
```

<br>

# 기본 생성자 없이도 @RequestBody 바인딩이 된다?
Spring Boot는 기본적으로 Jackson의 `ObjectMapper`를 사용하여 직렬화/역직렬화를 처리한다.

`@RequestBody` 어노테이션을 사용할 때 HTTP 요청 본문을 자바 객체로 변환하기 위해 `ObjectMapper`가 사용되는데,<br>
객체에 기본 생성자가 없어도 에러가 발생하지 않고 정상적으로 바인딩 되는 것을 확인할 수 있다.

```java
@Getter
public class RequestData {
    private final int id;
    private final String name;

    public RequestData(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

```java
@RestController
@RequestMapping("/test")
@RequiredArgsConstructor
public class TestController {
    
    @PostMapping
    public ResponseEntity<Void> test(@RequestBody RequestData request){
        return ResponseEntity.ok();
    }
}
```

<br>

결론부터 말하자면, **Spring Boot가 `ObjectMapper` 자동 구성 과정에서 `ParameterNamesModule` 모듈을 추가해주기 때문이다.**

## ParameterNamesModule
파라미터가 있는 생성자를 이용해 객체 바인딩을 할 수 있게 해주는 모듈이다. [github](https://github.com/FasterXML/jackson-modules-java8/tree/master/parameter-names)

```gradle
implementation 'com.fasterxml.jackson.module:jackson-module-parameter-names'
```

해당 모듈을 `ObjectMapper`에 등록하고, Java 컴파일 옵션 `-parameters` 추가하면 사용할 수 있다.
```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.registerModule(new ParameterNamesModule());
```

## JacksonAutoConfiguration
Spring Boot가 Jackson `ObjectMapper`를 자동 구성하는 과정에서 `ParameterNamesModule` 모듈의 의존성이 있을 경우 자동으로 설정을 추가해 준다.
```java
@Configuration(
        proxyBeanMethods = false
)
@ConditionalOnClass({ParameterNamesModule.class})
static class ParameterNamesModuleConfiguration {
    ParameterNamesModuleConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean
    ParameterNamesModule parameterNamesModule() {
        return new ParameterNamesModule(Mode.DEFAULT);
    }
}
```
`org.springframework.boot:spring-boot-starter-web` 의존성에
`com.fasterxml.jackson.module:jackson-module-parameter-names`이 포함되어 있기 때문에 별도의 설정 없이 적용된 것이었다.

디버깅해 보면 `ObjectMapper`에 `jackson-module-parameter-names` 모듈이 등록되어있는 것을 확인할 수 있다.

![ObjectMapper](https://github.com/user-attachments/assets/716e7332-27e5-40d4-8aa5-1edab90d02f4)

`-parameters` 옵션은 Spring Boot Gradle 플러그인에서 처리해 준다.<br>
프로젝트에 `java` 플러그인이 적용되어 있다면 Spring Boot 플러그인은 `JavaCompile` 태스크가 `-parameters` 컴파일 옵션을 사용하도록 설정한다.
```java
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.2'
}
```
> Spring Boot Gradle 플러그인에 대한 자세한 내용은 [공식 문서](https://docs.spring.io/spring-boot/gradle-plugin/reacting.html#reacting-to-other-plugins.java)에서 확인할 수 있다.

---
**Reference**<br>
- https://mangkyu.tistory.com/223
- https://beaniejoy.tistory.com/76
