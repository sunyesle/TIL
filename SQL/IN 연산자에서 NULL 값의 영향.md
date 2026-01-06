# IN 연산자에서 NULL 값의 영향
SQL에서 `NULL`은 값이 아니라 **`UNKNOWN`** 상태를 의미한다.
따라서 어떤 값과 비교해도 결과는 `TRUE`나 `FALSE`가 아닌 `UNKNOWN`이 된다.

`IN`이나 `NOT IN` 연산자에 `NULL`을 포함할 경우 **예상치 못한 결과가 나올 수 있으므로 주의해야 한다.**

`NULL`값 비교가 필요할 때는 반드시 `IS NULL` 또는 `IS NOT NULL`을 사용해야 한다.

> **샘플 데이터**
> ```sql
> CREATE TABLE emp (
>     empno  INT,
>     ename  VARCHAR(10),
>     deptno INT
> );
> 
> INSERT INTO emp VALUES (7001, 'SMITH',   10);
> INSERT INTO emp VALUES (7002, 'ALLEN',   20);
> INSERT INTO emp VALUES (7003, 'WARD' ,   30);
> INSERT INTO emp VALUES (8001, 'KIM'  , NULL);
> INSERT INTO emp VALUES (8002, 'PARK' , NULL);
> ```

## IN 연산자와 NULL
```sql
SELECT *
FROM emp
WHERE deptno IN (10, 20, NULL);
```
> **결과**: deptno가 10, 20인 직원만 조회되고, `NULL`인 직원은 조회되지 않는다.

IN 연산자는 내부적으로 다음과 같이 동작한다.
- `deptno IN (10, 20, NULL)`
- → `deptno = 10 OR deptno = 20 OR deptno = NULL`

`deptno = NULL`의 결과는 `TRUE`가 아닌 `UNKNOWN`이므로 조회되지 않는다.

### 수정된 쿼리
`IN (..., NULL)`은 무의미하며, `NULL`인 행을 찾으려면 `OR deptno IS NULL`을 붙여야 한다.
```sql
SELECT *
FROM emp;
WHERE deptno IN (10, 20) OR deptno IS NULL;
```

## NOT IN 연산자와 NULL
```sql
SELECT *
FROM emp
WHERE deptno NOT IN (10, 20, NULL);
```
> **결과**: 아무런 데이터도 조회되지 않는다.

NOT IN 연산자는 내부적으로 다음과 같이 동작한다.
- `deptno NOT IN (10, 20, NULL)`
- → `deptno != 10 AND deptno != 20 AND deptno != NULL`

`deptno != NULL`의 결과는 `UNKNOWN`이므로 `AND` 연산은 전체가 `TRUE`가 될 수 없게되고, 모든 행이 조건에서 탈락하게 된다.

### 수정된 쿼리
`NOT IN (..., NULL)`은 치명적이며, 결과를 증발시킨다.
`NOT IN`에서 서브쿼리 사용 시 `NULL` 데이터가 포함되지 않도록 주의해야 한다.
`NOT IN`에서 `NULL`인 행은 항상 제외되므로 `AND deptno IS NOT NULL`은 추가하지 않아도 된다.
```sql
SELECT *
FROM emp
WHERE deptno NOT IN (10, 20);
```
