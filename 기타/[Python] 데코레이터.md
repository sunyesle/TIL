# [Python] 데코레이터(Decorator)
**데코레이터**는 기존 함수를 수정하지 않고, 해당 함수의 동작을 확장하거나 변경할 수 있도록 하는 기능이다.

일반적으로 기존 함수를 인자로 받아 래퍼 함수를 반환하며, `@` 문법을 사용하여 적용할 수 있다.

```py
def trace(func):                         # 호출할 함수를 매개변수로 받는다.
    def wrapper():                       # 호출할 함수를 감싸는 함수
        print(func.__name__, '함수 시작')
        func()                           # 매개변수로 받은 함수를 호출한다.
        print(func.__name__, '함수 끝')
    return wrapper                       # wrapper 함수 반환


def hello():
    print('hello')

trace_hello = trace(hello) # 데코레이터에 호출할 함수를 넣는다.
trace_hello()              # 반환된 함수를 호출한다.


# @를 사용하여 데코레이터를 간편하게 사용할 수 있다.
@trace
def world():
    print('world')

world()
```

여러 개의 데코레이터를 지정하면 아래에서 위 순서로 적용된다. 함수를 호출하면 바깥쪽 래퍼부터 안쪽 래퍼 순으로 실행된다.
```py
def decorator1(func):
    def wrapper():
        print('decorator1')
        func()
    return wrapper

def decorator2(func):
    def wrapper():
        print('decorator2')
        func()
    return wrapper


# decorator1(decorator2(hello)) 처럼 동작한다.
@decorator1
@decorator2
def hello():
    print('hello')

hello()
# [실행 결과]
# decorator1
# decorator2
# hello
```

## 예시
실행 시간 측정 데코레이터
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

- `Callable[P, R]`: 기존 함수의 매개변수와 반환 타입을 유지하도록 타입을 표현한다.
- `@wraps(func)`: 기존 함수의 이름, 독스트링 등의 메타데이터를 보존한다.

---
**Reference**
- https://dojang.io/mod/page/view.php?id=2427
