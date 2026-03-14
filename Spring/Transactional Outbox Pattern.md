# Transactional Outbox Pattern

서비스에서 DB 상태를 변경함과 동시에 메시지를 발행해야 하는 경우가 있다.

이때, 간단한 구현 방법은 하나의 트랜잭션 안에서 DB 저장과 메시지 발행을 함께 처리하는 것이다.
```java
@Transactional 
public Order createOrder(OrderRequest orderRequest) { 
    Order order = createOrder(orderRequest); // 비즈니스 로직

    order = orderRepository.save(order); // DB에 주문 정보 저장

    // 메시지 발행
    OrderCreatedEvent event = new OrderCreatedEvent(order);
    kafkaTemplate.send("order-topic", event);

    return order;
}
```

위 코드는 겉보기에는 문제가 없어보이지만 다음과 같은 문제가 발생할 수 있다.
1. **트랜잭션이 롤백되는 경우**: `kafkaTemplate.send()`가 호출되는 시점은 DB 트랜잭션이 커밋되기 전이다. 만약 오류가 발생해 트랜잭션이 롤백되어도 발행된 메시지는 취소가 불가능하다.
2. **Kafka 브로커에 일시적인 장애가 발생하는 경우**: 물론 트랜잭션을 롤백할 수는 있지만 외부 시스템의 일시적인 오류로 인해 전체 트랜잭션을 모두 롤백하는 것이 좋은 방법일까?

## Transactional Outbox Pattern
DB 트랜잭션은 DB 차원에서 원자성을 보장하므로 트랜잭션에 포함된 쿼리들은 원자적으로 실행되지만, Message Broker는 다른 Data Source이므로 원자적인 처리가 불가능하다.

이러한 문제를 해결하기 위해 사용하는 방법 중 하나가 **Transactional Outbox Pattern** 이다.

핵심 아이디어는 메시지 발행 정보를 비즈니스 데이터와 동일한 DB에 저장하여 원자성을 확보하는 것이다.

DB에 발행할 메시지 정보를 저장할 `OUTBOX` 테이블을 생성하고, 메시지를 발행하는 대신 이 테이블에 데이터를 저장한다.
이후 스케줄러 등 별도의 프로세스로 `OUTBOX` 테이블에 저장된 정보를 읽어 메시지를 발행한다.

## 구현 예시
메시지 정보를 저장할 `Outbox` 엔티티를 정의한다.
```java
@Entity
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class Outbox {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String eventType;

    private String payload;

    @Enumerated(EnumType.STRING)
    private OutboxStatus status;

    private LocalDateTime createdAt;
}
```

서비스에서 직접 메시지를 발행하는 대신 `Outbox` 테이블에 발행할 메시지 정보를 저장한다.
```java
@Service
public class OrderService {

    @Transactional 
    public Order createOrder(OrderRequest orderRequest) { 
        Order order = createOrder(orderRequest); // 비즈니스 로직

        order = orderRepository.save(order); // DB에 주문 정보 저장

        // DB에 이벤트 저장
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        outboxRepository.save(new Outbox("order-topic", objectMapper.writeValueAsString(event)));

        return order;
    }
}
```

스케줄러를 통해 주기적으로 `Outbox` 테이블의 데이터를 읽어 메시지를 발행한다. 
```java
@Component
public class OutboxScheduler {

    @Scheduled(fixedDelay = 10000)
    public void processOutbox() {
        outboxProcessor.processAll();
    }
}
```

`OutboxProcessor`에서 메시지를 발행하고, 처리 완료된 건을 구분하기 위해 `Outbox`의 상태 값을 업데이트한다.
```java
@Component
public class OutboxProcessor {

    public void processAll() {
        outboxService.getUnprocessedOutboxes().forEach(this::processInternal);
    }

    private void processInternal(Outbox outbox) {
        try {
            // 메시지 발행
            kafkaTemplate.send(outbox.getEventType(), outbox.getPayload())
                    .get(5, TimeUnit.SECONDS);

            // 발행 완료 처리
            outboxService.markCompleted(outbox.getId());
        } catch (Exception e) {
            log.error("Outbox {} failed: {}", outbox.getId(), e.getMessage());
        }
    }
}
```

특정 메시지 처리중에 예외가 발생하더라도, 처리 완료된 다른 메시지의 상태까지 롤백되는 것을 방지하기 위해 `REQUIRES_NEW` 전파 속성을 사용하여 트랜잭션을 분리했다.
```java
@Service
public class OutboxService {

    // 메시지마다 별도의 트랜잭션으로 진행 
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void markCompleted(Long id) {
        Outbox outbox = outboxRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Outbox not found"));
        outbox.markCompleted();
    }
}
```

## Application Event를 이용한 Outbox 저장/발행 로직 분리
Application Event를 활용하여, 비즈니스 로직과 `Outbox` 저장 로직을 분리하고 이벤트가 즉시 발행될 수 있도록 개선했다.

이벤트 객체를 정의한다.
```java
public class OutboxEvent {
    private final String eventType;
    private final Object data;
    private Long outboxId;

    OutboxEvent(String eventType, Object data) {
        this.eventType = eventType;
        this.data = data;
    }

    public void setOutboxId(Long outboxId) {
        this.outboxId = outboxId;
    }
    public Long getOutboxId() {
        return outboxId;
    }
}
```

서비스에서 직접 `Outbox` 테이블에 데이터 저장하는 대신, 애플리케이션 이벤트를 발행한다.
```java
@Service
public class OrderService {

    private final ApplicationEventPublisher publisher;

    @Transactional 
    public Order createOrder(OrderRequest orderRequest) { 
        Order order = createOrder(orderRequest); // 비즈니스 로직

        order = orderRepository.save(order); // 데이터베이스에 주문 정보 저장

        // ApplicationEvent 발행
        OrderCreatedEvent data = new OrderCreatedEvent(order);
        publisher.publishEvent(new OutboxEvent("order-placed", data));

        return order;
    }
}
```

이벤트 리스너 로직은 다음과 같다.
- `BEFORE_COMMIT`: 동일한 트랜잭션에서 `Outbox` 테이블에 데이터를 저장하고, 이벤트 객체에 `OutboxId` 값을 설정한다.
- `AFTER_COMMIT`: 커밋 완료 후 즉시 `OutboxId`에 해당하는 메시지를 발행한다.
```java
@Component
public class OutboxEventListener {

    @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
    public void saveOutbox(OutboxEvent event) {
        Outbox outbox = outboxRepository.save(new Outbox(event.eventType(), objectMapper.writeValueAsString(event.payload())));
        
        // 생성된 ID를 이벤트 객체에 보관 (AFTER_COMMIT에서 쓰기 위함)
        event.setOutboxId(outbox.getId());
    }

    @Async(AsyncConfig.EVENT_ASYNC_TASK_EXECUTOR) // 비동기로 진행
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void processOutbox(OutboxEvent event) {
        // 메시지 발행
        outboxProcessor.process(outbox.getId());
    }
}
```

스케줄러는 즉시 발행 실패건을 처리하기 위한 안전장치로 사용하며, 변경사항은 다음과 같다.
- 스케줄러 실행 주기를 1분으로 늘림
- 최근 30초 이내에 저장된 outbox는 이벤트 리스너에서 처리 중일 수 있으므로 스케줄러 대상에서 제외

---
**Reference**
- https://helloworld.kurly.com/blog/2026-outbox-pattern-and-retry-topic/
- https://ridicorp.com/story/transactional-outbox-pattern-ridi/
