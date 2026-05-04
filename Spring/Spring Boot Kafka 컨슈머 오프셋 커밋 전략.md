# Spring Boot Kafka 컨슈머 오프셋 커밋 전략

Kafka는 컨슈머가 메시지를 어디까지 읽었는지 추적하기 위해 오프셋(offset)을 저장한다. 컨슈머는 커밋을 통해 오프셋 값을 갱신하며, 이후 이 값을 참조하여 커밋된 위치 이후부터 메시지를 읽어온다.

컨슈머의 커밋 전략은 `spring.kafka.consumer.enable-auto-commit` 설정으로 지정할 수 있으며, 기본값은 자동 커밋(`true`)이다.

## 오프셋 자동 커밋
```yml
spring:
  kafka:
    consumer:
      enable-auto-commit: true # 컨슈머 자동 커밋 활성화
      auto-commit-interval: 5000ms # 5초 간격으로 커밋을 진행
      max-poll-records: 100 # poll() 호출당 반환할 최대 레코드 수
```
`poll()` 메서드가 호출될 때, 마지막 커밋 이후 `auto.commit.interval.ms`가 지났으면 오프셋을 커밋한다.

### 주의점
자동 커밋 사용 시 다음과 같은 상황에서 메시지 유실 또는 중복이 발생할 수 있다.

**1. 메시지 유실**<br>
메시지의 처리 시간이 5초보다 긴 경우, 메시지 처리가 완료되기 전에 오프셋 커밋이 일어난다. 이후 메시지 처리에서 오류가 발생하는 경우 해당 메시지는 다시 처리할 수 없게 된다.

**2. 메시지 중복**<br>
`poll()` 메서드를 호출하여 100개의 레코드를 가져온다. 이 중 30개의 레코드를 처리한 상태에서 커밋이 이루어지기 전에 리밸런싱이 발생하여 컨슈머들이 재할당되는 경우, 마지막 커밋으로 부터 다시 데이터를 polling하여 이미 처리한 30개의 레코드를 다시 중복으로 처리하게 된다.

이러한 메시지 유실/중복을 방지하기 위해서는 **수동 커밋**을 사용해야 한다.

## 오프셋 수동 커밋
```yml
spring:
  kafka:
    consumer:
      enable-auto-commit: false
    listener:
      ack-mode: MANUAL_IMMEDIATE
```
카프카 클라이언트의 자동 커밋 기능을 끄고, **스프링 리스너 컨테이너**가 커밋 타이밍을 관리한다. 리스너의 `AckMode` 설정에 따라 커밋 시점이 달라진다.

### AckMode
| AckMode | 설명 |
|---|---|
| `BATCH` | `poll()` 메서드로 호출된 레코드가 모두 처리된 이후 커밋한다. (default) |
| `RECORD` | 레코드 단위로 프로세싱 이후 커밋한다. |
| `TIME` | 특정 시간 이후에 커밋한다.(`AckTime` 설정 필요) |
| `COUNT` | 특정 개수만큼 레코드가 처리된 이후에 커밋한다. (`AckCount` 설정 필요) |
| `COUNT_TIME` | TIME, COUNT 옵션 중 하나라도 충족되면 커밋한다. |
| `MANUAL` | `Acknowledgement.acknowledge()` 메서드가 호출되면 다음번 `poll()`때 커밋한다. |
| `MANUAL_IMMEDIATE` | `Acknowledgement.acknowledge()` 메서드를 호출한 즉시 커밋한다. |

**비즈니스 로직 중간에 명시적으로 커밋 시점을 제어**해야 하는 경우 `MANUAL` 또는 `MANUAL_IMMEDIATE`를 사용해야 한다.

이 방식을 사용하면 리스너 파라미터로 `Acknowledgment` 객체를 받을 수 있으며, `acknowledge()` 메서드를 호출하여 직접 오프셋을 커밋할 수 있다.
```java
@KafkaListener(topics = "test")
public void listen(ConsumerRecord<?, ?> record, Acknowledgment ack) {
    System.out.println("Received message: " + record.value());
    // 비즈니스 로직...
    ack.acknowledge(); // 로직이 성공했을 때만 명시적으로 커밋
}
```

---
**Reference**
- https://pula39.tistory.com/19
- https://dkswnkk.tistory.com/744
