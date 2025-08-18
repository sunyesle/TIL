# 소트 튜닝
## Union vs Union All
Union을 사용하면 옵티마이저는 두 집합 간의 중복을 제거하기 위해 소트 연산을 수행한다.
**가능하면 Union All을 사용**해야 한다.

**예시1**<br>
<img width="301" height="111" alt="Image" src="https://github.com/user-attachments/assets/e86cf0b1-09df-4f97-b943-f82ababca83b" />

두 집합이 상호배타적인 경우 Union 대신 Union All을 사용해도 된다.
```sql
select 결제번호, 주문번호, 결제금액, 주문일자
from 결제
where 결제수단코드 = 'M' and 결제일자 = '20180316'
union all
select 결제번호, 주문번호, 결제금액, 주문일자
from 결제
where 결제수단코드 = 'C' and 결제일자 = '20180316'
```

**예시2**<br>
<img width="401" height="131" alt="Image" src="https://github.com/user-attachments/assets/e5de24d3-9cc7-4003-ba35-0916213390a4" />

소트 연산이 일어나지 않도록 Union All을 사용하면서도 데이터 중복을 피하려면 다음과 같이 하면 된다.
```sql
select 결제번호, 주문번호, 결제금액, 주문일자
from 결제
where 결제일자 = '20180316'
union all
select 결제번호, 주문번호, 결제금액, 주문일자
from 결제
where 주문일자 = '20180316'
and   결제일자 != '20180316'
```

결제일자가 Null 허용 컬럼이면 맨 아래 조건절을 다음과 같이 변경해야 한다.
```sql
and (결제일자 != '20180316' or 결제일자 is null)
```
또는
```sql
and LNNVL(결제일자 = '20180316')
```

## Exists 활용
Exists 서브쿼리는 데이터의 존재 여부만 확인하면 되기 때문에 조건절을 만족하는 데이터를 모두 읽지 않는다.

**Distinct, Minus 연산자**를 사용한 쿼리는 대부분 **Exists 서브쿼리로 변환**할 수 있다.

### Distinct 변환 예시
중복 레코드를 제거할 목적으로 Distinct를 사용하면 조건에 해당하는 데이터를 모두 읽어 중복을 제거해야 한다.
부분범위 처리가 불가능하고, 데이터를 읽는 과정에서 많은 I/O가 발생한다.

아래 쿼리는 상품유형코드 조건절에 해당하는 상품에 대해, 기간 내 계약 데이터를 모두 읽는 비효율이 있다.
상품 수는 적고 상품별 계약 건수가 많을수록 비효율이 큰 패턴이다.
```sql
select DISTINCT p.상품번호, p.상품명, p.상품가격, ...
from   상품 p, 계약 c
where  p.상품유형코드 = :pclscd
and    c.상품번호 = p.상품번호
and    c.계약일자 between :dt1 and :dt2
and    c.계약구분코드 = :ctpcd
```

아래와 같이 Exists 서브쿼리로 변환할 수 있다. 상품 테이블에 대한 부분범위 처리도 가능하다.
```sql
select p.상품번호, p.상품명, p.상품가격, ...
from   상품 p
where  p.상품유형코드 = :pclscd
and    EXISTS (select 'x' from 계약 c
               where  c.상품번호 = p.상품번호
               and    c.계약일자 between :dt1 and :dt2
               and    c.계약구분코드 = :ctpcd)
```

### Minus 변환 예시
```sql
select st.상황접수번호, st.관제일련번호, st.상황코드, st.관제일시
from   관제진행상황 st
where  상황코드 = '0001'
and    관제일시 between :v_timefrom || '000000' and :v_timeto || '235959'
MINUS
select st.상황접수번호, st.관제일련번호, st.상황코드, st.관제일시
from   관제진행상황 st, 구조활동 rpt
where  상황코드 = '0001'
and    관제일시 between :v_timefrom || '000000' and :v_timeto || '235959'
and    rpt.출동센터ID = :v_cntr_id
and    st.상황접수번호 = rpt.상황접수번호
order by 상황접수번호, 관제일시
```
아래와 같이 Not Exists 서브쿼리로 변환할 수 있다.
```sql
select st.상황접수번호, st.관제일련번호, st.상황코드, st.관제일시
from   관제진행상황 st
where  상황코드 = '0001'
and    관제일시 between :v_timefrom || '000000' and :v_timeto || '235959'
and    NOT EXISTS (select 'x' from 구조활동 rpt
                   where  rpt.출동센터ID = :v_cntr_id
                   and    rpt.상황접수번호 = st.상황접수번호)
order by 상황접수번호, 관제일시
```

## 조인 방식 변경
아래 쿼리에서 계약_X01 인덱스가 [지점ID + 계약일시] 순이면 소트 연산을 생략할 수 있지만, 해시 조인이기 때문에 Sort Order By가 나타났다.
```sql
select c.계약번호, c.상품코드, p.상품명, p.상품구분코드, c.계약일시, c.계약금액
form   계약 c, 상품 p
where  c.지점ID = :brch_id
and    p.상품코드 = c.상품코드
order by c.계약일시 desc

Execution Plan
-----------------------------------------------------
SELECT STATEMENT Optimizer=ALL_ROWS
  SORT (ORDER BY)
    HASH JOIN
      TABLE ACCESS (FULL) OF '상품' (TABLE)
      TABLE ACCESS (BY INDEX ROWID) OF '계약' (TABLE)
        INDEX (RANGE SCAN) OF '계약_X01' (INDEX)
```

계약 테이블 기준으로 상품 테이블과 NL 조인하도록 조인 방식을 변경하면 소트 연산을 생략할 수 있다.

지점 ID 조건을 만족하는 데이터가 많고 부분범위 처리 가능한 상황에서 큰 성능 개선 효과를 얻을 수 있다.
```sql
select /*+ leading(c) use_nl(p) */
       c.계약번호, c.상품코드, p.상품명, p.상품구분코드, c.계약일시, c.계약금액
form   계약 c, 상품 p
where  c.지점ID = :brch_id
and    p.상품코드 = c.상품코드
order by c.계약일시 desc

Execution Plan
-----------------------------------------------------
SELECT STATEMENT Optimizer=ALL_ROWS
  NESTED LOOPS
    NESTED LOOPS
      TABLE ACCESS (BY INDEX ROWID) OF '계약' (TABLE)
        INDEX (RANGE SCAN DESCENDING) OF '계약_X01' (INDEX)
      INDEX (UNIQUE SCAN) OF '상품_PK' (INDEX (UNIQUE))
    TABLE ACCESS (BY INDEX ROWID) OF '상품' (TABLE)
```
정렬 기준이 조인 키 컬럼이면 소트머지 조인도 Sort Order By 연산을 생략할 수 있다.

---
**Reference**
- 친절한 SQL 튜닝 5장
