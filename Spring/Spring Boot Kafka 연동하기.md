# Spring Boot Kafka 연동하기

## 의존성
spring kafka는 스프링 프레임워크에서 Kakfa를 쉽게 사용할 수 있도록 도와주는 라이브러리이다.
```gradle
implementation 'org.springframework.kafka:spring-kafka'
```

## Topic
### KafkaAdmin
`KafkaAdmin`은 브로커에 토픽을 생성하는 역할을 한다.

Spring Boot를 사용하는 경우 `KafkaAdmin` 빈이 자동으로 등록되며, 이를 통해 브로커에 토픽을 생성할 수 있다.

### NewTopic
토픽을 생성하려면 `NewTopic` 빈을 정의해야 한다.

애플리케이션 시작 시, `KafkaAdmin`은 등록되어 있는 `NewTopic` 빈을 참고하여, 해당 토픽이 존재하지 않으면 자동으로 생성한다.
```java
@Configuration
public class KafkaTopicConfig {

    @Bean
    public NewTopic messageTopic() {
        return TopicBuilder.name("message-topic")
                .partitions(3)
                .replicas(2)
                .build();
    }
}
```

## Producer
메시지를 전송하려면 먼저 `ProducerFactory`를 설정해야 한다.
이를 통해 프로듀서 인스턴스의 생성 전략이 정의된다.

그다음으로 프로듀서 인스턴스를 래핑하는 `KafkaTemplate`을 설정해야 한다.
`KafkaTemplate`은 토픽에 메시지를 전송하기 위한 편리한 메서드를 제공한다.

설정은 application.yml 또는 Java Config 통해 할 수 있다.
### application.yml
```yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```

### Java Config
```java
@Configuration
public class KafkaProducerConfig {
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(props);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

### KafkaTemplate
`KafkaTemplate` 클래스를 사용하여 메시지를 전송할 수 있다.
```java
@Component
@RequiredArgsConstructor
public class KafkaMessageProducer {
    private static final String TOPIC_NAME = "message-topic";

    private final KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String message) {
        kafkaTemplate.send(TOPIC_NAME, message);
    }
}
```

## Consumer
메시지를 수신하려면 `ConsumerFactory` 와 `KafkaListenerContainerFactory`를 설정해야 한다.

`ConcurrentKafkaListenerContainerFactory`는 `KafkaListenerContainerFactory`의 구현체로 `@KafkaListener` 어노테이션이 달린 메서드에 대한 컨테이너를 만드는 데 사용된다.

설정은 application.yml 또는 Java Config 통해 할 수 있다.
### application.yml
```yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: default-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

### Java Config
```java
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "default-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

### @KafkaListener
`@KafkaListener` 애노테이션을 사용하여 메서드를 메시지 리스너로 지정할 수 있다.
```java
@Slf4j
@Component
@RequiredArgsConstructor
public class KafkaMessageConsumer {
    private static final String TOPIC_NAME = "message-topic";

    @KafkaListener(topics = TOPIC_NAME)
    public void listen(String message) {
        log.info("Consumed message: value={}", message);
    }
}
```

## spring kafka properties
지원되는 옵션은 [KafkaProperties.java](https://github.com/spring-projects/spring-boot/blob/3.5.x/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/kafka/KafkaProperties.java) 파일을 참조
```yml
spring:
  kafka:
    # 공통 Kafka 클라이언트 속성
    bootstrap-servers: # Kafka 클러스터에 연결하기 위한 host:port 목록 (쉼표로 구분)
    client-id: # 서버에 요청 시 전달되는 클라이언트 ID (서버 측 로그에 사용됨)
    properties: {} # 생산자와 소비자 공통으로 적용할 Kafka 클라이언트 속성
    ssl: {} # 공통 SSL 설정

    admin:
      client-id: # 어드민 클라이언트의 ID
      fail-fast: false # 브로커가 시작 시점에 사용 불가하면 즉시 실패할지 여부
      properties: {} # 어드민 클라이언트 구성용 추가 속성
      ssl: {} # 어드민 클라이언트용 SSL 설정

    producer:
      bootstrap-servers: # 프로듀서 클라이언트에 적용할 host:port 목록 (전역 속성 덮어쓰기)
      client-id: # 프로듀서 클라이언트 ID
      acks: # 요청 완료로 간주되기 위한 리더의 응답 수
      batch-size: # 기본 배치 크기
      buffer-memory: # 전송 대기 중인 레코드를 버퍼링할 수 있는 전체 메모리 크기
      compression-type: # 모든 데이터에 적용할 압축 타입
      retries: # 0보다 크면, 전송 실패 시 재시도 활성화
      key-serializer: # 키 직렬화 클래스
      value-serializer: # 값 직렬화 클래스
      transaction-id-prefix: # 값이 설정되면 트랜잭션 기능 활성화
      properties: {} # 추가 프로듀서 전용 속성
      ssl: {} # 프로듀서 클라이언트용 SSL 설정

    template:
      default-topic: # 메시지 전송 시 사용할 기본 토픽

    consumer:
      bootstrap-servers: # 컨슈머 클라이언트에 적용할 host:port 목록 (전역 속성 덮어쓰기)
      client-id: # 컨슈머 클라이언트 ID
      enable-auto-commit: # 오프셋을 백그라운드에서 주기적으로 커밋할지 여부
      auto-commit-interval: # enable-auto-commit이 true일 때, 오프셋 자동 커밋 주기
      auto-offset-reset: # 초기 오프셋이 없거나 유효하지 않을 때 동작 방식
      fetch-max-wait: # 최소 fetch 크기 조건이 충족되지 않았을 경우, 서버의 최대 대기 시간
      fetch-min-size: # fetch 요청에 대해 서버가 반환해야 하는 최소 데이터 양
      group-id: # 이 컨슈머가 속한 컨슈머 그룹의 고유 식별자
      heartbeat-interval: # Consumer Coordinator에 보내는 하트비트 간격
      key-deserializer: # 키 역직렬화 클래스
      value-deserializer: # 값 역직렬화 클래스
      max-poll-records: # poll() 호출당 반환할 최대 레코드 수
      properties: {} # 추가 컨슈머 전용 속성
      ssl: {} # 컨슈머 클라이언트용 SSL 설정

    listener:
      ack-count: # ackMode가 "COUNT" 또는 "COUNT_TIME"일 때 커밋 간 레코드 수
      ack-mode: # 리스너 AckMode (spring-kafka 문서 참고)
      ack-time: # ackMode가 "TIME" 또는 "COUNT_TIME"일 때 커밋 간 시간
      client-id: # 리스너 컨슈머 client.id의 접두사
      concurrency: # 리스너 컨테이너에서 실행할 스레드 수
      idle-event-interval: # 수신 데이터가 없을 때 idle 이벤트를 발행하는 간격
      log-container-config: # 초기화 시 컨테이너 설정을 INFO 레벨로 로그 출력할지 여부
      monitor-interval: # 응답 없는 소비자를 점검하는 주기 (단위 미지정 시 초)
      no-poll-threshold: # 컨슈머가 응답 없다고 판단하는 임계값 (pollTimeout의 배수)
      poll-timeout: # 컨슈머를 poll할 때 사용할 타임아웃
      type: # 리스너 유형

    jaas:
      enabled: false # JAAS 설정 활성화 여부
      control-flag: required # JAAS 로그인 구성의 control flag
      login-module: com.sun.security.auth.module.Krb5LoginModule # 로그인 모듈 클래스
      options: {} # 추가 JAAS 옵션

    streams:
      application-id: # 스트림의 application.id (기본값은 spring.application.name)
      auto-startup: true # 스트림 팩토리 빈 자동 시작 여부
      bootstrap-servers: # 스트림에 사용할 host:port 목록
      cache-max-size-buffering: # 전체 스레드에서 버퍼링에 사용할 최대 메모리 크기
      client-id: # 스트림 클라이언트 ID
      replication-factor: # 토픽의 복제 계수
      properties: {} # 스트림 구성용 추가 속성
      ssl: {} # 스트림용 SSL 설정
      state-dir: # 상태 저장소의 디렉토리 위치
```

---
**Reference**<br>
- https://adjh54.tistory.com/640
- https://adjh54.tistory.com/641
- https://devel-repository.tistory.com/46
- https://www.baeldung.com/spring-kafka
