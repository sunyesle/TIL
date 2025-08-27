# [Oracle] MERGE INTO 조건에 따라 INSERT, UPDATE, DELETE 처리
MERGE 문은 조건에 따라 INSERT, UPDATE, DELETE를 한 번에 수행할 수 있다.

```sql
MERGE INTO target_table t
USING source_table s
ON (t.id = s.id)
WHEN MATCHED THEN
     UPDATE SET t.name = s.name
WHEN NOT MATCHED THEN
     INSERT (t.id, t.name)
     VALUES (s.id, s.name);
```
- `target_table`: 대상 테이블 (INSERT, UPDATE, DELETE를 수행할 테이블)
- `source_table`: 소스 테이블

ON 절은 대상 테이블과 소스 테이블 간의 일치 여부를 확인하기 위한 조인 조건을 지정하는 부분이다.<br>
ON 절에서 사용한 컬럼을 UPDATE 하는 경우 ORA-38104 오류가 발생한다.

## 단일 테이블
```sql
MERGE INTO emp e
USING dual
ON (e.empno = 7788)
WHEN MATCHED THEN
    UPDATE SET e.deptno = 20
WHEN NOT MATCHED THEN
    INSERT (e.empno, e.ename, e.deptno)
    VALUES (7788, 'SCOTT', 20);
```

## 조인 사용
```sql
MERGE INTO emp m
USING emp_tmp t
ON (m.empno = t.empno)
WHEN MATCHED THEN
     UPDATE SET m.ename = t.ename, m.deptno = t.deptno
WHEN NOT MATCHED THEN
     INSERT (m.empno, m.ename, m.deptno)
     VALUES (t.empno, t.ename, t.deptno);
```

## 인라인뷰(서브쿼리) 사용
```sql
MERGE INTO emp_bonus b
USING (
    SELECT e.empno
    FROM emp e, dept d
    WHERE e.deptno = d.deptno
    AND d.deptno = 10
) s
ON (b.empno = s.empno)
WHEN MATCHED THEN
    UPDATE SET b.bonus = b.bonus + 100
WHEN NOT MATCHED THEN
    INSERT (b.empno, b.bonus)
    VALUES (s.empno, 100);
```

## WHERE 사용
```sql
MERGE INTO emp e
USING dual
ON (e.empno = 7788)
WHEN MATCHED THEN
    UPDATE SET e.deptno = 20
    WHERE e.job = 'ANALYST';
```

## DELETE 사용
```sql
MERGE INTO emp e
USING dual
ON (e.empno = 7788)
WHEN MATCHED THEN
    UPDATE SET e.deptno = 20
    WHERE e.job = 'ANALYST'
    DELETE
    WHERE e.job != 'ANALYST';
```

---
**Reference**<br>
- https://gent.tistory.com/406
- https://offbyone.tistory.com/253
- https://drg2524.tistory.com/182
