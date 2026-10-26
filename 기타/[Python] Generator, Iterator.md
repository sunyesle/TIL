# [Python] Generator, Iterator

## 이터레이터(Iterator)
**이터레이터**는 `__iter__()`와 `__next__()`라는 메서드를 구현하여 값을 순차적으로 꺼낼 수 있도록 하는 객체이다.

- `__iter__()`: 이터레이터 객체 자신을 반환한다.
- `__next__()`: 다음 값을 반환한다. 더 이상 반환할 값이 없는 경우 `StopIteration` 예외를 발생시킨다.

```py
class MyIterator:
    def __init__(self, data):
        self.data = data
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        value = self.data[self.index]
        self.index += 1
        return value

my_iter = MyIterator([1, 2, 3])
for item in my_iter:
    print(item)
```

### 장단점
- **장점**:
   - **클래스 기반의 유연성**: 이터레이터는 클래스로 구현되므로, 복잡한 로직을 가진 이터레이터를 만들 수 있습니다. 예를 들어, 특정 조건에 따라 값을 필터링하거나 변환하는 이터레이터를 만들 수 있다.
   - **상태 유지**: 이터레이터는 현재 상태(예: `self.index`)를 유지하므로, 순회 중에 필요한 정보를 저장하고 사용할 수 있다.
- **단점**:
   - **코드 복잡성**: `__iter__()`와 `__next__()` 메서드를 모두 구현해야 하므로 코드가 상대적으로 복잡해진다. 간단한 이터레이터를 만들기 위해서는 보일러플레이트 코드(boilerplate code)가 많아질 수 있다.

## 제너레이터(Generator)
**제너레이터**는 이터레이터의 한 종류로, `yield`를 사용하여 값을 순차적으로 제공하는 이터레이터를 간편하게 구현할 수 있도록 하는 기능이다.

제너레이터 함수는 일반 함수처럼 정의되지만, `yield`문을 사용하여 값을 반환하고 실행을 일시 중단한다.
이후 다음 값을 요청하면 중단된 지점부터 실행을 재개한다.

```py
def my_generator(data):
    for item in data:
        yield item

gen = my_generator([1, 2, 3])
for item in gen:
    print(item)
```

컴프리헨션과 유사한 문법으로도 생성할 수 있으며, 이를 **제너레이터 표현식**(Generator Expression)이라고 한다.
```py
gen = (x+1 for x in range(3))
for value in gen:
    print(value)
```

### 장단점
- **장점**
   - **간결성**: `yield`를 사용하여 이터레이터를 간결하게 구현할 수 있다.
   - **자동 상태 관리**: 제너레이터는 자동으로 함수의 상태(지역 변수, 실행 위치 등)를 저장하고 복원하므로, 개발자가 직접 상태를 관리할 필요가 없다.
   - **메모리 효율성**: 값을 미리 생성하여 메모리에 저장하지 않고, 필요할 때마다 하나씩 생성하므로 메모리 사용량이 적다. 특히 큰 데이터 시퀀스를 처리할 때 유용하다.
   - **지연 평가**(Lazy Evaluation): 값이 실제로 필요할 때까지 계산을 미루므로, 불필요한 계산을 줄이고 성능을 향상시킬 수 있다.
- **단점**:
   - **재사용 제약**: 소비가 완료된 제너레이터 객체는 다시 순회할 수 없다. 다시 순회하려면 새로운 제너레이터 객체를 생성해야 한다.
   - **접근 방식의 제약**: 리스트처럼 인덱스를 통한 임의 접근이나 이미 소비한 값의 재조회가 어렵다.

---
**Reference**
- https://bcho.tistory.com/1461
- https://wikidocs.net/22802
