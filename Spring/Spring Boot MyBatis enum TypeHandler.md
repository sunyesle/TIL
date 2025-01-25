# Spring Boot MyBatis enum TypeHandler

DB에 enum 값을 저장할 때 enum의 상수 값을 그대로 저장하거나, 순서 값을 저장하는 게 아니라면 커스텀 `TypeHandler`가 필요하다.
> Mybatis 기본 제공 `TypeHandler`
> - `EnumTypeHandler`: enum 상수 값이 DB에 저장된다.
> - `EnumOrdinalTypeHandler`: enum 순서 값이 DB에 저장된다.

<br>

DB에 저장되는 코드 값과 enum이 매핑될 수 있도록, 커스텀 `TypeHandler`를 구현해 보자.

## CodeEnum
```java
public interface CodeEnum {
    String getCode();

    static <E extends Enum<E> & CodeEnum> E fromCode(Class<E> type, String code){
        return EnumSet.allOf(type)
                .stream()
                .filter(codeEnum -> codeEnum.getCode().equals(code))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("Invalid code value '" + code + "' for enum type " + type.getSimpleName()));
    }
}
```
- `getCode()`: DB에 저장할 코드 값을 반환한다.
- `fromCode()`: 코드 값에 해당하는 enum 상수를 반환한다.

## TypeHandler
`CodeEnum` 타입에 대한 `TypeHandler` 인터페이스를 구현한다.
```java
@MappedTypes(CodeEnum.class)
public class CodeEnumTypeHandler<E extends Enum<E> & CodeEnum> implements TypeHandler<CodeEnum> {

    private final Class<E> type;

    public CodeEnumTypeHandler(Class<E> type) {
        this.type = type;
    }

    @Override
    public void setParameter(PreparedStatement ps, int i, CodeEnum parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter.getCode());
    }

    @Override
    public CodeEnum getResult(ResultSet rs, String columnName) throws SQLException {
        return CodeEnum.fromCode(type, rs.getString(columnName));
    }

    @Override
    public CodeEnum getResult(ResultSet rs, int columnIndex) throws SQLException {
        return CodeEnum.fromCode(type, rs.getString(columnIndex));
    }

    @Override
    public CodeEnum getResult(CallableStatement cs, int columnIndex) throws SQLException {
        return CodeEnum.fromCode(type, cs.getString(columnIndex));
    }
}
```

## application.yml
MyBatis가 `TypeHandler` 클래스를 스캔할 패키지를 지정해 준다.
```yml
mybatis:
  type-handlers-package: com.sunyesle.spring_boot_mybatis.infra
```

## 예제
### CodeEnum 구현체
```java
public enum OrderStatus implements CodeEnum {
    PENDING("1000"),
    READY("1001"),
    SHIPPING("1002"),
    SHIPPED("1003"),
    CANCELED("2000");

    private final String code;

    OrderStatus(String code) {
        this.code = code;
    }

    @Override
    public String getCode() {
        return code;
    }
}
```

### VO
```java
@ToString
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class OrderVO {
    private Long orderId;
    private Long memberId;
    private Integer totalAmount;
    private OrderStatus orderStatus;

    public OrderVO(long memberId, int totalAmount, OrderStatus orderStatus) {
        this.memberId = memberId;
        this.totalAmount = totalAmount;
        this.orderStatus = orderStatus;
    }
}
```

### schema.sql
```SQL
DROP TABLE IF EXISTS orders;

CREATE TABLE orders (
    order_id BIGINT AUTO_INCREMENT,
    member_id BIGINT,
    total_amount INT,
    order_status VARCHAR(4),
    PRIMARY KEY (order_id)
);
```

### Mapper 인터페이스
```java
@Mapper
public interface OrderMapper {
    void insert(OrderVO order);

    List<OrderVO> selectAll();
}
```

### Mapper XML
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sunyesle.spring_boot_mybatis.order.OrderMapper">

    <insert id="insert" parameterType="OrderVO">
        INSERT INTO orders(member_id, total_amount, order_status)
        VALUES(#{memberId}, #{totalAmount}, #{orderStatus})
    </insert>

    <select id="selectAll" resultType="OrderVO">
        SELECT order_id, member_id, total_amount, order_status
        FROM orders
    </select>

</mapper>
```

### 테스트 코드
```java
@SpringBootTest
class OrderMapperTest {

    @Autowired
    private OrderMapper orderMapper;

    @Test
    void test() {
        OrderVO order1 = new OrderVO(1L, 1000, OrderStatus.READY);
        OrderVO order2 = new OrderVO(1L, 1000, OrderStatus.SHIPPING);

        orderMapper.insert(order1);
        orderMapper.insert(order2);

        List<OrderVO> orders = orderMapper.selectAll();

        assertThat(orders).hasSize(2);
        System.out.println(orders);
    }
}
```

테스트 코드를 실행시키고 로그를 확인해보면
```java
2025-01-25T19:42:12.572+09:00  INFO 17580 --- [spring-boot-mybatis] [    Test worker] p6spy : #1737801732572 | took 1ms | statement | connection 3| url jdbc:h2:mem:test
INSERT INTO orders(member_id, total_amount, order_status)
        VALUES(?, ?, ?)
INSERT INTO orders(member_id, total_amount, order_status)
        VALUES(1, 1000, '1001');

2025-01-25T19:42:12.573+09:00  INFO 17580 --- [spring-boot-mybatis] [    Test worker] p6spy : #1737801732573 | took 0ms | statement | connection 4| url jdbc:h2:mem:test
INSERT INTO orders(member_id, total_amount, order_status)
        VALUES(?, ?, ?)
INSERT INTO orders(member_id, total_amount, order_status)
        VALUES(1, 1000, '1002');

2025-01-25T19:42:12.583+09:00  INFO 17580 --- [spring-boot-mybatis] [    Test worker] p6spy : #1737801732583 | took 2ms | statement | connection 5| url jdbc:h2:mem:test
SELECT order_id, member_id, total_amount, order_status
        FROM orders
SELECT order_id, member_id, total_amount, order_status
        FROM orders;

[OrderVO(orderId=1, memberId=1, totalAmount=1000, orderStatus=READY), OrderVO(orderId=2, memberId=1, totalAmount=1000, orderStatus=SHIPPING)]
```
DB에 `CodeEnum`의 code 값이 저장되고, VO로 받아올 때 `CodeEnum`이 정상적으로 매핑되는 것을 확인할 수 있다.

---
**Reference**<br>
- https://goodgid.github.io/MyBatis-Handling-TypeHandler-Enum/
- https://velog.io/@dondonee/MyBatis-Enum-%ED%83%80%EC%9E%85-%EB%A7%A4%ED%95%91%ED%95%98%EA%B8%B0-TypeHandler
- https://www.podo-dev.com/blogs/120
