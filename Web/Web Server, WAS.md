# Web Server, WAS
## Web Server
웹 서버는 클라이언트로부터 HTTP 요청을 받아서 **정적인 콘텐츠**를 제공해 주는 서버이다.

대표적인 웹 서버로는 Apache HTTP Server, Nginx 등이 있다.

## WAS (Web Application Server)
WAS는 웹서버에서 단독으로 처리할 수 없는 **동적인 콘텐츠**를 제공해 주는 미들웨어이다.

자바 기반 WAS는 대부분 Java EE(Java Enterprise Edition) 스펙을 구현하여 동적인 웹 애플리케이션 실행 환경을 제공한다.
Java EE에서는 일반적으로 WAS를 다음과 같이 나눈다.
- **웹 컨테이너**: Servlet/JSP 실행 환경을 제공하는 모듈이다. 서블릿의 생명주기를 관리하고, URL과 특정 서블릿을 맵핑하며 URL 요청이 올바른 접근 권한을 갖도록 보장하며, 서블릿 컨테이너라고도 부른다.
- **EJB 컨테이너**: 애플리케이션의 비즈니스 로직을 갖고있는 모듈이다.
 
전체 Java EE 스펙을 구현한 WAS로는 JBoss, WebLogic 등이 있다.

흔히 Tomcat을 WAS라고 부르지만, Tomcat은 Java EE 스펙 중 Servlet/JSP 실행 환경만 구현하였기 때문에 웹 컨테이너(서블릿 컨테이너)에 가깝다.
비즈니스 로직을 실행하기 위한 EJB 같은 기능은 없지만, Spring 프레임워크 등과 결합하면 WAS로 활용할 수 있다.

---
**Reference**<br>
- https://ko.wikipedia.org/wiki/웹_애플리케이션_서버
- https://starplaying.tistory.com/274
