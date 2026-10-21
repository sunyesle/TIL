# [Python] 매직 메서드
메서드 중에 `__`로 시작해서 `__`로 끝나는 메서드들이 있는데, 이를 **매직 메서드** 또는 **특별 메서드**(special method)라고 한다.

## __init__
객체가 생성된 후 초기화할 때 호출
```py
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

## __call__
객체를 함수처럼 `()` 연산자로 호출할 때 실행
```py
class MyFunc:
    def __call__(self, *args, **kwargs):
        print("호출됨")

f = MyFunc()
f()
```

## __getattribute__
`.` 연산자 등을 통해 객체의 속성에 접근할 때 호출
```py
class Stock:
    def __getattribute__(self, item):
        print(item, "객체에 접근하셨습니다.")

s = Stock()
s.data
```

## __str__
사용자 친화적인 문자열 반환
```py
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
 
    def __str__(self):
        return f'{self.name}은(는) {self.price} 달러입니다.'
 
iPhone = Product('iPhone', 999)
print(str(iPhone)) # iPhone은(는) 999 달러입니다.
```

## __ref__
객체 디버깅 또는 로깅을 위한 문자열 반환
```py
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
 
    def __repr__(self):
        return f'Product(name={self.name}, price={self.price})'
 
iPhone = Product('iPhone', 999)
print(repr(iPhone)) # Product(name=iPhone, price=999)
```

---
**Reference**
- https://wikidocs.net/83755
- https://docs.kanaries.net/ko/topics/Python/python-str-vs-repr
