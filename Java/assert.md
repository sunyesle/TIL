# assert
Java assert 키워드는 Java 1.4에 추가되었다.
주로 개발 중에 논리적 오류를 검증하는 용도로 사용된다.

- 사전 조건, 사후 조건 및 내부 코드의 불변성을 확인할 때 사용한다.
- 공개 메서드에는 assert를 사용하는 것을 피하고, 대신 예외를 사용한다.
- 개발 도구로 프로덕션에서는 비활성화해야 한다.

> assert : 절대로 일어나서는 안될 일<br>
> exception : 일어날 법한 일

## assert 활성화
이전 버전과의 호환성을 위해 JVM은 기본적으로 어설션 검증을 비활성화한다.<br>
assert를 사용하기 위해서는 실행 시 -ea(-enableassertions) 옵션을 사용하여 명시적으로 활성화해야 한다.

## assert 사용법
assert 키워드는 다음과 같은 형태로 사용한다.<br>
조건문이 false일 경우 AssertionError를 던진다.
```java
assert (조건문);
assert (조건문) : "에러 메시지";
```

## 예시
```java
public class AssertTest {
    public static void main(String[] args) {
        sendMessage(null);
    }

    public static void sendMessage(String message){
        assert message != null : "message is null";
        System.out.println(message);
    }
}
```

위 코드 실행 시 다음과 같은 오류가 발생한다. 
```log
Exception in thread "main" java.lang.AssertionError: message is null
	at com.sunyesle.javastudy.assertTest.sendMessage(assertTest.java:9)
	at com.sunyesle.javastudy.assertTest.main(assertTest.java:5)
```

---
**Reference**<br>
- https://www.baeldung.com/java-assert
- https://www.datacamp.com/doc/java/assert
- https://limecoding.tistory.com/250
- https://puleugo.tistory.com/177
- https://dkswnkk.tistory.com/710
- https://velog.io/@urtimeislimited/Java-Assertion-vs-Exception
