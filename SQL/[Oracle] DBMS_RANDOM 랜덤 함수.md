# [Oracle] DBMS_RANDOM 랜덤 함수
DBMS_RANDOM 패키지는 랜덤한 숫자/문자를 만들어 주는 기능을 제공한다.

## RANDOM
-2의 31제곱 이상 2의 31제곱 미만 구간에서 임의의 정수를 생성한다.

```sql
SELECT DBMS_RANDOM.RANDOM
FROM DUAL;

-- 결과: -823529341
```

## VALUE
`DBMS_RANDOM.VALUE(low, high)`

`low` 이상 `high` 미만 구간에서 임의의 숫자를 생성한다.<br>
범위를 지정하지 않으면 0 이상 1 미만 구간에서 임의의 숫자를 생성한다.
```sql
SELECT DBMS_RANDOM.VALUE(0, 10)
FROM DUAL;

-- 결과: 9.5446315849796399905082112380680939179
```

```sql
SELECT DBMS_RANDOM.VALUE
FROM DUAL;

-- 결과: 0.80527060191912217584545540234795290665
```

**0이상 10미만 자연수 생성**
```sql
SELECT FLOOR(DBMS_RANDOM.VALUE(0, 10))
FROM DUAL;

SELECT FLOOR(DBMS_RANDOM.VALUE * 10)
FROM DUAL;
```

## STRING
`DBMS_RANDOM.STRING(opt, len)`

임의의 문자열을 생성한다.
### 파라미터
- **opt**: 문자열 형태
- **len**: 문자열 길이

| opt      | 설명                    |
|----------|-------------------------|
| `U`, `u` | 대문자 알파벳            |
| `L`, `l` | 소문자 알파벳            |
| `A`, `a` | 대소문자 구분 없는 모든 알파벳 |
| `X`, `x` | 모든 알파벳과 숫자       |
| `P`, `p` | 특수문자를 포함한 모든 문자열 |


```sql
SELECT DBMS_RANDOM.STRING('U', 10) AS STR1,
       DBMS_RANDOM.STRING('L', 10) AS STR2,
       DBMS_RANDOM.STRING('A', 10) AS STR3,
       DBMS_RANDOM.STRING('X', 10) AS STR4,
       DBMS_RANDOM.STRING('P', 10) AS STR5
FROM DUAL;

- 결과: FBGDWULMTH    zogqcohncm    okKyVoyebo    7A69L7YYE3    $q"aEgON{0
```

## NORMAL
표준정규분포(가우스 분포)를 따르는 임의의 숫자를 생성한다.
```sql
SELECT DBMS_RANDOM.NORMAL
FROM DUAL;

- 결과: 0.4690737477334749859620918323574848482177
```

---
**Reference**<br>
- https://docs.oracle.com/database/timesten-18.1/TTPLP/d_random.htm#TTPLP040
- https://javabuilders.tistory.com/83
