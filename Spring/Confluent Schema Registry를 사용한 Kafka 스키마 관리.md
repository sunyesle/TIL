# Confluent Schema Registry를 사용한 Kafka 스키마 관리

## Schema Registry
Schema Registry는 Kafka와 별도로 동작하는 프로세스로, 데이터 형식(스키마)을 중앙에서 저장하고 관리하는 서비스이다.

프로듀서와 컨슈머 모두 Schema Registry에 등록된 스키마를 사용하면 데이터 형식이 변경되더라도 일관성과 호환성을 유지할 수 있다.

## Schema Registry Flow
<img width="921" height="452" alt="Image" src="https://github.com/user-attachments/assets/22864421-0a6b-4c10-bc4d-406311c0d990" />

1. 메시지가 생성된다. 프로듀서의 `KafkaAvroSerializer`는 메시지 스키마와 연관된 스키마 id를 가져온다.
2. 메시지가 Avro 형식으로 직렬화되고, 검색된 스키마를 사용하여 검증된다.
3. 메시지가 Kafka 토픽에 기록된다.
4. 메시지가 토픽에서 Kafka 컨슈머에 의해 소비된다.
5. 컨슈머의 `KafkaAvroDeserializer`는 메시지에서 스키마 ID를 가져오고, 이를 사용하여 스키마 레지스트리에서 스키마를 조회한다.
6. 메시지가 역직렬화 되고, 검색된 스키마를 사용하여 검증된다.

### Producer 직렬화 과정
<img width="1116" height="862" alt="Image" src="https://github.com/user-attachments/assets/73264e9d-bed3-43d3-ab28-8703b17f249d" />

### Consumer 역직렬화 과정
<img width="1273" height="862" alt="Image" src="https://github.com/user-attachments/assets/6926981c-ce68-45c8-83a7-a8f935bc5e7c" />

## Schema 호환성 유형
예를 들어 v1, v2 스키마가 있다고 하자.

### BACKWARD
> v1 ← (read) v2

새로운 스키마를 사용하는 컨슈머가 이전 스키마로 생성된 데이터를 읽을 수 있음을 의미한다.<br>
즉, v2로 생성된 데이터는 v1을 사용하는 컨슈머가 처리할 수 있다.

컨슈머를 변경 없이 유지하고 프로듀서만 업데이트하는 경우에 유용하다.

### FORWARD
> v1 (read) → v2

새로운 스키마로 생성된 데이터를 이전 스키마를 사용하는 컨슈머가 읽을 수 있음을 의미한다.<br>
즉, v2로 생성된 데이터는 v1을 사용하는 컨슈머가 읽을 수 있다.

프로듀서는 그대로 두고 컨슈머를 업데이트하는 경우에 유용하다.

### TRANSITIVE
`*_TRANSITIVE`

전이적 호환성을 설정하면, 새로 등록하려는 스키마가 이전에 등록된 모든 스키마와 호환되는지 검증한다.<br>
전이적이 아닌 경우에는 직전 버전과의 호환성만 확인한다.

| Compatibility Type  | Changes allowed                             | Check against which schemas | Upgrade first  |
|---------------------|---------------------------------------------|-----------------------------|----------------|
| `BACKWARD` (기본값)   | fields 삭제 가능<br>optional fields 추가 가능      | 직전 버전                       | Consumers      |
| `BACKWARD_TRANSITIVE` | fields 삭제 가능<br>optional fields 추가 가능          | 모든 이전 버전                    | Consumers      |
| `FORWARD`             | fields 추가 가능<br>optional fields 삭제 가능          | 직전 버전                       | Producers      |
| `FORWARD_TRANSITIVE`  | fields 추가 가능<br>optional fields 삭제 가능          | 모든 이전 버전                    | Producers      |
| `FULL`                | optional fields 추가 가능<br>optional fields 삭제 가능 | 직전 버전                       | 순서 상관 없음 |
| `FULL_TRANSITIVE`     | optional fields 추가 가능<br>optional fields 삭제 가능 | 모든 이전 버전                    | 순서 상관 없음 |
| `NONE`                | 모든 변경 사항이 가능함                               | 호환성 검사 비활성화                 | Depends        |

---
**Reference**<br>
- https://www.lydtechconsulting.com/blog-kafka-schema-registry-intro.html
- https://medium.com/@gaemi/kafka-와-confluent-schema-registry-를-사용한-스키마-관리-1-cdf8c99d2c5c
- https://docs.confluent.io/cloud/current/sr/fundamentals/schema-evolution.html
