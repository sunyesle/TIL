# 재시도(Retry) 패턴과 백오프 전략

분산 시스템 구조에서 서버와 서버 간의 네트워크 통신은 언제든지 실패할 수 있다.

만약 한 번의 네트워크 호출 실패만으로 서비스의 비즈니스 로직 전체를 실패 처리한다면,
실제로는 몇 번의 재시도로 해결될 수 있었던 상황에서도 과도한 리소스를 낭비하게 된다.

## 재시도 패턴
재시도 패턴은 분산 시스템에서 일시적인 실패를 처리하는 알고리즘이다.
작업이 실패할 경우 동일한 요청을 다시 전송하여 작업을 재시도 한다.

## 적용 시 고려사항
- 재시도 횟수와 시간
- 백오프(Backoff) 전략
- 어떤 에러를 재시도 할 것인지
  - 일시적인지
  - 복구 가능한지
- 멱등성 보장 여부

시스템의 복원력을 높이려면 적절한 재시도 횟수와 재시도 간의 지연시간을 설정해야 한다.
과도한 재시도는 오히려 시스템에 부담을 가중시킬 수 있으므로 주의해야 한다.

복구 가능한 상황에서 적용해야 한다.
예를 들어, 명확한 비지니스 로직의 실패가 아닌, 네트워크의 일시적 장애로 인한 Read Timeout 같은 오류가 재시도를 고려해 볼 수 있는 상황이다.

안전한 재시도를 위해서는 멱등성을 보장해야 한다.
예를 들어, 결제 서버의 경우 동일한 결제 요청이 여러 번 도달하더라도 실제 결제는 한 번만 처리되어야 한다.

> **멱등성**<br>
> 같은 요청을 여러 번 수행하더라도 처음 요청과 동일한 결과를 갖도록 하는 성질을 의미한다.

---
**Reference**<br>
- https://news.hada.io/topic?id=20872
- https://www.infoq.com/news/2025/04/resilient-distributed-systems/
- https://hudi.blog/safely-handling-network-errors/
- https://gist.github.com/eugene70/87e15438bcde0a71a16c3fc1e7c9bfbc
- https://lifeonroom.com/study-lab/develop-tip/difference-between-circuit-breaker-and-retry-in-spring-boot/
- https://jungseob86.tistory.com/12
- https://www.baeldung.com/resilience4j-backoff-jitter
