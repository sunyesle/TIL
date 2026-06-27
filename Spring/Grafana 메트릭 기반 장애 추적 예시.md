# Grafana 메트릭 기반 장애 추적 예시
k6를 활용하여 인위적으로 장애 상황을 재현하고, Grafana 대시보드의 매트릭 변화를 통해 각 장애 상황이 발생했을 때 어떤 위험 신호가 감지되는지 확인해 보자.

## 테스트 환경 설정
테스트 편의성을 위해 다음과 같이 설정한다.

> application.properties
```properties
# DB 커넥션
spring.datasource.hikari.maximum-pool-size=3     # HikariCP의 커넥션 풀 사이즈 3개로 제한
spring.datasource.hikari.connection-timeout=3000 # 커넥션 획득 대시 최대 시간 3초. 에러를 빨리 보기 위함

# 톰켓 커넥션
server.tomcat.threads.max=10          # 최대 스레드 수를 10개로 제한
server.tomcat.threads.accept-count=20 # 요청 대기 큐 사이즈를 20개로 제한
```

> JVM 옵션

최소/최대 힙 메모리를 256MB로 고정한다.
- **IntelliJ**: Run/Debug Configurations → 애플리케이션 선택 → Modify options → Add VM options 클릭 후 -Xms256m -Xmx256m 입력
- **Jar 실행 시**: java -Xms256m -Xmx256m -jar name.jar

## 1. 메모리 누수 및 GC 과부하
static 컬렉션에 데이터를 계속 쌓거나, 캐시에 적절한 만료 정책을 두지 않으면 메모리가 점진적으로 증가한다. 이로 인해 JVM은 메모리를 확보하기 위해 GC 빈번하게 수행하게 된다. GC가 과도하게 발생하면 성능이 저하되고, 메모리를 회수하지 못하는 상태에 이르면 `OutOfMemoryError`(OOM)가 발생하게 된다.

### 재현 코드
호출할 때마다 static List에 큰 객체를 추가하는 `/api/memory/leak` 엔드포인트를 만든다.
```java
@RestController
@RequestMapping("/api/memory")
public class MemoryLeakController {

    private static final List<DummyObject> leakList = new ArrayList<>();

    // 요청이 올 때마다 1MB짜리 객체를 리스트에 추가
    @GetMapping("/leak")
    public String makeLeak() {
        leakList.add(new DummyObject());
        return "Total items: " + leakList.size();
    }

    // 메모리를 비움
    @GetMapping("/reset")
    public String resetMemory() {
        leakList.clear();
        return "Memory cleared!";
    }
}
```

```java
public class DummyObject {
    private byte[] heavyData = new byte[1024 * 1024];
    private long timestamp = System.currentTimeMillis();
}
```

### k6 스크립트
적당한 부하로 지속해서 호출한다.
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
    stages: [
        { duration: '30s', target: 7 },
        { duration: '10s', target: 0 },
    ],
};

export default function () {
    http.get('http://localhost:8080/api/memory/leak');
    sleep(1);
}
```

### 관측 포인트
GC 횟수가 급증하고 GC 후에도 메모리가 감소되지 않는다.
- `jvm_memory_used{jvm_memory_type="heap"}`
- `jvm_gc_pause_count`
- `jvm_gc_pause_sum`
- `jvm_gc_live_data_size`

## 2. 톰캣 스레드 풀 고갈
동기식 스프링 부트는 요청마다 하나의 톰캣 스레드가 할당된다. 외부 시스템 지연이나 느린 비즈니스 로직으로 인해 특정 요청이 스레드를 오래 점유하면 가용 스레드 수가 감소한다. 모든 스레드가 사용 중인 상태가 되면 새로운 요청은 처리되지 못하고 대기하게 된다. 이로 인해 단순하고 빠른 API까지 함께 지연되며, 결국 전체 서비스가 응답 불가 상태에 빠질 수 있다.

### 재현 코드
응답까지 5초가 걸리는 `/api/tomcat/heavy`와, 즉시 응답하는 `/api/tomcat/light` 두 개의 엔드포인트를 만든다.
```java
@RestController
@RequestMapping("/api/tomcat")
public class ThreadPoolController {

    // 톰캣 스레드를 5초 동안 붙잡고 있는 API
    @GetMapping("/heavy")
    public String heavyRequest() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        return "Heavy Task Done";
    }

    // 즉시 응답해야 하는 API
    @GetMapping("/light")
    public String lightRequest() {
        return "Light Task Done";
    }
}
```

### k6 스크립트
30명의 유저는 `/api/tomcat/heavy`를 호출하고, 중간에 2명의 유저가 `/api/tomcat/light`를 호출한다.
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
    scenarios: {
        // 시나리오 1: 무거운 API로 스레드 풀을 꽉 채워 고갈시킴
        heavy_load: {
            executor: 'constant-vus',
            vus: 30, // 톰캣 스레드 풀(10개)보다 많은 30명의 유저 투입
            duration: '20s',
            exec: 'runHeavy',
        },
        // 시나리오 2: 중간에 가벼운 API를 호출해봄
        light_check: {
            executor: 'constant-vus',
            vus: 2,
            duration: '20s',
            exec: 'runLight',
            startTime: '3s', // 부하가 걸린 지 3초 뒤부터 시작
        },
    },
};

export function runHeavy() {
    http.get('http://localhost:8080/api/tomcat/heavy');
    sleep(0.1);
}

export function runLight() {
    http.get('http://localhost:8080/api/tomcat/light');
    sleep(0.1);
}
```

### 관측 포인트
작업이 지연되면 `timed_waiting` 상태의 스레드가 증가한다. 이후 톰캣 스레드 부족으로 `waiting` 상태의 스레드가 증가하며, 응답시간이 지연된다.
- `jvm_thread_count{jvm_thread_state="timed_waiting"}`
- `jvm_thread_count{jvm_thread_state="waiting"}`
- `http_server_request_duration_sum`
- `http_server_request_duration_count`

## 3. DB 커넥션 풀 고갈
트랜잭션 범위가 과도하게 길거나, 외부 API 호출과 같은 작업이 트랜잭션 내부에서 수행될 때 발생한다. 스레드가 DB 커넥션을 장시간 점유하게 되어 커넥션 반환이 지연된다. 커넥션 풀이 모두 사용되면 이후 요청들은 커넥션을 얻기 위해 대기 상태에 들어가고, 대기 시간이 임계치를 초과하면 `ConnectionTimeoutException`이 발생한다.

### 재현 코드
`@Transactional`을 걸고, `Thread.sleep(3000)`으로 데이터베이스를 붙잡고 있는 `/api/conn/leak` 엔드포인트를 만든다.
```java
@RestController
@RequestMapping("/api/conn")
@RequiredArgsConstructor
public class DbConnectionController {

    private final ItemRepository repository;

    // DB 커넥션을 5초 동안 붙잡고 있는 API
    @GetMapping("/leak")
    @Transactional
    public String slowQuery() {
        repository.findById(1L);

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        return "Success";
    }
}
```

```java
@Entity
public class Item {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long Id;
    private String name;
}
```

```java
public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

### k6 스크립트
여러 명의 유저가 동시에 호출한다.
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
    vus: 10, // DB 커넥션 풀(3개)보다 많은 10명의 유저 투입
    duration: '30s',
};

export default function () {
    http.get('http://localhost:8080/api/conn/leak');
    sleep(0.1);
}
```

### 관측 포인트
`active` 커넥션 수가 `max` 커넥션 수와 같아지고, `pending` 커넥션이 증가한다. 이후 `timeout`이 발생한다.
- `hikaricp_connections_idle`
- `hikaricp_connections_active`
- `hikaricp_connections_pending`
- `hikaricp_connections_max`
- `hikaricp_connections_timeout`

## 4. CPU 과부하
암호화 해싱, 대용량 데이터 파싱, 복잡한 연산 등 CPU를 많이 쓰는 작업이 몰릴 때의 현상이다. CPU가 포화되면 스레드는 살아있지만 실제 작업 처리가 지연된다. 전체 응답 속도가 급격히 느려지며, 타임아웃 및 장애로 이어질 수 있다.

### 재현 코드
반복적인 연산을 수행하는 `/api/cpu/contention` 엔드포인트를 만든다.
```java
@RestController
@RequestMapping("/api/cpu")
public class CpuContentionController {

    private final Object lock = new Object();

    @GetMapping("/contention")
    public String simpleCpuTask() {
        long startTime = System.currentTimeMillis();

        // 300밀리초 동안 CPU 코어 하나를 점유 한다.
        while (System.currentTimeMillis() - startTime < 300) {
            double value = ThreadLocalRandom.current().nextDouble();
            double result = Math.sin(value) * Math.cos(value);
        }
        return "CPU Task Done";
    }
}
```
### k6 스크립트
많은 유저가 동시에 호출한다.
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
    vus: 25,
    duration: '30s',
};

export default function () {
    http.get('http://localhost:8080/api/cpu/contention');
}
```

### 관측 포인트
CPU 사용률이 급증하며, 응답시간이 지연된다.
- `system_cpu_usage`
- `http_server_request_duration_sum`
- `http_server_request_duration_count`

## 5. 데드락
두 개 이상의 스레드가 서로가 점유한 락을 기다리며 무한 대기 상태에 빠지는 상황이다. 어떤 스레드도 작업을 진행하지 못하고 시스템이 멈춘 것처럼 보이게 된다.

### 재현 코드
A → B 순서로 락을 획득하는 `/api/deadlock/1`, B → A 순서로 락을 획득하는 `/api/deadlock/2` 엔드포인트를 만든다.
```java
@RestController
@RequestMapping("/api/deadlock")
public class DeadlockController {

    private final Object lockA = new Object();
    private final Object lockB = new Object();

    // Lock A를 잡고 1초 대기 후 Lock B를 요청
    @GetMapping("/1")
    public String pipelineOne() {
        synchronized (lockA) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }

            synchronized (lockB) {
            }
        }
        return "1 Complete";
    }

    // Lock B를 잡고 1초 대기 후 Lock A를 요청
    @GetMapping("/2")
    public String pipelineTwo() {
        synchronized (lockB) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }

            synchronized (lockA) {
            }
        }
        return "2 Complete";
    }
}
```

### k6 스크립트
두 엔드포인트를 동시에 호출한다.
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
    vus: 10,
    duration: '15s',
};

export default function () {
    // 1번과 2번 엔드포인트를 교차 호출하여 충돌 유도
    if (__VU % 2 === 0) {
        http.get('http://localhost:8080/api/deadlock/1');
    } else {
        http.get('http://localhost:8080/api/deadlock/2');
    }

    sleep(0.1);
}
```

### 관측 포인트
`blocked` 상태의 스레드가 증가하고, 요청 처리가 진행되지 않는다.
- `jvm_thread_count{jvm_thread_state="blocked"}`
