# Lombok @Accessors 어노테이션
## 기본
일반적인 getter 및 setter 메서드를 추가한다.
```java
@Getter
@Setter
public class StandardItem {
    private String name;
    private int price;
}
```

```java
@Test
void standardAccessors() {
    StandardItem item = new StandardItem();
    item.setName("Standard Item");
    item.setPrice(1000);

    assertThat(item.getName()).isEqualTo("Standard Item");
    assertThat(item.getPrice()).isEqualTo(1000);
}
```

## fluent 옵션
get 또는 set 접두사가 없는 접근자를 사용할 수 있다.
```java
@Getter
@Setter
@Accessors(fluent = true)
public class FluentItem {
    private String name;
    private int price;
}
```

```java
@Test
void fluentAccessors() {
    FluentItem item = new FluentItem();
    item.name("Fluent Item");
    item.price(1000);

    assertThat(item.name()).isEqualTo("Fluent Item");
    assertThat(item.price()).isEqualTo(1000);
}
```

## chain 옵션
setter가 자기 자신(`this`)을 반환한다.<br>
만약 `fluent` 옵션이 `true`인 경우, `chain` 옵션을 명시하지 않으면 기본값이 `true`가 된다.
```java
@Getter
@Setter
@Accessors(chain = true)
public class ChainedItem {
    private String name;
    private int price;
}
```

```java
@Test
void chainedAccessors() {
    ChainedItem item = new ChainedItem()
            .setName("Chained Item")
            .setPrice(1000);

    assertThat(item.getName()).isEqualTo("Chained Item");
    assertThat(item.getPrice()).isEqualTo(1000);
}
```

<br>

그 외에도 다음과 같은 옵션이 존재한다.
- `prefix`: getter 및 setter에서 어떤 접두사를 무시할지 지정한다.
- `makeFinal`: getter 및 setter를 `final` 메서드로 만들어준다.

---
**Reference**
- https://www.baeldung.com/lombok-accessors
