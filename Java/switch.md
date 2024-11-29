# switch

## JDK14
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

---
**Reference**<br>
- https://docs.oracle.com/en/java/javase/23/language/switch-expressions-and-statements.html
- https://gngsn.tistory.com/254?category=879289
