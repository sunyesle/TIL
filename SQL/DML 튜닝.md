# DML 튜닝
## DML 성능에 영향을 미치는 요소
- 인덱스
- 무결성 제약
- 조건절
- 서브쿼리
- Redo 로깅
- Undo 로깅
- Lock
- 커밋

### 인덱스
인덱스는 정렬된 자료구조이므로 데이터를 수정할 때, 수직적 탐색을 통해 삭제/삽입할 블록을 찾아야 한다.
인덱스의 개수가 DML 성능에 미치는 영향이 매우 큰 만큼, 인덱스 설계에 심혈을 기울여야 한다.

### 무결성 제약
DBMS에서 PK, FK, Check, Not Null같은 제약을 설정하여 데이터 무결성을 지켜낼 수 있다.
PK, FK 제약은 실제 데이터를 조회해 봐야 하기 때문에 Check, Not Null 제약보다 성능에 더 큰 영향을 미친다.

### 조건절
SELECT 문과 실행계획이 다르지 않으므로 인덱스 튜닝 원리를 그대로 적용할 수 있다.
```sql
update emp set sal = sal * 1.1 where deptno = 40;
```

### 서브쿼리
SELECT 문과 실행계획이 다르지 않으므로 조인튜닝 원리를 그대로 적용할 수 있다.
```sql
update emp e set sal = sal * 1.1
where exists (select 'x' from dept where e.deptno and loc = 'CHICAGO');
```

### Redo 로깅
오라클은 데이터 파일과 컨트롤 파일에 가해지는 모든 변경 사항을 Redo 로그에 기록한다.
과거를 현재 상태로 되돌리는 데 사용한다. 트랜잭션을 재현하는 데 필요한 정보를 로깅 한다.

DML을 수행할 때마다 Redo 로그를 생성해야 하므로 성능에 영향을 미친다. INSERT 작업에 대해 Redo 로깅 생략 기능을 제공하는 이유가 여기에 있다.

> **Redo 로그의 용도**
> 
> - **Database Recovery**: 물리적으로 디스크가 깨지는 등의 Media Fail 발생 시 데이터베이스를 복구하기 위해 사용된다.
> - **Cache Recovery**: 버퍼 캐시에 저장된 변경 사항이 디스크 상의 데이터 블록에 기록되지 않은 상태에서 정전 등으로 인해 유실되었을 때(Instance Crash) 복구하기 위해 사용된다.
> - **Fast Commit**: 변경 사항을 우선 Append 방식으로 빠르게 로그 파일에 기록하여 커밋을 완료하고, 디스크상의 블록에 반영하는 작업은 나중에 배치 방식으로 일괄 수행한다.

### Undo 로깅
트랜잭션을 롤백함으로써 현재를 과거 상태로 되돌리는 데 사용한다. 변경된 블록을 이전 상태로 되돌리는 데 필요한 정보를 로깅 한다.

DML을 수행할 때마다 Undo 로그를 생성해야 하므로 성능에 영향을 미친다. 하지만 오라클은 Undo 로그를 남기지 않는 방법은 제공하지 않는다.

> **Undo 로그의 용도**
> 
> 오라클은 데이터를 입력, 수정, 삭제할 때마다 Undo 세그먼트에 기록을 남긴다.<br>
> Undo 데이터를 기록한 공간은 해당 트랜잭션이 커밋하는 순간 다른 트랜잭션이 재사용할 수 있는 상태로 바뀌며, 가장 오래전에 커밋한 Undo 공간부터 재사용한다.
> 
> - **Transaction Rollback**: 트랜잭션에 의한 변경 사항을 최종 커밋하지 않고 롤백하고자 할 때 사용된다.
> - **Transaction Recovery**: Instance Crash 발생 후 Redo 로그를 이용해 최종 커밋되지 않은 사항까지 모두 복구된다. 이후 최종 커밋되지 않았던 트랜잭션을 롤백하는 데 Undo 로그가 사용된다.
> - **Read Consistency**: 읽기 일관성을 위해 사용된다.

> **MVCC(Multi-Version Concurrency Control) 모델**
> 
> MVCC 모델을 사용하는 오라클은 데이터를 두 가지 모드로 읽는다.
> - **Current 모드**: 디스크에서 캐시로 적재된 원본 블록을 현재 상태 그대로 읽는 방식.
> - **Consistent 모드**: 쿼리가 시작된 이후에 다른 트랜잭션에 의해 변경된 블록을 만나면 원본 블록으로부터 복사본(CR Copy) 블록을 만들고, 거기에 Undo 데이터를 적용해 쿼리가 시작된 시점으로 되돌려서 읽는 방식.
> 
> SELECT 문은 (몇몇 예외 케이스를 제외하고는) 항상 Consistent 모드로 데이터를 읽는다.<br>
> DML 문은 Consistent 모드로 대상 레코드를 찾고, Current 모드로 원본 블록을 찾아서 갱신한다.

### 커밋
커밋은 DML과 별개로 실행하지만, DML을 끝내려면 커밋까지 완료해야 하므로 서로 밀접한 관련이 있다.

Fast Commit의 도움으로 커밋을 빠르게 처리하기는 하지만 결코 가벼운 작업이 아니다.
Redo 로그도 파일이다. Append 방식으로 기록하더라도 디스크 I/O는 느리다.

너무 자주 커밋하는 경우 프로그램 성능이 저하될 수 있다.
그렇다고 오랫동안 커밋하지 않은 채 데이터를 계속 갱신하면 Undo 공간이 부족해져 시스템에 장애 상황을 유발할 수 있다.

트랜잭션을 논리적으로 잘 정의함으로써 불필요한 커밋이 발생하지 않도록 구현해야 한다.

## 데이터베이스 Call과 성능
Call의 발생 위치에 따라 다음과 같이 종류를 나눌 수 있다.
- **User Call**: 네트워크를 경유해 DBMS 외부로부터 인입되는 Call이다.
- **Recursive Call**: DBMS 내부에서 발생하는 Call이다. SQL 파싱과 최적화 과정에서 발생하는 데이터 딕셔너리 조회, 사용자 정의 함수/프로시저/트리거에 내장된 SQL을 실행할 때 발생하는 Call이 여기 해당된다.

Call이 많으면 성능은 느릴 수밖에 없다. 특히, 네트워크를 경유하는 User Call이 성능에 미치는 영향은 매우 크다.

### One SQL의 중요성
업무 로직이 복잡하지 않다면, 한 번의 Call로 처리할 수 있도록 One SQL로 구현하려고 노력해야 한다.
- Insert Into Select
- 수정가능 조인 뷰
- Merge 문

### Array Processing
업무 로직이 복잡하여 One SQL로 구현하는게 힘들다면, JDBC의 addBatch/executeBatch 를 활용하여 Call 부하를 줄일 수 있다.

## 수정가능 조인 뷰
### 전통적인 UPDATE 문
```sql
update 고객 c
set 최종거래일시 = (select max(거래일시) from 거래
                  where 고객번호 = c.고객번호
                  and 거래일시 >= trunc(add_months(sysdate, -1)))
  , 최근거래횟수 = (select count(*) from 거래
                  where 고객번호 = c.고객번호
                  and 거래일시 >= trunc(add_months(sysdate, -1)))
  , 최근거래금액 = (select sum(거래금액) from 거래
                  where 고객번호 = c.고객번호
                  and 거래일시 >= trunc(add_months(sysdate, -1)))
where exists (select 'x' from 거래
              where 고객번호 = c.고객번호
              and 거래일시 >= trunc(add_months(sysdate, -1)))
```

다음과 같이 고칠 수 있다. 이전보다 개선되었지만 한 달 이내 고객별 거래 데이터를 두 번 조회하는 비효율은 존재한다.
총 고객 수와 한 달 이내 거래 고객 수에 따라 성능이 좌우된다.
```sql
update 고객 c
set (최종거래일시, 최근거래횟수, 최근거래금액) = 
    (select max(거래일시), count(*), sum(거래금액)
     from 거래
     where 고객번호 = c.고객번호
     and 거래일시 >= trunc(add_months(sysdate, -1)))
where exists (select 'x' from 거래
              where 고객번호 = c.고객번호
              and 거래일시 >= trunc(add_months(sysdate, -1)))
```

만약 총 고객 수가 아주 많다면 다음과 같이 해시 세미 조인으로 유도하는 것을 고려할 수 있다.
```sql
update 고객 c
set (최종거래일시, 최근거래횟수, 최근거래금액) = 
    (select max(거래일시), count(*), sum(거래금액)
     from 거래
     where 고객번호 = c.고객번호
     and 거래일시 >= trunc(add_months(sysdate, -1)))
where exists (select /*+ unest hash_sj */ 'x' from 거래
              where 고객번호 = c.고객번호
              and 거래일시 >= trunc(add_months(sysdate, -1)))
```

만약 한 달 이내에 거래를 발생시킨 고객이 많아 Update 발생량이 많다면 다음과 같이 변경하는 것을 고려할 수 있다. 하지만 모든 고객 레코드에 Lock이 걸리는 것은 물론, 같은 값으로 갱신되는 비중이 높을수록 Redo 로그 발생량이 증가해 오히려 비효율적일 수 있다.
```sql
update 고객 c
set (최종거래일시, 최근거래횟수, 최근거래금액) = 
    (select nvl(max(거래일시), c.최종거래일시)
          , decode(count(*), 0, 최근거래횟수, count(*))
          , nvl(sum(거래금액), c.최근거래금액)
     from 거래
     where 고객번호 = c.고객번호
     and 거래일시 >= trunc(add_months(sysdate, -1)))
```
이처럼 다른 테이블과 조인이 필요할 때 전통적인 UPDATE 문을 사용하면 비효율을 완전히 해소할 수 없다.

### 수정가능 조인 뷰
수정가능 조인 뷰를 활용하면 참조 테이블과 두 번 조인하는 비효율을 없앨 수 있다.
```sql
update (
  select /*+ ordered use_hash(c) no_merge(t) */
         c.최종거래일시, c.최근거래횟수, c.최근거래금액
         t.거래일시, t.거래횟수, t.거래금액
  from (select 고객
             , max(거래일시) 거래일시, count(*) 거래횟수, sum(거래금액) 거래금액
        from 거래 t
        where 거래일시 >= trunc(add_months(sysdate, -1))
        group by 고객) t
      , 고객 c
  where c.고객번호 = t.고객번호
) 
set 최종거래일시 = 거래일시
  , 최근거래횟수 = 거래횟수
  , 최근거래금액 = 거래금액
```
**조인 뷰**는 FROM 절에 두개 이상 테이블을 가진 뷰를 말한다.<br>
**수정가능 조인 뷰**는 입력, 수정, 삭제가 허용되는 조인 뷰를 말한다.

1쪽 집합과 조인하는 M쪽 집합에만 입력, 수정, 삭제가 허용된다.<br>
1쪽 집합에 PK 제약을 설정하거나 Unique 인덱스를 생성해야 수정가능 조인 뷰를 통한 입력, 수정, 삭제가 가능하다.

예를 들어 DEPT와 EMP는 1:M 관계이며, DEPT.deptno는 EMP.deptno의 외래 키로 사용된다.<br>
EMP 테이블의 데이터만 수정 가능하며, DEPT.deptno에 PK 제약 또는 Unique 인덱스가 필요하다.

#### ORA-01779 오류 회피
```sql
update (
  select d.deptno, d.avg_sal d_avg_sal, e.avg_sal e_avg_sal
  from (select deptno, round(avg(sal), 2) avg_sal from emp group by deptno) e
     , dept d
  where d.deptno = e.deptno
)
set d_avg_sal = e_avg_sal
```
11g 이하 버전에서는 위 UPDATE 문을 실행하면 ORA-01779 오류가 발생한다.<br>
EMP 테이블을 DEPTNO로 Group By 했으므로 DEPTNO 컬럼으로 조인한 DEPT 테이블은 키가 보존되는데도 옵티마이저가 불필요한 제약을 가한 것이다.

10g 버전에서는 `/*+ bypass_ujvc */`힌트를 사용해 오류를 회피할 수 있다.<br>
11g 버전에서는 이 힌트를 사용할 수 없게 되어서 위 UPDATE 문을 실행할 방법이 없다.<br>
12c 버전부터는 수정가능 조인 뷰 기능이 개선되어 힌트를 사용하지 않아도 오류없이 UPDATE문이 실행된다.

예를 들어, 고객_t 테이블에 Unique 인덱스가 없는 경우 아래 쿼리를 실행할 수 없지만
```sql
update ( 
  select o.주문금액, o.할인금액, c.고객등급
  from 주문_t o, 고객_t c
  where o.고객번호 = c.고객번호
  and o.주문금액 >= 1000000
  and c.고객등급 = 'A'
)
set 할인금액 = 주문금액 * 0.2
  , 주문금액 = 주문금액 * 0.8
```
12c에서는 고객_t 테이블을 Group By 처리하여 ORA-01779 에러를 회피할 수 있다.
```sql
update (
     select o.주문금액, o.할인금액, c.고객등급
     from (select 고객번호, 고객등급 고객_t where 고객등급 = 'A' group by 고객번호) c
        , 주문_t o
     where o.고객번호 = c.고객번호
     and o.주문금액 >= 1000000
)
set 할인금액 = 주문금액 * 0.2
  , 주문금액 = 주문금액 * 0.8
```
배치 프로그램 등에서 사용하는 중간 임시 테이블에는 일일이 PK 제약이나 인덱스를 생성하지 않으므로 이 패턴이 유용할 수 있다.

## MERGE문 활용
조건에 따라 INSERT와 UPDATE를 선택적으로 처리할 수 있다.

다음과 같이 수정가능 조인 뷰 기능을 대체할 수 있게 되었다.
```sql
-- 수정가능 조인 뷰
update (
  select d.deptno, d.avg_sal d_avg_sal, e.avg_sal e_avg_sal
  from (select deptno, round(avg(sal), 2) avg_sal from emp group by deptno) e
     , dept d
  where d.deptno = e.deptno
)
set d_avg_sal = e_avg_sal

-- Merge 문
merge into dept d
using (select deptno, round(avg(sal), 2) avg_sal from emp group by deptno) e
on (d.deptno = e.deptno)
when matched then
  update set d.avg_sal = e.avg_sal
```

## 그 외 DML 튜닝 방법
- Direct Path I/O 활용
  - Direct Path I/O는 버퍼 캐시를 경유하지 않고 곧바로 데이터 블록을 읽고 쓸 수 있는 기능이다.
  - Direct Path Insert, 병렬 쿼리, 병렬 DML
- 파티션 활용
  - 테이블 파티션, 인덱스 파티션

---
**Reference**<br>
- 친절한 SQL 튜닝 6장
