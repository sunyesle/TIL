# Enum, EnumMap, EnumSet

## Enum
-   열거 타입(`enum`)은 클래스이다.
-   상수 하나당 자신의 인스턴스를 만들어 `public static final` 필드로 공개한다.
-   생성자는 `private`이다.
-   클라이언트에서 인스턴스를 생성하거나 확장할 수 없으므로, 열거 타입으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다. (싱글턴)
-   컴파일 타임에 타입 안전성을 제공한다.
-   `namespace`가 있어서 이름이 같은 상수도 공존할 수 있다.
-   임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
-   자신 안의 정의된 상수들의 값을 배열에 담아 반환하는 정적메서드 `values()`를 제공한다.

### 열거 타입마다 다르게 동작해야 하는 경우
#### switch문
코드는 잘 동작하지만, 새로운 상수를 추가해도 컴파일러가 `switch`문에 해당 상수를 추가했는지 검증하지 않으므로 런타임 오류가 발생할 수 있다.
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

#### 상수별 메서드 구현(constant-specific method implementation)
열거 타입에 추상 메서드를 선언 후 상수별로 클래스 몸체(constant-specific class body)를 재정의한다. 컴파일 타임에 각 열거 상수마다 `apply()` 메서드가 구현되어 있는지 검증해준다.
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

#### 람다식(Lambda Expression)
람다식을 사용하여 코드를 간결하게 만들 수 있다.
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

<br>

```java
@Test
void operationTest(){
    double x = 2D;
    double y = 4D;

    for (Operation op : Operation.values()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
    }
    // 2.000000 + 4.000000 = 6.000000
    // 2.000000 - 4.000000 = -2.000000
    // 2.000000 * 4.000000 = 8.000000
    // 2.000000 / 4.000000 = 0.500000
}
```

## EnumMap
`EnumMap`은 열거 타입을 키로 사용하도록 최적화된 `Map` 구현체이다. 내부적으로 ordinal을 인덱스로 하는 배열을 사용하기 때문에 성능이 뛰어나다.

### 예시
`Plant` 클래스는 내부적으로 식물의 생애주기를 나타내는 `enum` 필드를 갖는다.
```java
public class Plant {
    enum LifeCycle {
        ANNUAL, PERENNIAL, BIENNIAL;
    }

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

이러한 식물들을 생애주기를 기준으로 집합을 만들어 관리하고 싶다고 하자.
```java
private final Plant[] garden = {
        new Plant("바질", Plant.LifeCycle.ANNUAL),
        new Plant("캐러웨이", Plant.LifeCycle.BIENNIAL),
        new Plant("딜", Plant.LifeCycle.ANNUAL),
        new Plant("라벤더", Plant.LifeCycle.PERENNIAL),
        new Plant("파슬리", Plant.LifeCycle.BIENNIAL),
        new Plant("로즈마리", Plant.LifeCycle.PERENNIAL),
};
```

다음과 같이 `EnumMap`을 활용할 수 있다.
```java
@Test
void enumMapTest(){
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

    for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
        plantsByLifeCycle.put(lc, new HashSet<>());
    }

    for (Plant p : garden) {
        plantsByLifeCycle.get(p.lifeCycle).add(p);
    }

    System.out.println(plantsByLifeCycle);
    // {ANNUAL=[딜, 바질], PERENNIAL=[라벤더, 로즈마리], BIENNIAL=[캐러웨이, 파슬리]}
}
```

`Stream`을 사용해 `Map`을 관리하면 코드를 더 줄일 수 있다.
```java
@Test
void enumMapStreamTest() {
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = Arrays.stream(garden)
            .collect(groupingBy(
                            p -> p.lifeCycle,                           // 분류 기준 (Key)
                            () -> new EnumMap<>(Plant.LifeCycle.class), // 맵 구현체 지정
                            toSet()                                     // 컬렉션 작업 (Value)
                    )
            );
    System.out.println(plantsByLifeCycle);
    // {ANNUAL=[딜, 바질], PERENNIAL=[라벤더, 로즈마리], BIENNIAL=[캐러웨이, 파슬리]}
}
```
## EnumSet
`EnumSet`은 열거 타입의 요소들을 집합으로 다루는데 최적화된 `Set` 구현체이다. 내부적으로 비트 벡터로 구현되어 있어 성능이 뛰어나다. `null` 값을 추가하는 것을 허용하지 않으며, `null` 값을 추가하려고 하면 `NullPointerException`이 발생한다.

### 예시
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

```java
@Test
void enumSetTest(){
    // of() : 특정 요소 포함
    EnumSet<Day> weekend = EnumSet.of(Day.SATURDAY, Day.SUNDAY);
    System.out.println(weekend);
    // [SATURDAY, SUNDAY]

    // allOf() : 모든 요소 포함
    EnumSet<Day> allDays = EnumSet.allOf(Day.class);
    System.out.println(allDays);
    // [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]

    // noneOf() : 비어있는 EnumSet
    EnumSet<Day> studyDays = EnumSet.noneOf(Day.class);

    // range() : 특정 범위의 요소 포함
    EnumSet<Day> weekdays = EnumSet.range(Day.MONDAY, Day.FRIDAY);
    System.out.println(weekdays);
    // [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY]

    // complementOf() : 특정 요소를 제외한 나머지 요소 포함
    EnumSet<Day> restDays = EnumSet.complementOf(weekdays);
    System.out.println(restDays);
    // [SATURDAY, SUNDAY]
}
```

---
**Reference**
- https://catsbi.oopy.io/4678b976-bd7e-4353-b4f0-04c06f66df03
- https://jjingho.tistory.com/91
- https://happyer16.tistory.com/entry/6-Enums-and-Annotations-Effective-Java-3th
- https://dahye-jeong.gitbook.io/java/java/effective_java/2021-06-06-use-enummap
- https://www.baeldung.com/java-enumset
- https://honbabzone.com/java/java-enum/
