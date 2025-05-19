# Spring Cloud OpenFeign

## Feign이란
Feign은 선언형 HTTP 클라이언트이다. 인터페이스 정의를 통해 간단하게 HTTP Client 서비스를 작성할 수 있다.

## 의존성
Open Feign은 Spring Cloud 기반의 기술이므로 Spring Cloud에 대한 의존성이 필요하다.
[Spring Cloud 문서](https://spring.io/projects/spring-cloud)에서 Spring Boot 버전별로 적합한 Spring Cloud 버전을 확인할 수 있다.

예시는 Spring Boot 3.4.5를 기준으로 작성되었다.

<br>

Spring Cloud 의존성을 추가한다.
```gradle
ext {
    set('springCloudVersion', "2024.0.1")
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}
```
Open Feign 의존성을 추가한다.
```gradle
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
```

## 예제
### @EnableFeignClients
Feign 관련 컴포넌트 스캔을 위해 `@EnableFeignClients` 어노테이션을 추가한다.
```java
@SpringBootApplication
@EnableFeignClients
public class CustomerServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(CustomerServiceApplication.class, args);
    }
}
```

### 인터페이스 정의
```java
@FeignClient(name = "order-service")
public interface OrderClient {

    @GetMapping
    List<OrderResponse> getOrders(@RequestParam int customerId);
}

```
`@FeignClient`의 name에는 Eureka 서버에 등록된 어플리케이션 이름을 지정한다.

Eureka 서버에 연결되어 있다면 클라이언트의 url을 name을 기반으로 탐색한다.
이를 통해 별도로 url을 지정하지 않아도 Eureka 서버에 등록된 클라이언트로 요청을 보낼 수 있다.

Eureka 서버에 연결되지 않은 경우 url을 지정하여 요청을 보낼 수 있다.

### 사용
정의한 인터페이스를 주입받아서 사용한다.
```java
@RestController
public class CustomerController {
    private final OrderClient orderClient;

    public CustomerController(OrderClient orderClient) {
        this.orderClient = orderClient;
    }

    @GetMapping("/{id}/orders")
    public List<OrderResponse> getOrdersForCustomer(@PathVariable int id) {
        return orderClient.getOrdersByCustomerId(id);
    }
}
```

---
**Reference**<br>
- https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html
- https://code-lab1.tistory.com/414
