# JPA 복합키 설정
**복합키**란 두 개 이상의 열을 조합하여 기본키를 구성하는 것이다.

복합키 설정 방법에는 `@IdClass`, `@EmbeddedId` 두 가지 방식이 있다. 

## 복합키 클래스 정의
- `Serializable`을 구현해야 한다.
- `equals()`와 `hashCode()` 메서드를 오버라이딩해야 한다.
- 기본 생성자(매개변수가 없는 생성자)가 있어야 한다.
```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
public class OrderItemId implements Serializable {
    private String orderNumber;
    private String itemCode;
}
```

## @IdClass
엔티티 클래스에 복합키 필드들을 일반 필드처럼 나열하고, `@Id` 어노테이션을 추가한다.
```java
@Entity
@IdClass(OrderItemId.class) // 복합키 클래스 지정
@Getter
@NoArgsConstructor
public class OrderItem {
    @Id
    private String orderNumber;
    @Id
    private String itemCode;
    private Integer quantity;

    public OrderItem(String orderNumber, String itemCode, Integer quantity) {
        this.orderNumber = orderNumber;
        this.itemCode = itemCode;
        this.quantity = quantity;
    }
}
```

## @EmbeddedId
복합키 클래스에 `@Embeddable` 어노테이션을 추가한다.
```java
@Embeddable // 추가
@Getter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
public class OrderItemId implements Serializable {
    private String orderNumber;
    private String itemCode;
}
```

`@EmbeddedId` 어노테이션을 사용하여 복합키를 엔티티 내부에 추가한다.
```java
@Entity
@Getter
@NoArgsConstructor
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
    private Integer quantity;

    public OrderItem(String orderNumber, String itemCode, Integer quantity) {
        this.id = new OrderItemId(orderNumber, itemCode);
        this.quantity = quantity;
    }

    // 편의 메서드
    public String getOrderNumber(){
        return id.getOrderNumber();
    }

    public String getItemCode( ) {
        return id.getItemCode();
    }
}
```

## 선택 기준
- 복합키의 각 부분을 개별적으로 접근하는 일이 많다면 `@IdClass`
- 복합키 객체를 통째로 사용하는 일이 많다면 `@EmbeddedId`

---
**Reference**
- https://leapday.tistory.com/23
- https://www.baeldung.com/jpa-composite-primary-keys
