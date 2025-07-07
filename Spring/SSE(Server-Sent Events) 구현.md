# SSE(Server-Sent Events) 구현

## SSE란
SSE는 서버로부터 클라이언트에 실시간으로 데이터를 전달할 수 있는 기술 중 하나이다.
HTTP 기반으로, 서버의 데이터를 실시간으로 streaming 한다.

### WebSocket과의 차이점
가장 큰 차이점은 **WebSocket**은 서버와 클라이언트 간의 **양방향 통신**을 지원하고, **SSE**는 서버에서 클라이언트로의 **단방향 통신**을 지원한다는 점이다.

|            | WebSocket                     | Server-Sent-Event                            |
|------------|-------------------------------|----------------------------------------------|
| 통신 방향      | 양방향                           | 단방향(서버에서 클라이언트로)                             |
| 브라우저 지원    | 대부분 브라우저에서 지원                 | 대부분 모던 브라우저에서 지원(polyfills 가능)               |
| 데이터 형태     | Binary, UTF-8                 | UTF-8                                        |
| 자동 재연결     | 브라우저에서 자동 재연결 지원 안함. 직접 구현 필요 | 브라우저에서 자동 재연결 지원                             |
| 최대 동시 접속 수 | 브라우저 연결 한도는 없지만 서버 설정에 따라 다름  | HTTP/1.1 브라우저당 최대 6개 / HTTP/2 브라우저당 100개 기본값 |
| 프로토콜       | WebSocket                     | HTTP                                         |

![websocket sse](https://github.com/user-attachments/assets/3abb5389-5ad2-44e6-a0d4-fcab31a60342)

## SSE 통신 개요
SSE를 이용하는 통신의 개요는 다음과 같다.

<img width="1009" alt="Image" src="https://github.com/user-attachments/assets/5e50d037-b49f-4854-bab6-7b6bfc94830e" />

### Client: SSE 연결 요청
SSE의 미디어 타입은 `text/event-stream`이다.
이벤트는 캐싱하지 않으며, 지속적 연결을 사용해야 한다.
```
GET /connect HTTP/1.1
Accept: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

### Server: SSE 연결 요청 응답
응답의 미디어 타입은 `text/event-stream`이다.
서버는 전달할 이벤트가 생길 때마다 데이터를 Chuck 단위로 전달한다.
```
HTTP/1.1 200 OK
Content-Type: text/event-stream;charset=UTF-8
Cache-Control: no-cache
Connection: keep-alive
Transfer-Encoding: chunked
```

### Server: 이벤트 전달
서로 다른 이벤트는 줄바꿈 문자 두개(`\n\n`)로 구분된다.
각 이벤트는 한 개 이상의 `name:value` 필드로 구성되며 이들은 줄바꿈 문자 한개(`\n`)로 구분된다.
- id: 이벤트 id
- event: 이벤트 타입
- data: 이벤트 데이터. 데이터가 많으면 Multiline으로 구성한다.
- comment: 주석. 콜론으로 시작한다. 클라이언트에 의해 무시되며 연결 유지(heartbeat), 디버깅 등의 용도로 사용된다.
```
id:0
event:type1
data:hello

id:1
event:type2
data:{
data:"msg":"2",
data:"id":123
data:}

:hartbeat
```

## 클라이언트 예시 코드
```js
// SSE 이벤트 수신
const eventSource = new EventSource(`/sse/connect?userId=${Math.random()}`);


// 연결됐을 때
eventSource.onopen = (e) => {
    console.log(e).data);
};

// 오류가 발생했을 때
eventSource.onerror = (e) => {
    console.error("EventSource failed:", e);
};

// 이벤트가 수신되었을 때
// 이벤트 타입 필드가 없거나, 이벤트 타입이 "message"인 경우
eventSource.onmessage = (e) => {
    console.log(e.data)
}

// 이벤트가 수신되었을 때
// 이벤트 타입이 "notice"인 경우
eventSource.addEventListener("notice", (e) => {
    console.log(e.data);
});
```

## 서버 예시 코드
Spring Framework 4.2부터 SSE 통신을 지원하는 `SseEmitter` API를 제공한다.

**Controller**
```java
@RestController
@RequestMapping("/api/sse")
@RequiredArgsConstructor
public class SseController {
    private final SseService sseService;

    @GetMapping(value = "/connect", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter connect(@RequestParam String userId) {
        return sseService.connect(userId);
    }

    @PostMapping("/send")
    public void send(@RequestBody SseRequest request) {
        sseService.send(request);
    }
}
```

**DTO**
```java
public record SseRequest(String userId, String message) {
}
```

**Service**
```java
@Service
@RequiredArgsConstructor
public class SseService {
    private static final long DEFAULT_TIMEOUT = 1000L * 60 * 10; // SSE emitter 연결 시간, 10분

    // userId 별로 SseEmitter 객체를 저장한다.
    // 멀티스레드 환경에서 동시성을 보장하기 위해 ConcurrentHashMap 사용한다.
    private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

    // SSE 연결 생성
    public SseEmitter connect(String userId) {
        SseEmitter emitter = new SseEmitter(DEFAULT_TIMEOUT);
        emitters.put(userId, emitter);

        emitter.onCompletion(() -> {
            emitters.remove(userId);
        });
        emitter.onTimeout(() -> {
            emitter.complete();
            emitters.remove(userId);
        });
        emitter.onError(e -> {
            emitter.completeWithError(e);
            emitters.remove(userId);
        });

        // SseEmitter의 유효 시간동안 아무런 데이터도 전송하지 않으면 503 에러가 발생한다.
        // 503 Service Unavailable 에러를 방지하기 위해 더미 이벤트 전송한다.
        sendEvent(userId, "connect", "SSE connection established");

        return emitter;
    }

    // SSE 이벤트 전송
    public void send(SseRequest request) {
        sendEvent(request.userId(), "message", request.message());
    }

    private void sendEvent(String userId, String eventName, Object data) {
        SseEmitter emitter = emitters.get(userId);
        if (emitter == null) {
            return;
        }

        try {
            emitter.send(SseEmitter.event()
                    .name(eventName)
                    .data(data));
        } catch (IOException e) {
            emitter.completeWithError(e);
            emitters.remove(userId);
        }
    }
}
```

---
**Reference**<br>
- https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- https://gong-check.github.io/dev-blog/BE/%EC%96%B4%EC%8D%B8%EC%98%A4/sse/sse/
- https://jforj.tistory.com/419
