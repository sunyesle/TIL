# Spring Boot 4 OpenTelemetry
> 이 글은 Spring Boot 4.0.1 기준으로 작성되었다.

Spring Boot 4에 `spring-boot-starter-opentelemetry` 스타터가 추가되었다.

단일 스타터 추가만으로 **OTLP**(OpenTelemetry Protocol) 기반으로 로그, 메트릭, 트레이스를 내보내기 위한 위한 환경을 구축할 수 있다.

**주요 자동화 기능**
- **Traces**: OpenTelemetry SDK가 자동 구성되어 요청 흐름을 추적한다.
- **Metrics**: Micrometer로 수집한 메트릭을 OTLP 형식으로 자동 변환하여 전송한다.
- **Logs**: 애플리케이션 로그에 `traceId`와 `spanId`를 자동으로 주입하여 트레이스와 로그를 연결한다.

## 의존성
> build.gradle
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-opentelemetry'
    testImplementation 'org.springframework.boot:spring-boot-starter-opentelemetry-test'
}
```

## 인프라 구축 (Docker Compose)
`grafana/otel-lgtm`는 개발 및 테스트를 위한 이미지로 복잡한 설정 없이 Loki, Grafana, Tempo, Prometheus를 띄울 수 있다.
- **L**: Loki(로그 저장소)
- **G**: Grafana(데이터 시각화 도구)
- **T**: Tempo(트레이스 저장소)
- **M**: Prometheus(메트릭 저장소)

> docker-compose.yml
```yml
services:
  grafana-lgtm:
    image: 'grafana/otel-lgtm:latest'
    ports:
      - '3000:3000' # Grafana UI
      - '4317:4317' # OTLP gRPC 수신
      - '4318:4318' # OTLP HTTP 수신
```

## 메트릭 내보내기
스타터에 포함되어 있는 `io.micrometer:micrometer-registry-otlp` 종속성을 통해, 수집된 메트릭을 OTLP 형식으로 백엔드에 내보낸다.

별도 설정 없이도 메트릭 내보내기가 활성화되며, 엔드포인트 기본값은 `http://localhost:4318/v1/metrics`이다.
이 값은 다음 속성을 통해 변경할 수 있다.
```properties
management.otlp.metrics.export.url=http://localhost:4318/v1/metrics
```

### Semantic Convention 설정
Micrometer는 데이터의 일관성을 위해 **OpenTelemetry의 Semantic Convention**(표준 명명 규칙)을 구현하고 있다.
현재는 해당 규칙을 적용하기 위해 별도의 설정코드를 작성해야 하며, 이 부분은 향후 개선될 예정이다.
```java
@Configuration(proxyBeanMethods = false)
public class OpenTelemetryConfig {

    @Bean
    OpenTelemetryServerRequestObservationConvention openTelemetryServerRequestObservationConvention() {
        return new OpenTelemetryServerRequestObservationConvention();
    }

    @Bean
    OpenTelemetryJvmCpuMeterConventions openTelemetryJvmCpuMeterConventions() {
        return new OpenTelemetryJvmCpuMeterConventions(Tags.empty());
    }

    @Bean
    ProcessorMetrics processorMetrics() {
        return new ProcessorMetrics(List.of(), new OpenTelemetryJvmCpuMeterConventions(Tags.empty()));
    }

    @Bean
    JvmMemoryMetrics jvmMemoryMetrics() {
        return new JvmMemoryMetrics(List.of(), new OpenTelemetryJvmMemoryMeterConventions(Tags.empty()));
    }

    @Bean
    JvmThreadMetrics jvmThreadMetrics() {
        return new JvmThreadMetrics(List.of(), new OpenTelemetryJvmThreadMeterConventions(Tags.empty()));
    }

    @Bean
    ClassLoaderMetrics classLoaderMetrics() {
        return new ClassLoaderMetrics(new OpenTelemetryJvmClassLoadingMeterConventions());
    }
}
```

## 트레이스 내보내기
스타터에 포함되어 있는 `io.micrometer:micrometer-tracing-bridge-otel` 종속성을 통해, 생성된 트레이스가 OpenTelemetry API에 맞게 변환된다.

트레이스를 내보내려면 다음 속성을 설정해야 한다.
```properties
management.opentelemetry.tracing.export.otlp.endpoint=http://localhost:4318/v1/traces
```

## 로그 내보내기
Spring Boot는 OTLP 형식으로 로그를 내보낼 수 있도록 하는 자동 구성을 제공한다.
하지만 Logback이나 Log4j2에 어펜더를 자동으로 설치하지는 않으므로, 기본 설정만으로는 로그가 내보내지지 않는다.

로그를 내보내려면 다음 두 가지 작업이 필요하다.

### 속성 설정
```properties
management.opentelemetry.logging.export.otlp.endpoint=http://localhost:4318/v1/logs
```

### 어펜더 설정
Logback을 사용하려면 우선 `io.opentelemetry.instrumentation:opentelemetry-logback-appender-1.0` 의존성을 추가해야 한다.
```gradle
dependencies {
    runtimeOnly 'io.opentelemetry.instrumentation:opentelemetry-logback-appender-1.0:2.23.0-alpha'
}
```

다음으로 사용자 지정 Logback 설정을 생성해야 한다. `src/main/resources/logback-spring.xml` 경로에 파일을 생성한다.

아래 설정은 Spring Boot에서 Logback의 기본 설정을 가져온 다음, 모든 로그 이벤트를 OpenTelemetry API로 전송하는 `OTEL`이라는 추가 어펜더를 정의한다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <appender name="OTEL" class="io.opentelemetry.instrumentation.logback.appender.v1_0.OpenTelemetryAppender">
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="OTEL"/>
    </root>
</configuration>
```

마지막으로 `OpenTelemetryAppender`에게 사용할 OpenTelemetry API 인스턴스를 알려주기 위한 빈을 생성한다.
```java
@Component
class InstallOpenTelemetryAppender implements InitializingBean {
    private final OpenTelemetry openTelemetry;

    InstallOpenTelemetryAppender(OpenTelemetry openTelemetry) {
        this.openTelemetry = openTelemetry;
    }

    @Override
    public void afterPropertiesSet() {
        OpenTelemetryAppender.install(this.openTelemetry);
    }
}
```

## Micrometer Observation 사용하기
`@Observed`와 같은 observability 어노테이션 스캔을 활성화하려면 다음 작업이 필요하다.

### 속성 설정
```properties
management.observations.annotations.enabled=true
```

### AOP 의존성 추가
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-aspectj'
    testImplementation 'org.springframework.boot:spring-boot-starter-aspectj-test'
}
```

다음과 같이 어노테이션을 사용할 수 있다.
```java
@Observed(name = "user.find-with-id")
@Transactional(readOnly = true)
public User findWithId(@ObservationKeyValue("user.id") long id) {
    return userRepository.findById(id).orElse(null);
}
```

## Trace ID 헤더에 추가하기
Micrometer Tracing이 클래스패스에 있으면, Spring Boot는 로그 메시지에 trace ID와 span ID를 포함하도록 로그 패턴을 조정한다. 이는 특정 트레이스와 관련된 로그를 찾을 때 유용하다.

서버가 응답을 보낼 때 HTTP 헤더에 Trace ID를 포함시키면 장애 원인 파악과 대응 시간을 단축시킬 수 있다.
사용자가 에러를 겪었을 때 Trace ID를 알려주는 것만으로도, 해당 요청에 대한 모든 기록을 추적하여 원인을 파악할 수 있다.

다음 서블릿 필터를 사용하여 Trace ID를 응답 헤더에 추가할 수 있다.
```java
@Component
class TraceIdFilter extends OncePerRequestFilter {
    private final Tracer tracer;

    TraceIdFilter(Tracer tracer) {
        this.tracer = tracer;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String traceId = getTraceId();
        if (traceId != null) {
            response.setHeader("X-Trace-Id", traceId);
        }
        filterChain.doFilter(request, response);
    }

    private @Nullable String getTraceId() {
        TraceContext context = this.tracer.currentTraceContext().context();
        return context != null ? context.traceId() : null;
    }
}
```

## 비동기 작업 시 컨텍스트가 유실되는 문제 해결
`@Async`나 `AsyncTaskExecutor`를 사용하여 새로운 스레드에서 작업을 수행하면, 기존 스레드가 가지고 있던 Trace ID나 Span ID 같은 컨텍스트 정보가 유실된다.

트레이스 정보는 `ThreadLocal`에 보관되는데 이 `ThreadLocal`이 새 스레드로 전달되지 않기 때문에 발생하는 문제다.

`ContextPropagatingTaskDecorator` 빈을 등록하는 것만으로 해결할 수 있다.
이는 Micrometer의 Context Propagation API를 사용하여, 현재 스레드의 컨텍스트를 비동기 스레드로 복제해 주는 역할을 한다.
```java
@Configuration(proxyBeanMethods = false)
public class ContextPropagationConfiguration {

    @Bean
    ContextPropagatingTaskDecorator contextPropagatingTaskDecorator() {
        return new ContextPropagatingTaskDecorator();
    }
}
```

## HTTP 클라이언트 사용 시 주의사항
`RestTemplate`, `RestClient`, `WebClient`를 직접 `new` 키워드로 생성하면 안된다.

대신 `RestTemplateBuilder`, `RestClient.Builder`, `WebClient.Builder` 주입받고 이를 사용해 클라이언트를 생성해야 한다.

스프링 부트가 미리 구성해 둔 빌더는 요청 헤더에 트레이스 컨텍스트가 자동으로 포함되도록 설정되어 있다.
클라이언트를 직접 생성하면 이러한 설정이 적용되지 않아 컨텍스트가 전파되지 않는다.

---
**Reference**
- https://spring.io/blog/2025/11/18/opentelemetry-with-spring-boot
