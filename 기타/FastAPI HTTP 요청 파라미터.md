# FastAPI HTTP 요청 파라미터

## 파라미터 해석
FastAPI는 파라미터의 이름과 타입을 바탕으로 데이터를 가져올 위치를 추론한다.

- 파라미터 이름이 경로에 포함되어 있다면 → **Path**
- 파라미터가 단일 타입(`int`, `str`, `float`, `bool` 등)으로 선언되어 있다면 → **Query**
- 파라미터가 복합 타입(`Pydantic 모델`, `dict`, `list` 등)으로 선언되어 있다면 → **Body**
```py
from fastapi import FastAPI
from pydantic import BaseModel

# BaseModel을 상속받아 데이터 모델을 선언한다.
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(
    item_id: int,           # Path Parameter
    tag: str | None = None, # Query Parameter
    item: Item,             # Request Body
):
    return {
        "item_id": item_id,
        "tag": tag,
        "item": item
    }
```

## Path, Query, Body, Cookie, Header
`Path`, `Query`, `Body`, `Cookie`, `Header` 클래스를 사용하면 데이터의 위치와 검증 규칙을 명시적으로 지정할 수 있다.

```py
from typing import Annotated
from fastapi import Body, Cookie, FastAPI, Header, Path, Query
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(
    item_id: Annotated[int, Path()],                    # Path Parameter
    tag: Annotated[list[str] | None, Query()]= None,    # Query Parameter
    item: Annotated[Item, Body()],                      # Request Body
    ads_id: Annotated[str | None, Cookie()] = None,     # Cookie Parameter
    user_agent: Annotated[str | None, Header()] = None, # Header Parameter
):
    return {
        "item_id": item_id,
        "item": item,
        "tag": tag,
        "ads_id": ads_id,
        "user_agent": user_agent
    }
```

> **Header 자동 변환**: 대부분의 표준 헤더는 `-`문자로 구분된다. 파라미터명의 `_`를 `-`으로 자동 변환하여 매핑한다.

## Form, File
업로드된 파일 또는 폼 데이터를 받으려면 `python-multipart`를 설치해야 한다.
```bash
$ pip install python-multipart
```

<br>

```py
from typing import Annotated
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(
    file: Annotated[bytes, File()],
    fileb: Annotated[UploadFile, File()],
    token: Annotated[str, Form()],
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

> `UploadFile`은 자동으로 `File`로 인식된다. (`file: Annotated[UploadFile, File()]` = `file: UploadFile`)

### bytes vs UploadFile

| 특징 | bytes | UploadFile |
|---|---|---|
| **저장 방식** | 파일 전체를 메모리에 적재 | 스트리밍 처리(임시 파일 활용) |
| **메모리 효율** | 낮음(OOM 발생 위험) | 높음 |
| **메타데이터** | 없음 | `filename`, `content_type` 등 확인 가능 |
| **권장 용도** | 아주 작은 파일 | 일반적인 파일 업로드 |

---
**Reference**
- https://fastapi.tiangolo.com/tutorial/path-params/

