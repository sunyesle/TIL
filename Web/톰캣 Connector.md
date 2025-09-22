# 톰캣의 Connector
Coyote Connector는 톰캣의 핵심 구성 요소 중 하나로 클라이언트와의 통신을 처리한다.

## 동작 흐름
<img width="829" height="328" alt="Image" src="https://github.com/user-attachments/assets/9bf45320-c68f-4c00-a705-84965655d956" />

1. 지정된 포트에서 클라이언트 연결을 수신하고 Socket Connection을 얻는다.
2. Socket Connection으로부터 데이터 패킷을 수신한다.
3. 데이터 패킷을 파싱해서 ServletRequest 객체를 생성한다.
4. 얻어진 ServletRequest 객체를 알맞은 서블릿 컨테이너에게 보낸다.

## 특징
- **HTTP 요청 수신 및 응답 전송**
  - 특정 TCP 포트에서 서버로 들어오는 연결을 수신하고, 클라이언트의 요청을 Catalina에 전달한다.
  - Catalina에서 처리한 응답을 클라이언트에게 보낸다.
- **HTTP/1.1, HTTP/2, AJP 지원**
  - 기본적으로 HTTP/1.1을 지원한다. 설정을 통해 HTTP/2도 사용할 수 있다.
  - AJP(Apache JServ Protocol)를 사용하여 Apache HTTP 서버와 연동할 수 있다.
- **다양한 I/O 모델 지원**
  - BIO(Blocking I/O), NIO(Non-blocking I/O), NIO2, APR 등의 다양한 처리 모델을 지원한다.
  - Tomcat 7.x 및 이전 버전은 기본적으로 BIO 기반으로 동작하고, Tomcat 8.x 이후 버전은 기본적으로 NIO 기반으로 동작한다.
- **멀티스레드 기반으로 다중 요청 처리**
  - 스레드 풀(Thread Pool)을 활용해 각 요청을 별도 스레드에서 처리한다.
  - 다수 요청 동시 처리 가능, 효율적인 서버 성능 제공한다.
- **SSL/TLS 지원**
  - HTTPS 요청을 처리하기 위해 SSL/TLS 암호화를 지원한다.

## 설정
톰캣의 server.xml 파일에서 설정할 수 있다.
```xml
<Connector port="8080" protocol="HTTP/1.1"
           maxThreads="200"
           minSpareThreads="10"
           connectionTimeout="20000"
           redirectPort="8443" />
```

| 속성                  | 설명                                  |
|---------------------|-------------------------------------|
| `port`              | 연결을 대기할 TCP 포트 번호                   |
| `protocol`          | 사용할 프로토콜                            |
| `acceptCount`       | 연결 요청 대기 최대 개수                      |
| `connectionTimeout` | 연결을 수락한 이후 실제 요청이 들어올 때까지 대시 시간(ms) |
| `maxConnections`    | 동시 처리 가능한 최대 연결 수                   |
| `maxThreads`        | 서버에 생성할 수 있는 스레드 수                  |
| `minSpareThreads`   | 최소 유지할 대기 스레드 수                     |

---
**Reference**<br>
- https://velog.io/@jihoson94/BIO-NIO-Connector-in-Tomcat
- https://dev-ws.tistory.com/96
