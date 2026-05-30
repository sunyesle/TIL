# try-with-resources
**try-with-resources**는 `try` 괄호 안에서 선언된 자원들에 대해서 `try` 블록이 종료될 때 자동으로 자원을 닫아주는 기능이다. 이때 자원은 **`AutoCloseable` 인터페이스를 구현**하고 있어야 한다. `AutoCloseable` 인터페이스에는 `close()` 메서드가 정의되어 있으며 `try` 블록이 종료될 때 자동으로 객체의 `close()` 메서드가 호출된다.

## 장점
- **자원 누수 방지**: `try` 블록이 종료되면 자동으로 자원을 닫아주므로, 개발자가 자원을 명시적으로 닫지 않아도 된다.
- **코드의 가독성 향상**: 자원 관리를 위한 코드가 줄어든다.
- **예외 처리 개선**: 자원을 닫는 과정에서 발생하는 예외를 `try` 블록에서 발생하는 예외와 함께 관리할 수 있다.

## 비교
### try-catch-finally
```java
@Test
void tryCatchFinallyTest() {
    FileInputStream is = null;
    BufferedInputStream bis = null;
    try {
        is = new FileInputStream("testFile.txt");
        bis = new BufferedInputStream(is);
        int data = -1;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    } catch (IOException e) {
        throw new RuntimeException(e);
    } finally {
        if (is != null) try { is.close(); } catch (IOException e) { e.printStackTrace(); }
        if (bis != null) try { bis.close(); } catch (IOException e) { e.printStackTrace(); }
    }
}
```

### try-with-resources
```java
@Test
void tryWithResourcesTest() {
    try (FileInputStream is = new FileInputStream("/test/file.txt");
         BufferedInputStream bis = new BufferedInputStream(is)) {

        int data = -1;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
여러 개의 자원을 사용하는 경우 세미콜론으로 구분하여 선언한다.

## AutoCloseable
```java
public interface AutoCloseable { 
    void close() throws Exception;
}
```
`close()` 메서드는 `Exception`을 발생시키도록 선언되었지만, 실제 클래스를 구현할 때는 더 구체적인 예외를 발생시키거나 닫기 작업이 실패할 수 없는 경우 예외를 발생시키지 않도록 구현하는 것이 권장된다.

### 구현 예시
```java
class AutoCloseableResourcesA implements AutoCloseable {
    public AutoCloseableResourcesA() {
        System.out.println("Constructor - A");
    }

    public void doSomething() {
        System.out.println("Something - A");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closed A");
    }
}
```
```java
class AutoCloseableResourcesB implements AutoCloseable {
    public AutoCloseableResourcesB() {
        System.out.println("Constructor - B");
    }

    public void doSomething() {
        System.out.println("Something - B");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closed B");
    }
}
```

**테스트 코드**
```java
@Test
void autoCloseableTest() throws Exception {
    try (AutoCloseableResourcesA a = new AutoCloseableResourcesA();
         AutoCloseableResourcesB b = new AutoCloseableResourcesB()) {

        a.doSomething();
        b.doSomething();
    }
}
```
> 출력
```txt
Constructor - A
Constructor - B
Something - A
Something - B
Closed B
Closed A
```

---
**Reference**<br>
- https://codechacha.com/ko/java-try-with-resources/
- https://imksh.com/112
- https://f-lab.kr/insight/understanding-try-with-resources-in-java
- https://www.baeldung.com/java-try-with-resources
- https://docs.oracle.com/en%2Fjava%2Fjavase%2F21%2Fdocs%2Fapi%2F%2F/java.base/java/lang/AutoCloseable.html
