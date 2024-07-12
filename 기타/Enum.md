# Enum

## 열거타입마다 다르게 동작해야 할 때

### switch문
```java
public enum OperationV1 {
    PLUS("+"),
    MINUS("-"),
    TIMES("*"),
    DIVIDE("/");

    private final String symbol;

    OperationV1(String symbol) {
        this.symbol = symbol;
    }

    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("Unknown op: " + this);
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```
코드는 잘 동작하지만, 새로운 상수를 추가해도 컴파일러가 switch문에 해당 상수를 추가했는지 검증하지 않으므로 런타임 오류가 발생할 수 있다.

### 상수별 메서드 구현(constant-specific method implementation)
```java
public enum OperationV2{
    PLUS("+"){ public double apply(double x, double y) {return x+y;}},
    MINUS("-"){ public double apply(double x, double y) {return x-y;}},
    TIMES("*"){ public double apply(double x, double y) {return x*y;}},
    DIVIDE("/"){ public double apply(double x, double y) {return x/y;}};

    private final String symbol;

    OperationV2(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double x, double y);

    @Override
    public String toString() {
        return symbol;
    }
}
```
열거 타입에 추상 메서드를 선언 후 상수별로 클래스 몸체(constant-specific class body)를 재정의한다. 컴파일 타임에 각 열거 상수마다 apply 메서드가 구현되어 있는지 검증해준다.

### 람다식(Lambda Expression)
```java
public enum OperationV3 {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    OperationV3(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```
람다식을 사용하여 코드를 간결하게 만들 수 있다.

<br>

다음과 같이 사용할 수 있다.
```java
@Test
void operationTest(){
    double x = 2D;
    double y = 4D;

    for (Operation op : Operation.values()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
    }
}
```
> 출력
```
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```

---
**Reference**
- https://catsbi.oopy.io/4678b976-bd7e-4353-b4f0-04c06f66df03
- https://jjingho.tistory.com/91
- https://happyer16.tistory.com/entry/6-Enums-and-Annotations-Effective-Java-3th
