# [Python] 제네릭(Generics)

## 제네릭 함수
함수 이름 뒤에 타입 파라미터를 선언한다. 이렇게 선언된 T는 해당 함수 안에서만 유효한 지역적인 스코프를 가지며, 런타임에는 자동으로 `TypeVar` 객체가 만들어진다.

```py
# Python 3.12 이상
def first[T](items: list[T]) -> T:
    return items[0]
```

Python 3.11 이하에서는 타입 변수를 전역에 선언해야 한다.
```py
# Python 3.11 이하
T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]
```

## 제네릭 클래스
클래스 이름 뒤에 타입 파라미터를 선언한다. 내부적으로 자동으로 `Generic`이 베이스 클래스에 추가된다.
```py
# 파이썬 3.12 이상
class Box[T]:
    def __init__(self, content: T) -> None:
        self.content: T = content

    def get_content(self) -> T:
        return self.content

int_box: Box[int] = Box(100)
str_box: Box[str] = Box("Hello")    
```

Python 3.11 이하에서는 `Generic`을 상속받아야한다.
```py
# Python 3.11 이하
T = TypeVar("T")

class Box(Generic[T]):
    def __init__(self, content: T) -> None:
        self.content: T = content

    def get_content(self) -> T:
        return self.content
```

## 상한 제한(Bound)
타입 변수가 특정 타입 또는 해당 타입의 하위 타입으로 제한되도록 한다.
```py
def get_length[T: str](value: T) -> int:
    return len(value)

get_length("hello") # 허용
get_length(123) # 타입 오류
```

```py
from typing import Protocol

class HasName(Protocol):
    name: str

# name 속성을 가진 객체만 허용한다.
def print_name[T: HasName](obj: T) -> T:
    print(obj.name)
    return obj
```

## 제약 조건(Constraint)
타입 변수가 정해진 타입 후보 중 하나로 결정되도록 제한한다.
```py
# T는 str 또는 bytes 중 하나여야 한다.
def concat[T: (str, bytes)](a: T, b: T) -> T:
    return a + b

concat("hello", "world") # 허용
concat("hello", b"world") # 타입 오류
concat(10, 20) # 타입 오류
```

## ParamSpec
타입 파라미터 앞에 별표 두개(`**`)를 붙이면 `ParamSpec`을 선언할 수 있다. 함수 시그니처(매개변수 목록과 키워드 인수)를 표현한다.

```py
from collections.abc import Callable
from functools import wraps
from time import perf_counter

def timed[**P, R](func: Callable[P, R]) -> Callable[P, R]:

    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        start = perf_counter()

        result = func(*args, **kwargs)

        elapsed = perf_counter() - start
        print(f"{elapsed:.4f}초")

        return result

    return wrapper

@timed
def create_item(name: str, price: int) -> dict:
    return {"name": name, "price": price}

user = create_item("A000", 2000)
```

## TypeVarTuple
타입 파라미터 앞에 별표(`*`)를 붙이면 `TypeVarTuple`을 선언할 수 있다. 가변 길이의 타입 목록을 표현한다.

```py
def move_first_element_to_last[T, *Ts](
    tup: tuple[T, *Ts]
) -> tuple[*Ts, T]:
    return (*tup[1:], tup[0])

b = move_first_element_to_last((1, "hello", 3.14)) # tuple[str, float, int]
```

## 타입 파라미터 기본값
```py
# Python 3.13 이상
class Box[T = str]:
    def __init__(self, content: T):
        self.content = content

box = Box("hello") # Box[str]
int_box: Box[int] = Box(100)
```

## 요약
| 종류 | 선언 | 설명 |
|---|---|---|
| **TypeVar** | `T` | 하나의 타입 |
| **ParamSpec** | `**P` | 함수 매개변수 목록 전체 |
| **TypeVarTuple** | `*Ts` | 가변 타입 목록 |
| **Bound** | `T: Base` | 타입 상한 |
| **Constraint** | `T: (A, B)` | 허용 타입 후보 |
| **Default** | `T = str` | 기본 타입 인자 |

---
**Reference**
- https://kindloveit.tistory.com/431
