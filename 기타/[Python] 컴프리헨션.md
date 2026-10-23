# [Python] 컴프리헨션

## 리스트 컴프리헨션
대괄호(`[]`)내에 다음과 같이 `for`문을 작성한다.
```py
[ x for x in range(10) ]
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# 다음과 동일하다.
numbers = []
for n in range(0, 10):
    numbers.append(n)
```

### 조건
```py
[ x for x in range(1, 10+1) if x % 2 == 0 ]
# [2, 4, 6, 8, 10]
```

### 중첩 for
```py
[ (x, y) for x in ['쌈밥', '치킨', '피자'] for y in ['사과', '아이스크림'] ]
# [('쌈밥', '사과'),
#  ('쌈밥', '아이스크림'),
#  ('치킨', '사과'),
#  ('치킨', '아이스크림'),
#  ('피자', '사과'),
#  ('피자', '아이스크림')]

results = []
for x in ['쌈밥', '치킨', '피자']:
    for y in ['사과', '아이스크림']:
        results.append((x, y))
```

### 복수 조건
```py
[ x for x in range(10) if x < 5 if x % 2 == 0 ]
# [0, 2, 4]
```

## 딕셔너리 컴프리헨션
중괄호(`{}`)를 이용하면 `set comprehension`이 된다.
```py
{ x+y for x in range(10) for y in range(10) }
# {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18}
```

중괄호를 이용하면서, 동시에 `key: value` 의 형태로 내용을 채운다면 `dictionary comprehension`이 된다.
```py
students = ['철수', '영희', '길동', '순희']
{ student: 0 for student in students }
# {'철수': 0, '영희': 0, '길동': 0, '순희': 0}
```

기존 `dict` 데이터를 변경할 수도 있다.
```py
scores = {'철수': 50, '영희': 80, '길동': 90, '순희': 60, '전학생': 100}

# [전학간 학생 성적 제거]
scores = { name: score for name, score in scores.items() if name != '전학생' }
# {'길동': 90, '순희': 60, '영희': 80, '철수': 50}

# [통과, 불통과]
grades = { name: 'PASS' if value > 70 else 'No-PASS' for name, value in scores.items()}
# {'길동': 'PASS', '순희': 'No-PASS', '영희': 'PASS', '철수': 'No-PASS'}
```

---
**Reference**
- https://wikidocs.net/22805
