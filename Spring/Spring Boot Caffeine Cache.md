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

지원되는 캐시 구현체는 Ehcache, Caffeine, Cache2k, Redis 등이 있으며,
해당 캐시 구현체들은 Auto-configuration을 통해 설정 파일(yml) 기반으로 설정 가능하다.

예제에서는 캐시 구현체로 **Caffeine**을 사용한다.

## 의존성 추가
```gradle
implementation 'org.springframework.boot:spring-boot-starter-cache'
implementation 'com.github.ben-manes.caffeine:caffeine'
```

## CacheManager 설정
### 1. application.yml
Auto-configuration을 통해 설정한다. `CaffeineCacheManager`가 자동으로 등록된다.<br>
간단한 캐시 정책을 적용하는 경우 적합하다.
```yml
spring:
  cache:
    cache-names: "products,product-list"
    caffeine:
      spec: "maximumSize=1000,expireAfterWrite=600s"
```
### 2. Java Config
명시적으로 `CacheManager` 빈을 선언한다.<br>
세밀한 설정이 필요한 경우 적합하다.
```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager cacheManager = new CaffeineCacheManager();
    cacheManager.setCacheNames(List.of("products", "product-list"));
    cacheManager.setCaffeine(
            Caffeine.newBuilder()
                    .expireAfterWrite(600, TimeUnit.SECONDS)
                    .maximumSize(1000)
    );
    return cacheManager;
}
```

### 3. Java Config + Enum을 통한 캐시 정책 관리
캐시 정책이 다양한 경우 유용하다.
 
**CacheType 작성**<br>
캐시 항목별 세부 설정을 관리하는 Enum이다.
```java
@Getter
public enum CacheType {
    PRODUCTS("products", 600, 1000),
    PRODUCT_LIST("product-list", 300, 500);

    CacheType(String cacheName, int expireAfterWrite, int maximumSize) {
        this.cacheName = cacheName;
        this.expireAfterWrite = expireAfterWrite;
        this.maximumSize = maximumSize;
    }

    private final String cacheName;
    private final int expireAfterWrite;
    private final int maximumSize;
}
```

**CacheManager 빈 등록**<br>
위에서 정의한 `CacheType`을 이용해 `CacheManager`빈을 생성한다.
```java
@Bean
public CacheManager cacheManager() {
    SimpleCacheManager cacheManager = new SimpleCacheManager();
    // 각 캐시 타입에 대한 설정 적용
    List<CaffeineCache> caches = Arrays.stream(CacheType.values())
            .map(cache ->
                    new CaffeineCache(
                            cache.getCacheName(),
                            Caffeine.newBuilder()
                                    .expireAfterWrite(cache.getExpireAfterWrite(), TimeUnit.SECONDS)
                                    .maximumSize(cache.getMaximumSize())
                                    .build()
                    )
            )
            .toList();
    cacheManager.setCaches(caches);
    return cacheManager;
}
```

### 설정 값
- **maximumSize**: 캐시 최대 크기
- **maximumWeight**: 캐시 최대 무게 `Caffeine.newBuilder().maximumWeight(10).weigher((k, v) -> k.toString().length()).build()`
- **expireAfterAccess**: 항목 읽기/쓰기 이후 일정 시간 지나면 만료시킨다.
- **expireAfterWrite**: 항목 쓰기 이후 일정 시간 지나면 만료시킨다.
- **refreshAfterWrite**: 일정 시간이 지나면 백그라운드에서 새로고침한다.
- **weakKeys**: key가 *WeakReference*로 저장된다. 값을 비교할 때 `equals()` 대신 `==`을 사용한다.
- **weakValues**: value가 *WeakReference*로 저장된다. 값을 비교할 때 `equals()` 대신 `==`을 사용한다.
- **softValues**: key가 *SoftReference*로 저장된다. 값을 비교할 때 `equals()` 대신 `==`을 사용한다.
- **initialCapacity**: 내부 해시 테이블의 최소 크기
- **recordStats**: 통계 수집

> **WeakReference, SoftReference란?**
> - `WeakReference`: 객체를 참조하는 곳이 없으면 GC 대상으로 지정된다. 메모리 누수를 방지하는데 유용하다.
> - `SoftReference`: 객체를 참조하는 곳이 없더라도 메모리가 넉넉하다면 GC 대상이 되지 않는다.

### 캐싱 기능 활성화
`@EnableCaching` 어노테이션을 추가해 캐싱 기능을 활성화한다.
```java
@EnableCaching
@Configuration
public class CacheConfig {

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
