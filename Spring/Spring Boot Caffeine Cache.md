# Spring Boot Caffeine Cache

## 캐시
데이터를 미리 복사해 놓은 임시 장소를 가리킨다.
접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다.

### 캐시를 사용하기 적절한 데이터
- 자주 조회되는 데이터
- 데이터의 연산에 드는 비용이 비싼 데이터
- 업데이트가 자주 발생하지 않는 데이터
- 실시간으로 정확성을 요구하지 않는 데이터
- 데이터의 변경이 전파되지 않는 데이터

## 스프링 캐시(Spring Cache)
스프링은 캐시 추상화를 제공한다.
스프링 AOP 기반으로 캐시가 동작하며, 어노테이션으로 간편하게 캐싱을 적용할 수 있다.

### 의존성 추가
```gradle
implementation 'org.springframework.boot:spring-boot-starter-cache'
implementation 'com.github.ben-manes.caffeine:caffeine'
```

### 캐싱 기능 활성화
`@EnableCaching` 어노테이션을 추가해 캐싱 기능을 활성화한다.
```java
@EnableCaching
@Configuration
public class CacheConfig {

}
```

### CacheType 작성
캐시 항목별 세부 설정(만료 시간, 최대 크기)을 관리하는 enum이다.
```java
@Getter
public enum CacheType {
    PRODUCTS("products", 5 * 60, 10000);

    CacheType(String cacheName, int expiredAfterWrite, int maximumSize) {
        this.cacheName = cacheName;
        this.expiredAfterWrite = expiredAfterWrite;
        this.maximumSize = maximumSize;
    }

    private final String cacheName;
    private final int expiredAfterWrite;
    private final int maximumSize;
}
```

### CacheManager 빈 등록
위에서 정의한 `CacheType`을 이용해 `CacheManager`를 생성한다.
```java
@EnableCaching
@Configuration
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        List<CaffeineCache> caches = Arrays.stream(CacheType.values())
                .map(cache -> new CaffeineCache(cache.getCacheName(), Caffeine.newBuilder()
                                .expireAfterWrite(cache.getExpiredAfterWrite(), TimeUnit.SECONDS)
                                .maximumSize(cache.getMaximumSize())
                                .build()
                        )
                )
                .toList();
        cacheManager.setCaches(caches);
        return cacheManager;
    }
}
```

### 어노테이션으로 캐싱 사용하기
#### @Cacheable - 캐시 저장/조회
캐시에 데이터가 없을 경우에는 메서드를 실행한 후 캐시에 데이터를 저장하고, 캐시에 데이터가 있으면 캐시의 데이터를 반환한다.<br>
```java
@Cacheable(value = "products", key = "#id")
public ProductResponse getProduct(Long id) {
    log.info("getProduct 호출");
    return productRepository.findById(id).map(ProductResponse::of)
            .orElseThrow(NoSuchElementException::new);
}
```

> SpEL을 사용하여 key, conditional 속성 값을 지정할 수 있다.
자세한 내용은 [공식 문서](https://docs.spring.io/spring-framework/reference/integration/cache/annotations.html#cache-spel-context)에서 확인 가능하다.

| 이름            | 설명                | 예                       |
|---------------|-------------------|-------------------------|
| Argument name | 특정 메서드 인수의 이름     | `#id` 또는 `#a0` 또는 `#p0` |
| result        | 메서드 호출의 결과(캐시할 값) | `#result`               |

#### @CachePut - 캐시 저장
항상 메서드를 실행하고 캐시를 저장한다.
```java
@CachePut(value = "products", key = "#result.id")
public ProductResponse saveProduct(ProductRequest request) {
    log.info("saveProduct 호출");
    Product product = request.toEntity();
    productRepository.save(product);
    return ProductResponse.of(product);
}
```

#### @CacheEvict - 캐시 제거
캐시를 제거한다.
```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(Long id) {
    log.info("deleteProduct 실행");
    productRepository.deleteById(id);
}
```

---
**Reference**<br>
- https://docs.spring.io/spring-framework/reference/integration/cache/annotations.html
- https://velog.io/@ktf1686/Spring-Spring-Cache%EC%97%90-%EB%8C%80%ED%95%B4
- https://jiwondev.tistory.com/282
- https://mangkyu.tistory.com/179
- https://devel-repository.tistory.com/89
- https://velog.io/@itonse/%EA%B5%AC%ED%95%B4%EC%9C%A0-Caffeine-Cache-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%A0%81%EC%9A%A9
- https://blog.yevgnenll.me/posts/spring-boot-with-caffeine-cache
