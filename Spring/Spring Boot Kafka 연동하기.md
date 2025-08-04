# Spring Boot Kafka 연동하기

## 의존성
spring-kafka는 스프링 프레임워크에서 Kakfa를 쉽게 사용할 수 있도록 도와주는 라이브러리이다.
```gradle
implementation 'org.springframework.kafka:spring-kafka'
```

## Producer 설정
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


## Consumer 설정
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

## KafkaTemplate
> Producer - 메시지 전송
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

## @KafkaListener
> Consumer - 메시지 수신
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
