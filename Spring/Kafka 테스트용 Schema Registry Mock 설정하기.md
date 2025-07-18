# Kafka 테스트용 Schema Registry Mock 설정하기

`schema.registry.url` 설정 값에 `mock://` 프로토콜을 사용하면,<br>
`KafkaAvroSerializer`나 `KafkaAvroDeserializer`가 내부적으로 `MockSchemaRegistryClient`를 사용한다.

## 설정 예시
```yml
spring:
  kafka:
    properties:
      schema.registry.url: mock://my-scope
```

## 관련 코드
> **Github 링크**<br>
> [MockSchemaRegistry.java](https://github.com/confluentinc/schema-registry/blob/52a22fd26876bf24a79d952749f45444191cbb5e/client/src/main/java/io/confluent/kafka/schemaregistry/testutil/MockSchemaRegistry.java)<br>
> [AbstractKafkaSchemaSerDe.java](https://github.com/confluentinc/schema-registry/blob/9c2327084763e85f3617cef5260bd5f9add22474/schema-serializer/src/main/java/io/confluent/kafka/serializers/AbstractKafkaSchemaSerDe.java#L75)

```java
/**
 * A repository for mocked Schema Registry clients, to aid in testing.
 *
 * <p>Logically independent "instances" of mocked Schema Registry are created or retrieved
 * via named scopes {@link MockSchemaRegistry#getClientForScope(String)}.
 * Each named scope is an independent registry.
 * Each named-scope registry is statically defined and visible to the entire JVM.
 * Scopes can be cleaned up when no longer needed via {@link MockSchemaRegistry#dropScope(String)}.
 * Reusing a scope name after cleanup results in a completely new mocked Schema Registry instance.
 *
 * <p>This registry can be used to manage scoped clients directly, but scopes can also be registered
 * and used as {@code schema.registry.url} with the special pseudo-protocol 'mock://'
 * in serde configurations, so that testing code doesn't have to run an actual instance of
 * Schema Registry listening on a local port. For example,
 * {@code schema.registry.url: 'mock://my-scope-name'} corresponds to
 * {@code MockSchemaRegistry.getClientForScope("my-scope-name")}.
 */
public final class MockSchemaRegistry {
  private static final String MOCK_URL_PREFIX = "mock://";
  private static final Map<String, SchemaRegistryClient> SCOPED_CLIENTS = new HashMap<>();

  ...
}
```

---
**Reference**<br>
- https://stackoverflow.com/questions/39670691/how-to-have-kafkaproducer-to-use-a-mock-schema-registry-for-testing
- https://aerain.github.io/spring/2021/02/02/kafka-testcontainer.html
