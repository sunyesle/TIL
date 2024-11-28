# instanceof와 패턴 매칭

```java
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s);
}
```

## JDK 16
### Pattern Matching for instanceof
타입 비교 후 자동으로 타입 캐스팅된 값을 할당한다.
```java
if (obj instanceof String s) {
    System.out.println(s);
}
```
if문 내에서 String s를 사용할 수도 있다.
```java
if (obj instanceof String s && s.length() > 2) {
    System.out.println(s);
}
```

## JDK 21
### Record Patterns
타입 비교 후 Record 의 내부 필드 값을 추출하여 할당한다.
```java
public record Point(int x, int y) {}

if (obj instanceof Point(int x, int y)) {
    System.out.println(x + y);
}
```
중첩된 Record에도 적용할 수 있다.
```java
public record GPS(double latitude, double longitude) {}
public record Location(String name, GPS gps) {}

if (obj instanceof Location(String name, GPS(double latitude, double longitude))) {
    System.out.printf("%s is located at (%f, %f)", name, latitude, longitude);
}
```

---
**Reference**<br>
- https://docs.oracle.com/en/java/javase/23/language/pattern-matching-instanceof.html
- https://docs.oracle.com/en/java/javase/23/language/record-patterns.html
- https://devel-repository.tistory.com/26

