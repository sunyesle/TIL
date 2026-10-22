# [Python] 삼항 연산자
```
참일_때_값 if 조건 else 거짓일_때_값
```

## 예시
```
result = cursor.first()._asdict() if cursor.rowcount > 0 else None

# 다음과 동일하다.
if cursor.rowcount > 0:
    result = cursor.first()._asdict()
else:
    result = None
```
