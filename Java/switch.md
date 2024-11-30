# switch

## JDK 14
### Switch expressions

#### Arrow Cases
실행문을 화살표로(`->`) 표시하며, 다중 조건으로 판단해 구문을 실행할 수 있다. 또한 switch에서 값을 산출해 낼 수 있다.

```java
public enum Day { SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY; }

static int testOld(Day day) {
    int num = 0;
    switch (day){
        case SUNDAY:
        case FRIDAY:
        case SATURDAY:
            num = 1;
            break;
        case MONDAY:
            num = 3;
            break;
        case TUESDAY:
        case THURSDAY:
        case WEDNESDAY:
            num= 2;
            break;
        default:
            throw new IllegalArgumentException();
    }
    return num;
}

static int testNew(Day day) {
    return switch (day) {
        case SUNDAY, FRIDAY, SATURDAY -> 1;
        case MONDAY -> 3;
        case TUESDAY, THURSDAY, WEDNESDAY -> 2;
    };
}
```

#### Colon Cases and the the yield Statement
기존의 switch문에서도 `yield` 키워드를 통해 값을 산출할 수 있다.
```java
static int yieldTest(String s) {
    return switch (s) {
        case "Foo":
            yield 1;
        case "Bar":
            yield 2;
        default:
            yield 0;
    };
}
```

#### Exhaustiveness of switch
가능한 모든 케이스를 정의하도록 강제한다.
```java
static int test(Day day) {
    return switch (day) { // 컴파일 에러: the switch expression does not cover all possible input values
        case SUNDAY:
            System.out.println("dfd");;
    };
}
```

## JDK 21

### Pattern Matching with switch
switch를 이용한 패턴 매칭을 지원한다.
```java
interface Shape {}
record Rectangle(double length, double width) implements Shape {}
record Circle(double radius) implements Shape {}

static double getPerimeter(Shape s) {
    return switch (s) {
        case Rectangle r -> 2 * r.length() + 2 * r.width();
        case Circle c -> 2 * c.radius() * Math.PI;
        default -> throw new IllegalArgumentException("Unrecognized shape");
    };
}
```

#### When Clauses
pattern label 뒤에 `when` 구문을 사용하여 조건식을 추가할 수 있다.
```java
static void testOld(Object obj) {
    switch (obj){
        case String s -> {
            if(s.length() == 1) {
                System.out.println("short: " + s);
            }else {
                System.out.println(s);
            }
        }
        default -> System.out.println("Not a string");
    }
}

static void testNew(Object obj) {
    switch (obj){
        case String s when s.length() == 1 -> System.out.println("short: " + s);
        case String s -> System.out.println(s);
        default -> System.out.println("Not a string");
    }
}
```

#### Null case Labels
이전까지는 switch문에 입력한 변수가 null이라면 NullPointerException을 던졌지만,<br>
JDK 21 이후부터는 null case label을 사용하여 switch문 내에서 null 처리를 할 수 있다.
```java
void switchNullLabelTestOld(Object obj) {
    if(obj == null){
        System.out.println("null!");
        return;
    }

    switch (obj) {
        case String s -> System.out.println("String: " + s);
        default -> System.out.println("Something else");
    }
}

void switchNullLabelTest(Object obj) {
    switch (obj) {
        case null -> System.out.println("null!");
        case String s -> System.out.println("String: " + s);
        default -> System.out.println("Something else");
    }
}

// null은 default와만 결합할 수 있다.
void switchNullLabelTest2(Object obj) {
    switch (obj) {
        case String s -> System.out.println("String: " + s);
        case null, default -> System.out.println("null or not a string");
    }
}
```

---
**Reference**<br>
- https://docs.oracle.com/en/java/javase/23/language/switch-expressions-and-statements.html
- https://gngsn.tistory.com/254?category=879289
- https://docs.oracle.com/en/java/javase/23/language/pattern-matching.html
