# Resilience4j

## Resilience4j란
Resilience4j는 MSA와 같이 환경에서 여러 컴포넌트 간 원격 통신에 대한 장애 허용(fault tolerance)을 관리하여 더욱 회복력 있는 시스템을 구축하는 데 도움을 주는 경량화 라이브러리이다.

> **Fault tolerance(장애 허용, 결함 내성)**<br>
> 서비스의 일부에서 오류가 발생하더라도 정상적인 서비스처럼 작동할 수 있는 능력을 의미한다.

Resilience4j는 다음과 같은 6가지 모듈을 제공하고 있다.
- `resilience4j-circuitbreaker`: 서킷 브레이커 처리
- `resilience4j-bulkhead`: 병렬 작업 제한 관리
- `resilience4j-ratelimiter`: 요청 제한 관리
- `resilience4j-retry`: 재시도 관리
- `resilience4j-timelimiter`: 실행 시간 제한 관리
- `resilience4j-cache`: 캐시 처리

각 모듈은 다음과 같은 우선순위로 적용된다.
1. BulkHead
2. TimeLimiter
3. RateLimiter
4. CircuitBreaker
5. Retry

## CircuitBreaker
### 옵션

| 옵션                                         | default 값          | 설명                                                                                                                             |
|----------------------------------------------|--------------------|--------------------------------------------------------------------------------------------------------------------------------|
| failureRateThreshold                         | 50                 | 서킷 브레이커가 Open 상태로 전환되는 실패율 임계값                                                                                                 |
| slowCallRateThreshold                        | 100                | 서킷 브레이커가 Open 상태로 전환되는 느린호출 비율 임계값                                                                                             |
| slowCallDurationThreshold                    | 60000 [ms]         | 느린 호출의 기준 시간                                                                                                                   |
| permittedNumberOfCallsInHalfOpenState        | 10                 | 서킷 브레이커가 Half-open 상태에서 허용하는 최대 호출 수                                                                                           |
| maxWaitDurationInHalfOpenState               | 0 [ms]             | Half-open 상태에서 대기하는 최대 시간                                                                                                      |
| slidingWindowType                            | COUNT_BASED        | 서킷 브레이커가 동작할 때 사용할 슬라이딩 윈도우 타입<br>**COUNT_BASED**: 마지막 N번의 호출 결과를 기반으로 상태를 결정<br>**TIME_BASED**: 마지막 N초 동안의 호출 결과를 기반으로 상태를 결정 |
| slidingWindowSize                            | 100                | 슬라이딩 윈도우 크기                                                                                                                    |
| minimumNumberOfCalls                         | 100                | 서킷 브레이커가 동작하기 위해 필요한 최소한의 호출 수                                                                                                 |
| waitDurationInOpenState                      | 60000 [ms]         | Open -> Half-open 변경 대기 시간                                                                                                     |
| automaticTransitionFromOpenToHalfOpenEnabled | false              | Open -> Half-open 자동 변경 여부                                                                                                     |
| recordExceptions                             | empty              | 실패로 측정하는 예외 리스트                                                                                                                |
| ignoreExceptions                             | empty              | 실패로 처리하지 않을예외 리스트                                                                                                              |
| recordFailurePredicate                       | throwable -> true  | 특정 예외가 실패로 측정되도록 하는 커스텀예외 (기본값으로 모든 예외는 실패로 기록)                                                                                |
| ignoreExceptionPredicate                     | throwable -> false | 특정 예외가 측정되지 않도록 하는 커스텀예외 (기본값으로 모든 예외는 무시되지 않음)                                                                                |

### 예시
```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        registerHealthIndicator: true
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 5s
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        minimum-number-of-calls: 5
```

## Retry
### 옵션
| 옵션                    | default 값                                                  | 설명                                                                                                             |
|-------------------------|------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| maxAttempts             | 3                                                          | 최대 재시도 수                                                                                                       |
| waitDuration            | 500 [ms]                                                   | 재시도 간격. 재시도 사이의 고정된 대기 시간                                                                                      |
| intervalFunction        | numOfAttempts -> waitDuration                              | 시도 횟수를 기반으로 실패 후 대기 간격을 설정하는 함수. 기본적으로 대기 시간(waitDuration)이 일정하게 유지된다.                                         |
| intervalBiFunction      | (numOfAttempts, Either<throwable, result>) -> waitDuration | 시도 횟수와 결과 또는 예외를 기반으로 실패 후 대기 간격을 설정하는 함수. intervalFunction과 함께 사용하면 `IllegalStateException`이 발생한다.            |
| retryOnResultPredicate  | result -> false                                            | 결과에 따라 재시도 여부를 결정하기 위한 Predicate                                                                               |
| retryExceptionPredicate | throwable -> true                                          | 예외에 따라 재시도 여부를 결정하기 위한 Predicate                                                                               |
| retryExceptions         | empty                                                      | 실패로 기록되는 재시도되는 에러 클래스 리스트                                                                                      |
| ignoreExceptions        | empty                                                      | 무시되어 재시도되지 않는 에러 클래스 리스트                                                                                       |
| failAfterMaxAttempts    | false                                                      | 설정한 maxAttempts만큼 재시도하고 나서도 결과가 여전히 retryOnResultPredicate를 통과하지 못했을 때 `MaxRetriesExceededException`을 발생시킬지 여부 |

### 예시
```yaml
resilience4j:
  retry:
    configs:
      default:
        maxAttempts: 3
        waitDuration: 10s
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        retryExceptions:
            - org.springframework.web.client.HttpServerErrorException
            - java.io.IOException
        ignoreExceptions:
            - com.example.sunyesle.exception.BusinessException
```

## TimeLimiter
### 옵션

| 옵션                  | default 값 | 설명                             |
|---------------------|-----------|--------------------------------|
| cancelRunningFuture | true      | timeout이 경과한 후 future 자동 취소 여부 |
| timeoutDuration     | 1000 [ms] | timeout 시간                     |

### 예시
```yaml
resilience4j:
  timelimiter:
    configs:
      default:
        timeout-duration: 3s
```

---
**Reference**<br>
- https://resilience4j.readme.io/docs/getting-started
- https://seongwon.dev/MSA/20230428-Resilience4J%EB%A1%9C_SpringBoot%EC%97%90%EC%84%9C_CircuitBreaker_%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/
- https://happy-jjang-a.tistory.com/265
- https://developer-jinnie.tistory.com/83
