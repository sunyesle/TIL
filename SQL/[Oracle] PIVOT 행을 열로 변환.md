# [Oracle] PIVOT 행을 열로 변환

## PIVOT 문법
```sql
SELECT *
FROM ( 피벗 대상 쿼리문 )
PIVOT ( 그룹합수(집계컬럼) FOR 피벗컬럼 IN (피벗컬럼값 AS 별칭 ... )
```

## 예시
**직책별 부서별 평균급여 구하기**

### group by 사용
각각의 직책별 부서별 평균 급여를 구해보자.
```sql
SELECT job, deptno, AVG(sal)
FROM emp
GROUP BY job, deptno
```

간단하게 출력되었으나 여러 행으로 출력되어 보기 불편하다.

![group by](https://github.com/user-attachments/assets/046794b4-56ac-4fe4-b313-96aae7861f43)

### pivot 사용
세로(ROW)로 출력된 `deptno`들을 가로(COLUMN)로 나열될 수 있도록 변경해 보자.
```sql
SELECT *
FROM (
    SELECT job, deptno, sal
    FROM emp
)
pivot (
    AVG(sal)
    FOR deptno IN ('10', '20', '30')
)
```

집계함수에 사용되는 열(`sal`)을 제외한 나머지 열들(`deptno`, `job`)을 대상으로 GROUP BY 된다.<br>
이후 PIVOT 대상인 `job` 값을 기준으로 가로로 나열된다.

![pivot](https://github.com/user-attachments/assets/56f85cc5-48c3-4cb4-bbfa-9257002cde37)

---
**Reference**<br>
- https://dololak.tistory.com/811
- https://gent.tistory.com/42
- https://m.blog.naver.com/regenesis90/222205833002

