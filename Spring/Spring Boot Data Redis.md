# Spring Boot Data Redis

> Spring Boot 4.0.3 기준으로 작성되었다.
## 의존성
> build.gradle
```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-redis'
	testImplementation 'org.springframework.boot:spring-boot-starter-data-redis-test'
}
```

## application.properties
```yml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```
| 속성                                     | 설명                        |
|----------------------------------------|---------------------------|
| `spring.redis.host`                    | Redis 서버의 호스트 이름          |
| `spring.redis.port`                    | Redis 서버의 포트 번호           |
| `spring.redis.password`                | Redis 서버에 접근하기 위한 비밀번호    |
| `spring.redis.database`                | 사용할 Redis 데이터베이스의 인덱스     |
| `spring.redis.ssl`                     | SSL 연결 사용 여부              |
| `spring.redis.timeout`                 | Redis 서버와의 연결 시간 제한       |
| `spring.redis.lettuce.pool.max-active` | 동시에 유지할 수 있는 최대 연결 수      |
| `spring.redis.lettuce.pool.max-idle`   | 유휴 상태에서 유지할 수 있는 최대 연결 수  |
| `spring.redis.lettuce.pool.min-idle`   | 유휴 상태에서 유지할 수 있는 최소 연결 수  |
| `spring.redis.lettuce.pool.max-wait`   | 연결을 얻기 위해 기다리는 최대 시간     |

## Java 설정
```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }
}
```

## RedisTemplate
Spring Data Redis의 핵심 클래스로, Redis와 상호작용을 위한 메서드를 제공한다.

- **추상화**: Redis에 대한 저수준 상세 사항들을 추상화한다.
- **리소스 관리**: Redis 연결과 같은 리소스를 관리해준다.
- **직렬화/역직렬화**: Redis에 데이터를 저장하거나 가져올 때 직렬화와 역직렬화를 처리해준다.
- **예외 처리**: Redis 클라이언트(Lettuce, Jedis)에서 발생하는 예외를 스프링의 예외 계층으로 변환해준다.

### 주요 기능
`RedisTemplate`의 `opsFor...()` 메서드는 자료구조별 연산을 모아둔 `...Operations` 인터페이스를 반환한다.

| 메서드           | 인터페이스        | Redis 자료구조 | Operations 주요 메서드       |
|-----------------|-------------------|------------|-------------------------------|
| `opsForValue()` | `ValueOperations` | String     | `set`, `get`, `increment`     |
| `opsForList()`  | `ListOperations`  | List       | `leftPush`, `rightPop`        |
| `opsForSet()`   | `SetOperations`   | Set        | `add`, `intersect`, `members` |
| `opsForZSet()`  | `ZSetOperations`  | Sorted Set | `add`, `rangeByScore`         |
| `opsForHash()`  | `HashOperations`  | Hash       | `put`, `get`, `entries`       |

---
**Reference**
- https://adjh54.tistory.com/459
- https://techup.cocone.co.kr/2023/05/31/RedisTemplate.html
- https://docs.spring.io/spring-data/redis/reference/redis.html
