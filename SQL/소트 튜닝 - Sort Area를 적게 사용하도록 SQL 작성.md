# 소트 튜닝 - Sort Area를 적게 사용하도록 SQL 작성
소트 연산이 불가피하다면 메모리 내에서 처리를 완료할 수 있도록 노력해야 한다.

## 소트 데이터 줄이기
### 예시
```sql
--개선 전
select lpad(상품번호, 30) || lpad(상품명, 30) || lpad(고객ID, 30)
    || lpad(고객명, 20) || to_char(주문일시, 'yyyymmdd hh24:mi:ss')
from 주문상품
where 주문일시 between :start and :end
order by 상품번호

--개선 후
select lpad(상품번호, 30) || lpad(상품명, 30) || lpad(고객ID, 30)
    || lpad(고객명, 20) || to_char(주문일시, 'yyyymmdd hh24:mi:ss')
from (
    select 상품번호, 상품명, 고객ID, 고객명, 주문일시
    from 주문상품
    where 주문일시 between :start and :end
    order by 상품번호
)
```

```sql
--개선 전
select *
from 예수금원장
order by 총예수금 desc

--개선 후
select 계좌번호, 총예수금
from 예수금원장
order by 총예수금 desc
```

## Top N 쿼리의 소트부하 경감 원리
Top-N 쿼리 알고리즘은 소트 연산(값 비교) 횟수와 Sort Area 사용량을 최소화해준다.

먼저 인덱스로 소트 연산을 생략할 수 없을 때, Top-N 쿼리 알고리즘의 작동 방식을 알아보자.

예를 들어, 최솟값을 갖는 10개의 레코드를 찾으려고 한다.<br>
10개의 레코드를 담을 배열을 할당하고, 처음 읽은 10개 레코드를 오름차순으로 정렬된 상태로 담는다.<br>
이후 읽는 레코드에 대해서는 배열 맨 끝에 있는 값과 비교해서 그보다 작은 값이 나타날때만 배열 내에서 정렬을 시도한다.

이 방식으로 처리하면, 대상집합이 아무리 커도 많은 메모리 공간이 필요하지 않다.<br>
전체 레코드를 모두 정렬하지 않고도 최솟값을 갖는 10개의 레코드를 찾아낼 수 있다.

## 분석함수에서의 Top N 소트
윈도우 함수 중 rank나 row_number 함수는 max 함수보다 소트 부하가 적다. Top N 소트 알고리즘이 작동하기 때문이다.

### 예시
```sql
--개선 전 (max 함수)
select 장비번호, 변경일자, 변경순번, 상태코드, 메모
from (select 장비번호, 변경일자, 변경순번, 상태코드, 메모
           , max(변경순번) over (partition by 장비번호) 최종변경순번
      from 상태변경이력
      where 변경일자 = :upd_dt)
where 변경순번 = 최종변경순번

--개선 후 (rank 함수)
select 장비번호, 변경일자, 변경순번, 상태코드, 메모
from (select 장비번호, 변경일자, 변경순번, 상태코드, 메모
           , rank() over (partition by 장비번호) rnum
      from 상태변경이력
      where 변경일자 = :upd_dt)
where rnum = 1
```

---
**Reference**
- 친절한 SQL 튜닝 5장
