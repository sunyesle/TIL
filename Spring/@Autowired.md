# @Autowired
@Autowired를 통한 의존성 주입 방식

## 생성자 주입
필수 의존 관계에 사용한다.
스프링 프레임워크에서 권장하는 방식이다.

**장점**<br>
- 불변성 보장. 필드를 final로 선언 가능하다.
- 필수적인 의존성의 주입을 보장할 수 있다.
- 애플리케이션 구동 시점에 순환 참조 오류를 감지할 수 있다.
- 테스트 코드 작성이 용이하다.
```java
@Component
public class MyExample {
    // final로 선언할 수 있다.
    private final HelloService helloService;

    // 단일 생성자인 경우는 추가적인 어노테이션이 필요 없다.
    public MyExample(HelloService helloService) {
        this.helloService = helloService;
    }
}
```

## 수정자 주입
선택적 의존 관계, 의존 관계가 동적으로 변경될 수 있을 때 사용한다.
```java
@Component
public class MyExample {
    private HelloService helloService;

    @Autowired
    public void setHelloService(HelloService helloService) {
        this.helloService = helloService;
    }
}
```

## 필드 주입
간단한 애플리케이션이나 빠르게 개발해야 하는 상황에서 사용한다. (테스트 코드, 샘플 코드, 등)
```java
@Component
public class MyExample {
    @Autowired
    private HelloService helloService;
}
```
