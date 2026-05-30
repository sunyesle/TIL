# record

`record` 클래스는 **불변 데이터 객체**를 쉽게 만들 수 있도록 하는 특수한 형태의 클래스다.

## 특징
- 멤버 변수는 `private final`로 선언된다.
- 필드별 `getter`가 자동으로 생성된다.
- 모든 멤버변수를 인자로 하는 `public` 생성자가 자동으로 생성된다.
- `equals`, `hashcode`, `toString` 메서드가 자동으로 생성된다.
- 기본 생성자는 제공하지 않으므로 필요한 경우 직접 생성해야 한다.
- `static` 필드와 메서드를 선언할 수 있다.
- 다른 클래스를 상속(`extends`) 받을 수 없다. 인터페이스 구현(`implements`)은 가능하다.
- 암시적으로 `final`이며 `abstract`일 수 없다.

## 예시
`record` 클래스와 일반 클래스로 불변 객체를 구현한 예시이다. `record` 클래스를 사용하면 보일러플레이트 코드가 자동으로 생성되므로 간결하게 불변 객체를 정의할 수 있다.

### record 클래스
```java
public record Person (String name, String address) {}
```

### 일반 클래스
```java
public class Person {
    private final String name;
    private final String address;

    public Person(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public String getAddress() {
        return address;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name) && Objects.equals(address, person.address);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, address);
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```
