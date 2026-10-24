# [FastAPI] Lifespan
**lifespan**에는 애플리케이션이 시작될 때와 종료될 때 실행할 로직을 정의할 수 있다.

이는 데이터베이스 연결 풀과 같이 애플리케이션 전체에서 사용하는 자원을 초기화하고, 요청 간에 공유되는 자원을 설정하고, 애플리케이션 종료 시 정리하는 데 유용하다.

`state`는 lifespan과 요청 간에 객체를 공유하는 데 사용할 수 있는 딕셔너리이다.
요청에서 접근하는 `request.state`는 lifespan에서 제공한 `state`의 얕은 복사본(shallow copy)이다.

## 예시
```py
import contextlib
from typing import AsyncIterator, TypedDict

from fastapi import FastAPI
from sqlalchemy.ext.asyncio import AsyncEngine, create_async_engine


class State(TypedDict):
    engine: AsyncEngine


@contextlib.asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncIterator[State]:
    # 애플리케이션 시작 시
    engine = create_async_engine("sqlite+aiosqlite:///./test.db")

    # 요청 간 공유할 자원 전달
    yield {"engine": engine}

    # 애플리케이션 종료 시
    await engine.dispose()


app = FastAPI(lifespan=lifespan)
```

```py
from fastapi import Request


@app.get("/")
async def hello(request: Request):
    # lifespan에서 생성한 engine을 가져옴
    engine = request.state.engine
    return {"message": "Hello World"}
```

---
**Reference**
- https://starlette.dev/lifespan/
