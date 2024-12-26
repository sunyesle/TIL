# HttpMessageConverter

`HttpMessageConverter`는 Spring MVC에서 요청과 응답을 변환하는 인터페이스다.<br>
HTTP 요청 본문을 Java 객체로 변환하거나, Java 객체를 HTTP 응답 본문으로 변환할 때 사용된다.

- **HTTP 요청**: `@RequestBody`, `HttpEntity(RequestEntity)`
- **HTTP 응답**: `@ResponseBody`, `HttpEntity(ResponseEntity)`

## HttpMessageConverter 인터페이스

```java
public interface HttpMessageConverter<T> {
    boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

    boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

    List<MediaType> getSupportedMediaTypes();

    default List<MediaType> getSupportedMediaTypes(Class<?> clazz) {
        return !this.canRead(clazz, (MediaType)null) && !this.canWrite(clazz, (MediaType)null) ? Collections.emptyList() : this.getSupportedMediaTypes();
    }

    T read(Class<? extends T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException;

    void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException;
}
```
- `canRead()`, `canWrite()` : 메시지 컨버터가 해당 클래스, 미디어 타입을 지원하는지 체크한다.
- `read()`, `write()` : 메시지 컨버터를 통해서 메시지를 읽고 쓴다.

## HttpMessageConverter 종류
스프링은 다양한 메시지 컨버터를 제공한다.

스프링은 등록된 메시지 컨버터 목록을 **우선순위**대로 검토하며,
대상 **클래스 타입**과 **미디어 타입**을 기준으로 적합한 메시지 컨버터를 선택한다.

주요한 메시지 컨버터 구현체를 알아보자.

### ByteArrayHttpMessageConverter
- **클래스 타입**: `byte[]`, **미디어 타입**: `*/*`
- 요청 및 응답 예시 
  - `@RequestBody byte[] data`
  - `@ResponseBody return byte[]` 쓰기 미디어 타입은 `application/octet-stream`으로 반환된다.

### StringHttpMessageConverter
- **클래스 타입**: `String`, **미디어 타입**: `*/*`
- 요청 및 응답 예시
    - `@RequestBody String data`
    - `@ResponseBody return "ok"` 쓰기 미디어 타입은 `text/plain`으로 반환된다.

### MappingJackson2HttpMessageConverter
- **클래스 타입**: `객체` 또는 `HashMap`, **미디어 타입**: `application/json` 관련
- 요청 및 응답 예시
    - `@RequestBody RequestMyData data`
    - `@ResponseBody return data` 쓰기 미디어 타입은 `application/json`으로 반환된다.

> 자세한 HttpMessageConverter 구현체 정보는 [공식 문서](https://docs.spring.io/spring-framework/reference/web/webmvc/message-converters.html)에서 확인 가능하다.

<br>

<img width="640" alt="RequestMappingHandlerAdapter" src="https://github.com/user-attachments/assets/96b5ec9d-7af8-433b-a3f6-b96c42707581" />

> 지원하는 컨트롤러 메서드 인수 [공식 문서](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html)

> 지원하는 컨트롤러 메서드 반환값 [공식 문서](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/return-types.html)

---
**Reference**<br>
- https://velog.io/@woply/spring-메시지-바디의-데이터-처리를-담당하는-HttpMessageConverter
- https://kuidoli.tistory.com/16
- https://yo00ong.tistory.com/162
