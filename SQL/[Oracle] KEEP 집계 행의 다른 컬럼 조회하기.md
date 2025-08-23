# [Oracle] KEEP 집계 행의 다른 컬럼 조회하기
KEEP 키워드는 특정 컬럼을 기준으로 정렬하여 최댓값 또는 최솟값을 갖는 행을 찾고, 그 행의 다른 컬럼 값을 조회할 때 사용된다.

## 사용법
```sql
[집계함수]([집계컬럼]) KEEP(DENSE_RANK [LAST/FIRST] ORDER BY [정렬컬럼]  [ASC/DESC])

MAX(sal) KEEP(DENSE_RANK LAST ORDER BY sal)
```

## 예시
예시와 함께 KEEP 키워드에 대해 알아보자. 예를들어,

**직군별 최고 연봉은?** <br>
직군별로 GROUP BY 후 MAX를 사용해 조회할 수 있다.

**직군별 최고 연봉인 사원의 이름은?** <br>
이런 경우 정렬 기준과 필요한 컬럼이 다르므로 단순히 GROUP BY와 MAX로 값을 조회할 수 없다.
최고 연봉인 사람을 추출한 다음 이름을 찾아야 해서 서브쿼리나 다른 방법으로 쿼리를 해야 한다.

```sql
SELECT e.job
     , e.ename
     , e.sal
  FROM (
        SELECT job
             , ename
             , sal
             , ROW_NUMBER() OVER (PARTITION BY job 
                                  ORDER BY sal DESC, ename ASC) AS RNUM
          FROM EMP
       ) e
WHERE RNUM = 1
```

KEEP 키워드를 사용하면 다음과 같이 간결하게 작성할 수 있다.
```sql
SELECT job
     , MAX(sal)  AS top_sal,
     , MIN(ename) KEEP (DENSE_RANK FIRST ORDER BY sal DESC) AS top_ename
FROM emp
GROUP BY job
```
- `MAX(sal)` : 최고 연봉
- `KEEP`+`MIN(ename)` : 그 최고 연봉을 가진 사원 중 가장 앞에 오는 이름

`KEEP (DENSE_RANK FIRST ORDER BY sal DESC)` 부분에서 최고 연봉을 가진 행들을 먼저 선택한다.<br>
이후, 집계함수 `MIN(ename)` 이 적용되어 사전 순으로 가장 앞에 오는 이름을 반환하게 된다.

순위가 중복되는 경우 의도하지 않은 값이 표시될 수 있으므로, 정렬 조건을 세분화하여 순위가 중복되지 않게 하는 것이 좋다.

### OVER 절과 함께 사용
OVER 절과 함께 사용하면 GROUP BY 절 없이도 사용할 수 있다.
```sql
SELECT ename
     , job
     , sal
     , MAX(sal) KEEP(DENSE_RANK LAST ORDER BY sal) OVER(PARTITION BY job) AS sal_last
     , MAX(ename) KEEP(DENSE_RANK LAST ORDER BY sal) OVER(PARTITION BY job) AS ename_last
FROM emp
WHERE job IN ('MANAGER', 'SALESMAN')
```

---
**Reference**<br>
- https://gent.tistory.com/475
