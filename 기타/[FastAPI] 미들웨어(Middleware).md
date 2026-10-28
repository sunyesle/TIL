# [FastAPI] 미들웨어(Middleware)

**미들웨어**는 요청이 엔드포인트에 도달하기 전과 응답이 반환되는 과정에서 공통 작업을 수행하는 구성 요소이다. 요청과 응답을 가로채 필요한 처리를 수행할 수 있다.

## 처리 흐름
- 미들웨어는 애플리케이션으로 들어오는 요청을 받는다.
- 해당 요청에 대해 작업을 수행하거나 필요한 코드를 실행할 수 있다.
- 다음 미들웨어 또는 엔드포인트로 요청을 전달하고, 응답을 대기한다.
- 해당 응답에 대해 작업을 수행하거나 필요한 코드를 실행할 수 있다.
- 최종 응답을 반환한다.

## 예시

```py
import time

from fastapi import FastAPI, Request

app = FastAPI()


# 데코레이터로 HTTP 미들웨어를 등록한다.
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()

    # 다음 미들웨어 또는 엔드포인트로 요청을 전달하고 응답을 대기한다.
    response = await call_next(request)
    
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)

    # 응답을 반환한다.
    return response
```

## 미들웨어 실행 순서

`app.middleware()` 데코레이터 또는 `app.add_middleware()` 메서드를 사용해 여러 개의 미들웨어를 추가하면, 새로 추가된 각 미들웨어가 애플리케이션을 감싸 스택을 형성한다. 마지막에 추가된 미들웨어가 가장 바깥쪽이고, 처음에 추가된 미들웨어가 가장 안쪽이다.

```py
app.add_middleware(MiddlewareA)
app.add_middleware(MiddlewareB)
```
이 경우 실행순서는 다음과 같다.

- **요청**: `MiddlewareB` → `MiddlewareA` → route
- **응답**: route → `MiddlewareA` → `MiddlewareB`

---
**Reference**
- https://fastapi.tiangolo.com/ko/tutorial/middleware/
- https://fastapi.tiangolo.com/ko/advanced/middleware/
