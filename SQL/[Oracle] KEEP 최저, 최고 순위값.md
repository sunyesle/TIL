# [Oracle] KEEP 최저, 최고 순위값
KEEP 키워드를 사용하면 행 그룹(GROUP BY) 내애세 최저/최고 순위 행으로 집계를 수행할 수 있다.

특정 컬럼을 기준으로 정렬하여 최대/최소값을 같는 행을 찾고, 그 행의 다른 컬럼을 출력하고 싶을 때 사용하는 함수이다.

## 사용법
```sql
[집계함수]([집계컬럼]) KEEP(DENSE_RANK [LAST/FIRST] ORDER BY [정렬컬럼] [ASC/DESC])

MAX(sal) KEEP(DENSE_RANK LAST ORDER BY sal)
```

## 예시
예를들어

**직군별 최고 연봉은?** <br>
직군별로 GROUP BY 후 MAX를 사용해 조회할 수 있다.

**직군별 최고 연봉인 사원의 이름은?** <br>
이런경우 정렬 기준과 필요한 컬럼이 다르기 때문에 단순히 GROUP BY와 MAX로 값을 조회할 수 없다.
최고 연봉인 사람을 추출 한 다음 이름을 찾아야해서 서브쿼리나 다른 방법으로 쿼리를 해야한다.

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
SELECT job,
       MIN(ename) KEEP (DENSE_RANK FIRST ORDER BY sal DESC) AS top_ename,
       MAX(sal)  AS top_sal
FROM emp
GROUP BY job
```
직군(job) 별로 DENSE_RANK를 사용하여 연봉(sal) 내림차순으로 정렬 후 순위를 부여하고 첫번째 순위(FIRST)인 직원의 사원명(ename)을 출력한다.

만약, 최고 연봉인 사원이 여러명인 경우 집계함수에 의해서 MIN 값이 표시된다.<br>
순위가 중복되는 경우 의도하지 않은 값이 표시될 수 있으므로, 정렬 조건을 세분화하여 순위가 중복되지 않게 하는 것이 좋다.

---
**Reference**<br>
- https://gent.tistory.com/475
