## Controller의 책임 범위

- HTTP 요청을 수신한다.
- HTTP 요청을 파싱하여 URL, body, param 등의 데이터를 추출하고 이를 Java 객체로 변환(deserialize)한다. 
- 입력 데이터의 유효성을 검증한다.
- 비즈니스 로직을 호출한다.
- 비즈니스 로직의 결과를 HTTP 응답으로 직렬화(serialize)하여 반환한다.
- 처리 도중 예외가 발생하면, 사용자에게 의미 있는 오류 메시지와 적절한 HTTP 상태 코드를 포함한 응답을 반환한다.

---
### Reference

https://hello-woody.medium.com/spring-mvc-%EC%97%90%EC%84%9C-controller-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C%EC%97%90%EC%84%9C%EB%8A%94-%EB%AD%98-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C-d398d5b4a35f
