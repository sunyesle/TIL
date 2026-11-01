# [FastAPI] JSON Lines 스트리밍
## JSON Lines
**JSON Lines**는 한 줄에 하나의 JSON 객체를 보내는 형식이다.

응답의 콘텐츠 타입은 `application/jsonl`이고, 본문은 다음과 같다.
```
{"id": 1, "name": "상품A"}
{"id": 2, "name": "상품B"}
{"id": 3, "name": "상품C"}
```
JSON 배열과 매우 비슷하지만, 항목들을 `[]`로 감싸고 항목 사이에 `,`를 넣는 대신, 줄마다 하나의 JSON 객체가 있고, 새 줄 문자(`\n`)로 구분된다.

## 예시
JSON Lines를 스트리밍하려면, 경로 처리 함수에서 `return`을 사용하는 대신 `yield`로 각 항목을 차례로 생성하면 된다.

함수가 `async`라면 `AsyncIterable[...]`, 함수가 async가 아니라면 `Iterable[...]`로 반환타입을 선언할 수 있다.

```py
from collections.abc import AsyncIterable, Iterable

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    id: int
    name: str


items = [
    Item(id=1, name="상품A"),
    Item(id=2, name="상품B"),
    Item(id=3, name="상품C"),
]

@app.get("/items/stream")
async def stream_items() -> AsyncIterable[Item]:
    for item in items:
        yield item


@app.get("/items/stream-no-async")
def stream_items_no_async() -> Iterable[Item]:
    for item in items:
        yield item
```

## 활용
- 대용량 데이터 조회 및 다운로드
- AI 모델 응답 스트리밍
- 실시간 로그 및 이벤트 전달

---
**Reference**
- https://fastapi.tiangolo.com/ko/tutorial/stream-json-lines
