# [Oracle] LNNVL 함수 null 처리 + 조건문
LNNVL 함수는 해당 컬럼에 NULL이 존재할 경우 NULL 처리와 조건문을 한 번에 연산하기 위해서 사용한다.

주로 쿼리의 WHERE절이나 CASE 표현식의 WHEN 조건으로 사용된다.

NULL이거나 조건이 FALSE인 경우 TRUE를 반환한다.
- 컬럼이 NULL인 경우 = TRUE
- 함수 내부 조건이 FALSE인 경우 = TRUE

## 예시
### 예시 1
comm이 300 이하인 직원을 조회하고 싶다면 다음과 같이 쿼리를 작성할 수 있다.
```sql
SELECT ename
     , job
     , comm
FROM emp
WHERE deptno = 30
AND comm <= 300
```

comm이 NULL인 직원도 포함시키고 싶다면 LNNVL 함수를 사용하여 쿼리를 작성할 수 있다.
```sql
SELECT ename
     , job
     , comm
FROM emp
WHERE deptno = 30
AND LNNVL(comm > 300)
```
아래와 같이 2가지 조건을 선언한 것과 동일하다.
```sql
SELECT ename
     , job
     , comm
FROM emp
WHERE deptno = 30
AND (comm is not null OR comm <= 300)
```

### 예시 2
comm이 NULL이거나 0이 아닌 직원을 조회하는 쿼리이다.
```sql
SELECT ename
     , job
     , comm
FROM emp
WHERE deptno = 30
AND LNNVL(comm = 0)
```

```sql
SELECT ename
     , job
     , comm
FROM emp
WHERE deptno = 30
AND (comm is null OR comm != 0)
```

---
**Reference**<br>
- https://bestwizard.tistory.com/714
- https://gent.tistory.com/387
