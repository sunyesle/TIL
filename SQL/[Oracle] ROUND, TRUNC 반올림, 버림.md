# [Oracle] ROUND, TRUNC 반올림, 버림
숫자 또는 날짜 값을 반올림/내림하는 함수이다.

## ROUND(number), TRUNC(number)
```sql
ROUND( number [, decimal_places] )
TRUNC( number [, decimal_places] )
```
지정한 자릿수에서 올림(ROUND), 버림(TRUNC)

### 예시
```sql
SELECT TRUNC('123.456', 2)  -- 123.45
     , TRUNC('123.456', 1)  -- 123.4
     , TRUNC('123.456', 0)  -- 123     TRUNC('123.456')와 동일하다.
     , TRUNC('123.456', -1) -- 120
     , TRUNC('123.456', -2) -- 100
FROM DUAL;

SELECT ROUND('123.456', 2)  -- 123.46
     , ROUND('123.456', 1)  -- 123.5
     , ROUND('123.456', 0)  -- 123     ROUND('123.456')와 동일하다.
     , ROUND('123.456', -1) -- 120
     , ROUND('123.456', -2) -- 100
FROM DUAL;
```

## ROUND(date), TRUNC(date)
```sql
ROUND( date [, format] )
TRUNC( date [, format] )
```
지정한 Format으로 날짜를 반올림(ROUND), 내림(TRUNC)

### 예시
```sql
SELECT SYSDATE              -- 2025-09-30 14:48:34
     , TRUNC(SYSDATE, 'MI') -- 2025-09-30 14:48:00
     , TRUNC(SYSDATE, 'HH') -- 2025-09-30 14:00:00
     , TRUNC(SYSDATE, 'DD') -- 2025-09-30 00:00:00  TRUNC(SYSDATE)와 동일하다.
     , TRUNC(SYSDATE, 'MM') -- 2025-09-01 00:00:00
     , TRUNC(SYSDATE, 'YY') -- 2025-09-01 00:00:00
FROM DUAL;

SELECT SYSDATE              -- 2025-09-30 14:48:34
     , ROUND(SYSDATE, 'MI') -- 2025-09-30 14:49:00
     , ROUND(SYSDATE, 'HH') -- 2025-09-30 15:00:00
     , ROUND(SYSDATE, 'DD') -- 2025-10-01 00:00:00  ROUND(SYSDATE)와 동일하다.
     , ROUND(SYSDATE, 'MM') -- 2025-10-01 00:00:00
     , ROUND(SYSDATE, 'YY') -- 2026-01-01 00:00:00
FROM DUAL;
```

---
**Reference**
- http://developer-talk.tistory.com/51
- https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/ROUND-and-TRUNC-Date-Functions.html
