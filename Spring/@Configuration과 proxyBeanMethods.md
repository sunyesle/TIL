# @Configuration과 proxyBeanMethods

`@Configuration`은 `@Component`를 메타 어노테이션(Meta-annotation)으로 하는 합성 어노테이션(Composed-annotation)이다.
`@Component`와의 차이점은 `proxyBeanMethods` 옵션을 제공한다는 점이다.

## proxyBeanMethods 옵션
`proxyBeanMethods` 옵션은 `@Configuration` 클래스 내의 `@Bean` 메서드를 프록시로 감쌀지 여부를 지정하는 옵션으로, 기본값은 `true`이다.

`proxyBeanMethods = true`로 설정하면 `@Configuration` 클래스가 프록시 객체로 생성되며,
사용자 코드에서 `@Bean` 메서드를 직접 호출하는 경우에도 컨테이너에 등록된 싱글톤 빈 인스턴스를 반환한다.
이는 CGlib 라이브러리를 사용해 런타임에 `@Configuration` 클래스의 프록시 서브클래스를 생성하는 방식으로 구현된다.

## 옵션별 동작 차이
테스트 코드를 통해 `proxyBeanMethods` 옵션에 따른 동작 차이를 살펴보자.
```java
public class Common {
}

public class MyService {
    private final Common common;

    public MyService(Common common) {
        this.common = common;
    }

    public Common getCommon() {
        return common;
    }
}
```

### proxyBeanMethods = true
`proxyBeanMethods = true`인 경우부터 살펴보자.
다음 설정 클래스는 `@Configuration`에 옵션을 명시하지 않았으므로, 기본값인 `proxyBeanMethods = true`가 적용된다.
```java
@Configuration
public class ProxyConfig {

    @Bean
    public Common common(){
        System.out.println("Create common");
        return new Common();
    }

    // 파라미터를 통해 주입 받는다.
    @Bean
    public MyService injectedService(Common common){
        return new MyService(common);
    }

    // @Bean 메서드를 직접 호출한다.
    @Bean
    public MyService directCallService(){
        return new MyService(common());
    }
}
```

파라미터를 통해 주입 받은 `Common` 인스턴스와
`@Bean` 메서드를 직접 호출하여 얻은 `Common` 인스턴스 모두
컨테이너에 등록된 `Common` 빈임을 확인할 수 있다.
```java
@SpringBootTest(classes = ProxyConfig.class)
public class ProxyTest {

    @Autowired
    Common common;
    @Autowired
    MyService injectedService;
    @Autowired
    MyService directCallService;

    @Test
    void test() {
        Assertions.assertSame(common, injectedService.getCommon());
        Assertions.assertSame(common, directCallService.getCommon());
    }
}
```

### proxyBeanMethods = false
이번에는 `proxyBeanMethods` 옵션을 `false`로 지정하였다.
```java
@Configuration(proxyBeanMethods = false)
public class NonProxyConfig {

    @Bean
    public Common common(){
        System.out.println("Create common");
        return new Common();
    }

    // 파라미터를 통해 주입 받음.
    @Bean
    public MyService injectedService(Common common){
        return new MyService(common);
    }

    // @Bean 메서드를 직접 호출함.
    @Bean
    public MyService directCallService(){
        return new MyService(common());
    }
}
```

파라미터를 통해 주입 받은 `Common` 인스턴스는 컨테이너에 등록된 `Common` 빈이지만,
`@Bean` 메서드를 직접 호출하여 받은 `Common` 인스턴스는 새로 생성된 다른 인스턴스임을 확인할 수 있다.
```java
@SpringBootTest(classes = NonProxyConfig.class)
public class NonProxyTest {

    @Autowired
    Common common;
    @Autowired
    MyService injectedService;
    @Autowired
    MyService directCallService;

    @Test
    void test() {
        Assertions.assertSame(common, injectedService.getCommon());
        Assertions.assertNotSame(common, directCallService.getCommon()); // 메서드 직접 호출 시 새로운 인스턴스를 생성한다.
    }
}
```

### 요약
| 상황                      | `proxyBeanMethods = true` | `proxyBeanMethods = false` |
| ----------------------- | ----------------------- | ------------------------ |
| `@Bean` 메서드 직접 호출      | 컨테이너 빈 반환     | 새 인스턴스 생성          |
| 의존성 주입 (파라미터 주입 등) | 컨테이너 빈 주입     | 컨테이너 빈 주입          |

---
**Reference**
- https://www.danvega.dev/blog/spring-proxy-bean-methods
