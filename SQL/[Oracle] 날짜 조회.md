# [Oracle] 날짜 조회

### 현재 날짜 시간
```sql
SELECT SYSDATE
FROM dual
```

### 날짜를 문자열로
```sql
SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS')
FROM dual;
```

### 문자열을 날짜로
```sql
SELECT TO_DATE('2025-02-28 09:00:00', 'YYYY-MM-DD HH24:MI:SS')
FROM dual;
```

### 특정 날짜의 데이터 조회
```sql
SELECT * FROM [테이블명]
WHERE [컬럼명] >= TO_DATE('20250201', 'YYYYMMDD')
AND [컬럼명] < TO_DATE('20250201', 'YYYYMMDD') + 1;
```

### 현재 날짜의 데이터 조회
```sql
SELECT * FROM [테이블명]
WHERE [컬럼명] >= TRUNC(SYSDATE)
AND [컬럼명] < TRUNC(SYSDATE) + 1;
```

---
**Reference**<br>
- https://hrjeong.tistory.com/324
