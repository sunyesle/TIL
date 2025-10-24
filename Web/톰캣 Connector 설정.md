# 톰캣 Connector 설정

| 속성                  | 기본값                 | 설명                                |
|-----------------------|---------------------|-------------------------------------|
| **port**              |                     | 서버 소켓을 생성하고 연결을 대기할 TCP 포트 번호이다. |
| **protocol**          | `HTTP/1.1`          | 사용할 프로토콜을 설정한다. |
| **acceptCount**       | `100`               | maxConnections에 도달했을때 들어오는 연결 요청에 대해 운영 체제에서 제공하는 대기열의 최대 길이이다. 운영체제는 이 설정을 무시하고 다른 크기를 사용할 수 있다. |
| **maxConnections**    | `8192`              | 서버가 동시에 허용하고 처리할 수 있는 최대 연결 수이다. 최대 연결 수에 도달하더라도 운영 체제는 `acceptCount` 속성에 따라 연결을 계속 허용할 수 있다. |
| **connectionTimeout** | `60000(60초)`        | 연결을 수락한 후 요청 URI 줄이 표시될 때까지 기다리는 시간이다. -1로 설정하면 시간 초과가 발생하지 않는다. |
| **keepAliveTimeout**  |                     | 연결이 끊어지기 전에 다른 HTTP 요청을 기다리는 시간이다. 설정하지 않으면 connectionTimeout이 사용된다. -1로 설정하면 시간 초과가 발생하지 않는다. |
| **maxThreads**        | `200`               | Worker 스레드의 최대 개수이다. 가상 스레드가 활성화된 경우 적용되지 않는다 |
| **minSpareThreads**   | `10`                | Worker 스레드의 최소 개수이다. 여기에는 활성 스레드와 유휴 스레드가 모두 포함된다. 가상 스레드가 활성화된 경우 적용되지 않는다. |
| **maxQueueSize**      | `Integer.MAX_VALUE` | 스레드 풀 백업 큐의 최대 용량이다. |

---
**Reference**
- https://tomcat.apache.org/tomcat-9.0-doc/config/http.html
- https://docs.spring.io/spring-boot/appendix/application-properties/index.html
- https://px201226.github.io/tomcat/
