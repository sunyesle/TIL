# @ExceptionHandler, @ControllerAdvice

## 컨트롤러 내 예외처리
`@ExceptionHandler`를 사용하여 특정 컨트롤러 내에서 예외 처리를 할 수 있다.

```java
@RestController
@RequestMapping("/api/v1/members")
@RequiredArgsConstructor
public class MemberController {

    @GetMapping("/exception1")
    public void exception1() {
        throw new NullPointerException();
    }

    @GetMapping("/exception2")
    public void exception2() {
        throw new IllegalArgumentException();
    }

    @ExceptionHandler(NullPointerException.class)
    public String handleNullPointerException(NullPointerException ex) {
        return "NullPointerException!";
    }
}
```

여러 개의 예외를 지정할 수도 있다.
```java
@ExceptionHandler({NullPointerException.class, IllegalArgumentException.class})
public String handle(Exception ex) {
    return "Exception!";
}
```

## 글로벌 예외 처리
> ### @ControllerAdvice
> 컨트롤러 전반에 걸쳐 `@ExceptionHandler`(예외 처리), `@InitBinder`(바인딩 설정), `@ModelAttribute`(모델 객체) 메서드를 적용하는 기능을 제공한다.

`@ControllerAdvice`를 이용하여 전역적으로 예외 처리를 할 수 있다.<br>
컨트롤러 클래스에서 예외 처리 로직을 분리함으로써 컨트롤러 역할에 집중할 수 있고, 중복되는 예외 처리 코드를 해결할 수 있다.

`@RestControllerAdvice` = `@ResponseBody` + `@ControllerAdvice`

`@ControllerAdvice`는 기본적으로 모든 컨트롤러에 적용된다.
특정한 컨트롤러에만 적용하고 싶으면 다음과 같이 대상 컨트롤러를 지정할 수 있다.
```java
@RestControllerAdvice(basePackageClasses = MemberController.class)

@RestControllerAdvice(basePackages = "com.example.demo.controller")

@RestControllerAdvice(annotations = Controller.class)
```

### 예제
다음과 같은 두 개의 컨트롤러가 있다.<br>
컨트롤러에서 exception을 발생시키지만, 별도의 예외 처리 로직은 없다.
```java
@RestController
@RequestMapping("/members")
public class MemberController {

    @GetMapping("/exception1")
    public void exception1() {
        throw new NullPointerException();
    }

    @GetMapping("/exception2")
    public void exception2() {
        throw new IllegalArgumentException();
    }
}

@RestController
@RequestMapping("/products")
public class ProductController {

    @GetMapping("/exception1")
    public void exception1() {
        throw new NullPointerException();
    }

    @GetMapping("/exception2")
    public void exception2() {
        throw new IllegalArgumentException();
    }
}
```

컨트롤러에서 발생하는 예외를 공통적으로 처리하기 위해 `@RestControllerAdvice`를 사용하여 전역 예외 처리 클래스를 작성한다.
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler({NullPointerException.class, IllegalArgumentException.class})
    public String handleException(Exception ex) {
        return "Exception!";
    }
}
```

**요청 결과**<br>
```log
Request method:	GET
Request URI:	http://localhost:8080/members/exception1

HTTP/1.1 200 
Content-Type: text/plain;charset=UTF-8
Content-Length: 17
Date: Tue, 14 Jan 2025 13:50:41 GMT
Keep-Alive: timeout=60
Connection: keep-alive

Exception!
```

```log
Request method:	GET
Request URI:	http://localhost:8080/products/exception2

HTTP/1.1 200 
Content-Type: text/plain;charset=UTF-8
Content-Length: 17
Date: Tue, 14 Jan 2025 13:53:48 GMT
Keep-Alive: timeout=60
Connection: keep-alive

Exception!
```

---
**Reference**<br>
- https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-advice.html
- https://incheol-jung.gitbook.io/docs/q-and-a/spring/controlleradvice-exceptionhandler
- https://ttl-blog.tistory.com/1280
- https://devdebin.tistory.com/172
- https://parkmuhyeun.github.io/woowacourse/2023-04-19-Exception-Handler/
