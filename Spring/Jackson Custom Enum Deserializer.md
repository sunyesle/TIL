# Jackson Custom Enum Deserializer

`@RequestBody`로 JSON을 DTO에 매핑할 때, Enum 필드에 정의되지 않은 문자열이 들어올 경우 JSON parse error가 발생한다.

```log
Resolved [org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `com.sunyesle.atddmembership.enums.MembershipType` from String "INVALID": not one of the values accepted for Enum class: [NAVER, KAKAO, LINE]]
```

유효하지 않을 값이 들어왔을 때 오류 없이 null 값이 할당되도록 커스텀 Enum 역직렬화를 구현해보려고 한다.

## @JsonCreator
`@JsonCreator` 어노테이션으로 역직렬화에 사용되는 생성자나 팩토리 메서드를 지정할 수 있다.

```java
public enum MembershipType {
    NAVER, KAKAO, LINE;

    @JsonCreator
    public static MembershipType from(String value) {
        return Arrays.stream(values())
                .filter(type -> type.name().equals(value))
                .findAny()
                .orElse(null);
    }
}
```

> 특정 클래스만 역직렬화 방식을 변경해야 하는 경우에 사용하면 좋을 것 같다.

## @JsonDeserialize을 통해 Deserializer 등록
Deserializer를 구현한다. [(공식 문서 참고)](https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/latest/com/fasterxml/jackson/databind/JsonDeserializer.html)

### StdDeserializer
`StdDeserializer`는 사용자 지정 Deserializer을 구현을 위한 기본 클래스이다.

### ContextualDeserializer
`ContextualDeserializer`는 createContextual 메서드를 이용해 상황에 맞는 적절한 Deserialzer를 지정해 주는 역할을 한다.
`createContextual` 메서드는 지정된 속성 값의 Deserializer 인스턴스를 반환한다.

특정 클래스가 아니라 `Enum<?>`과같이 여러 클래스에 적용하고자 할 경우, 현재 Deserializer를 적용할 클래스 Type을 명확하게 `this._valueClass`에 주입할 수 있도록 반드시 `ContextualDeserializer`를 상속하고 `createContextual` 메서드를 구현해야 한다.

```java
public class CustomEnumDeserializer extends StdDeserializer<Enum<?>> implements ContextualDeserializer {

    public CustomEnumDeserializer() {
        this(null);
    }

    protected CustomEnumDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Enum<?> deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
        JsonNode jsonNode = jsonParser.getCodec().readTree(jsonParser);
        String text = jsonNode.asText();
        Class<? extends Enum<?>> enumType = (Class<? extends Enum<?>>) this._valueClass;

        return Arrays.stream(enumType.getEnumConstants())
                .filter(e -> e.name().equals(text))
                .findAny()
                .orElse(null);
    }

    @Override
    public JsonDeserializer<?> createContextual(DeserializationContext deserializationContext, BeanProperty beanProperty) throws JsonMappingException {
        return new CustomEnumDeserializer(beanProperty.getType().getRawClass());
    }
}
```

`@JsonDeserialize` 어노테이션을 이용해 Deserializer를 지정한다.
```java
@JsonDeserialize(using = CustomEnumDeserializer.class)
public enum MembershipType {
    NAVER, KAKAO, LINE;
}
```

어노테이션에 Deserializer 클래스를 명시하기 때문에 직관적이고 추적이 용이하다.<br>
하지만, 클래스마다 반복적으로 어노테이션을 추가해야 하므로 누락 가능성이 존재한다.

> 특정 클래스들에서 공통된 역직렬화 로직를 사용하는 경우에 사용하면 좋을 것 같다.

## Jackson Module을 이용한 Deserializer 등록
Deserializer 구현은 위와 동일하다.

### Module
`Module`은 Jackson의 확장을 위한 간단한 인터페이스이다.

Spring boot를 사용하는 경우 Module Bean을 등록하기만 하면 자동으로 `ObjectMapper`에 등록된다.

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Module customEnumDeserializerModule() {
        return new SimpleModule()
                .addDeserializer(Enum.class, new CustomEnumDeserializer(Enum.class));
    }
}
```

> 특정 타입에 대한 역직렬화 방식을 전역적으로 적용해야 하는 경우에 사용하면 좋을 것 같다.

---
**Reference**<br>
- https://d2.naver.com/helloworld/0473330
- https://medium.com/@taesulee93/spring-webmvc%EC%97%90%EC%84%9C-jackson-custom-serdes-serializer-deserializer-%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC-enum-%EB%98%91%EB%98%91%ED%95%98%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-with-jackson-9ee9af9f563c
- https://gengminy.tistory.com/49
- https://joon2974.tistory.com/entry/Java-%EA%B3%B5%ED%86%B5-%EC%A0%81%EC%9A%A9-JsonDeserializer-%EB%A7%8C%EB%93%A4%EC%96%B4-%EB%B3%B4%EA%B8%B0
