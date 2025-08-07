# Spring Boot Kafka 연동하기

## 의존성
spring-kafka는 스프링 프레임워크에서 Kakfa를 쉽게 사용할 수 있도록 도와주는 라이브러리이다.
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

---
**Reference**<br>
- https://adjh54.tistory.com/640
- https://adjh54.tistory.com/641
- https://devel-repository.tistory.com/46
- https://www.baeldung.com/spring-kafka
