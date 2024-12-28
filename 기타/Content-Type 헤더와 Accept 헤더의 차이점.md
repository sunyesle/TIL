# Content-Type 헤더와 Accept 헤더의 차이점
## Content-Type
**HTTP 메시지에 담기는 데이터 타입을 알려주는 헤더**

대부분의 HTTP 표준 스펙을 따르는 브라우저와 웹서버는 우선적으로 Content-Type 헤더를 기준으로 메시지에 담긴 데이터를 분석하고 파싱한다.

Content-Type 헤더는 POST나 PUT처럼 메시지 Body에 데이터를 보낼 때 필요로한다.<br>
GET 요청의 Body가 없고, 데이터가가 쿼리 스트링을 통해 전달되기 때문에 Content-Type 헤더가 필요 없다.

## Accept
**클라이언트가 서버에게 자신이 받을 수 있는 데이터 타입이 무엇인지 알려주기 위한 헤더**

웹서버는 Accept 헤더를 기준으로 클라이언트가 지원하는 데이터 형식에 맞춰 응답을 생성한다.

## 정리
둘 다 데이터 타입(MIME)을 다루는 헤더지만

Content-Type 헤더는 서버나 클라이언트가 보내는 데이터 형식을 지정하는 것이고,<br>
Accept 헤더는 클라이언트가 응답으로 받고 싶은 형식을 지정하는 것이다.

---
**Reference**<br>
- https://dololak.tistory.com/630
- https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Type
- https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Accept
