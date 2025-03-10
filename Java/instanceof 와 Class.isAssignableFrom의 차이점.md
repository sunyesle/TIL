# instanceof와 Class.isAssignableFrom의 차이점

- `instanceof`: 특정 **Object**가 어떤 클래스/인터페이스를 상속/구현했는지 체크한다.
- `Class.isAssignableFrom()`: 특정 **Class**가 어떤 클래스/인터페이스를 상속/구현했는지 체크한다.

## 예시
```java
class Parent{}
class Child extends Parent{}
```

### instanceof
```java
Child child = new Child();
if(child instanceof Parent) {
    // ...
}
```

### Class.isAssignableFrom()
```java
// Class.isAssignableFrom()
if(Parent.class.isAssignableFrom(Child.class)) {
    // ...
}
```

---
**Reference**<br>
- https://jistol.github.io/java/2017/08/22/different-instanceof-isassignablefrom/
