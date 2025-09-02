# DML 튜닝 - Lock과 트랜잭션 동시성 제어
## 오라클 Lock
오라클은 공유 리소스와 사용자 데이터를 보호할 목적으로 다양한 종류의 Lock을 사용한다.
### DML 로우 Lock (TX Lock)
**DML 로우 Lock**은 두 개의 동시 트랜잭션이 같은 **로우를 변경하는 것을 방지**한다.

하나의 로우를 변경하려면 로우 Lock을 먼저 설정해야 한다.

배타적 모드를 사용하므로 UPDATE/DELETE를 진행 중인 로우는 다른 트랜잭션이 UPDATE/DELETE 할 수 없다.
INSERT에 대한 로우 Lock 경합은 Unique 인덱스가 있을 때만 발생한다.

### DML 테이블 Lock (TM Lock)
**DML 테이블 Lock**은 트랜잭션이 갱신 중인 **테이블 구조를 변경하는 것을 방지**한다.

오라클은 DML 로우 Lock을 설정하기에 앞서 테이블 Lock을 먼저 설정한다.

로우 Lock에는 항상 배타적 모드를 사용하지만, 테이블 Lock에는 여러가지 Lock 모드를 사용한다.

> **대상 리소스가 사용 중일 때, 진로 선택**
> 
> Lock을 얻고자 하는 리소스가 사용 중일 때, 프로세스는 다음 세 가지 방법중 하나를 택한다.
> 보통은 내부적으로 진로가 결정되어 있지만, UPDATE FOR SELECT 문을 사용해 직접 선택할 수도 있다.
> 
> 1. Lock이 해제될 때까지 기다린다. (`select * from t for update`)
> 2. 일정 시간만 기다리다 포기한다. (`select * from t for update wait 3`)
> 3. 기다리지 않고 작업을 포기한다. (`select * from t for update nowait`)
> 
> DML 수행 시 1번, DDL 수행 시에는 3번이 자동으로 지정된다.

### Lock을 푸는 열쇠, 커밋
- **블로킹**(**Blocking**): 선행 트랜잭션이 설정한 Lock 때문에 후행 트랜잭션이 작업을 진행하지 못하고 멈춰있는 상태
- **교착상태**(**Deadlock**): 두 트랜잭션이 각각 특정 리소스에 Lock을 설정한 상태에서 상대방이 Lock을 설정한 리소스를 기다리고 있는 상황

오라클에서 교착상태가 발생하면, 이를 먼저 인지한 트랜잭션이 문장 수준 롤백을 진행한 후 아래 ORA-00060 에러 메시지를 던진다.

이제 교착상태는 해소되었으나 블로킹 상태에 놓이게 된다. 따라서 이 메시지를 받은 트랜잭션은 커밋 또는 롤백을 결정해야만 한다.
에러에 대한 예외 처리(커밋 또는 롤백)를 하지 않는다면 대기상태를 지속하게 되므로 주의가 필요하다.

#### 커밋 옵션
- **WAIT**(Default): LGWR가 로그버퍼를 파일에 기록했다는 완료 메시지를 받을 때까지 기다린다. (동기식 커밋)
- **NOWAIT**: LGWR의 완료 메시지를 기다리지 않고 바로 다음 트랜잭션을 진행한다. (비동기식 커밋)
- **IMMEDIATE**(Default): 커밋 명령을 받을 때마다 LGWR가 로그 버퍼를 파일에 기록한다.
- **BATCH**: 세션 내부에 트랜잭션 데이터를 일정량 버퍼링했다가 일괄 처리한다.

위 옵션을 조합해 아래 네 가지 커밋 명령을 사용할 수 있다.<br>
잦은 커밋으로 인한 성능저하가 있다면 비동기식 커밋이나, 배치 커밋을 활용하는 방안을 검토할 수 있다. 
```sql
COMMIT WRITE IMMEDIATE WAIT;
COMMIT WRITE IMMEDIATE NOWAIT;
COMMIT WRITE BATCH WAIT;
COMMIT WRITE BATCH NOWAIT;
```


## 트랜잭션 동시성 제어
### 비관적 동시성 제어(Pessimistic Concurrency Control)
사용자들이 같은 데이터를 동시에 수정할 것으로 가정한다.<br>
데이터를 읽는 시점에 Lock을 걸고, 트랜잭션을 완료하기 전까지 유지한다.

다음과 같이 SELETE 문에 FOR UPDATE를 사용하면 고객 레코드에 Lock을 설정하므로 데이터가 잘못 갱신되는 문제를 방지할 수 있다.
```sql
select 적립포인트, 방문횟수, 최근방문일시, 구매실적 from 고객
where 고객번호 = :cust_num for update;

--새로운 적립포인트 계산

update 고객 set 적립포인트 = :적립포인트 where 고객번호 = :cust_num
```
비관적 동시성 제어는 자칫 시스템 동시성을 크게 떨어뜨릴 우려가 있지만, FOR UPDATE에 WAIT 또는 NOWAIT 옵션을 함께 사용하면 Lock을 얻기 위해 무한정 기다리지 않아도 된다.

> **큐(Queue) 테이블 동시성 제어**
> 
> 큐 테이블에 쌓인 고객 입금 정보를 일정한 시간 간격으로 읽어서 입금 테이블에 반영하는 데몬 프로그램이 있다고 가정하자.
> 데몬이 여러 개이므로 Lock이 걸릴 수 있는 상황이다.
> 
> Lock이 걸리면 3초간 대기했다가 다시 시도하도록 하고, 한 번에 다 읽어서 처리할 경우 Lock이 풀릴 때까지 다른 데몬이 오래 걸릴 수 있으므로 100개씩만 읽도록 했다.
> ```sql
> select cust_id, rcpt_amt from cust_rcpt_Q
> where yn_upd = 'Y' and rownum <= 100 FOR UPDATE WAIT 3;
> ```
> 
> 이럴 때 다음과 같이 `SKIP LOCKED` 옵션을 사용하면 Lock이 걸린 레코드는 생략하고 다음 레코드를 읽도록 구현할 수 있다.
> ```sql
> select cust_id, rcpt_amt from cust_rcpt_Q
> where yn_upd = 'Y' FOR UPDATE SKIP LOCKED;
> ```

### 낙관적 동시성 제어(Optimistic Concurrency Control)
사용자들이 같은 데이터를 동시에 수정하지 않을 것으로 가정한다.<br>
데이터를 읽는 시점이 Lock을 설정하지 않지만, 데이터를 수정하고자 하는 시점에 앞서 읽은 데이터가 다른 사용자에 의해 변경되었는지 검사한다.

```sql
select 적립포인트, 방문횟수, 최근방문일시, 구매실적, 변경일시
into :a, :b, :c, :d, :mod_dt
from 고객
where 고객번호 = :cust_num;

--새로운 적립포인트 계산

update 고객 set 적립포인트 = :적립포인트, 변경일시 = sysdate
where 고객번호 = :cust_num
and 변경일시 = :mod_dt; -- 최종 변경일시가 앞서 읽은 값과 같은지 비교
```

### 동시성 향상과 데이터 품질을 위한 제언
- 데이터 품질을 위해 FOR UPDATE가 필요한 상황이면 이를 정확히 사용하자.
- 동시성 향상을 위해 WAIT, NOWAIT 옵션을 적절히 활용하자.
- 불필요하게 Lock을 오래 유지하지 않고, 트랜잭션의 원자성을 보장하는 범위 내에서 가급적 빨리 커밋하자.
- 동시성을 향상하고자 할 때 SQL 튜닝은 기본이다.

> **로우 Lock 대상 테이블 지정**
>
> <img width="321" height="131" alt="Image" src="https://github.com/user-attachments/assets/75fafb42-42cd-4f2c-bec3-c082f872d0e6" />
>
> 계좌마스터와 주문 테이블이 위와 같을 때, 쿼리를 다음과 같이 작성하면 두 테이블 모두에 로우 Lock이 걸린다.
> ```sql
> select b.주문수량
> from 계좌마스터 a, 주문 b
> where a.고객번호 = :cust_no
> and b.계좌번호 = a.계좌번호
> and b.주문일자 = :ord_dt
> for update
> ```
> 
> 다음과 같이 작성하면 주문수량이 있는 주문 테이블에만 로우 Lock이 걸린다.
> ```sql
> select b.주문수량
> from 계좌마스터 a, 주문 b
> where a.고객번호 = :cust_no
> and b.계좌번호 = a.계좌번호
> and b.주문일자 = :ord_dt
> for update of b.주문수량
> ```

## 채번 방식에 따른 INSERT 성능 비교
INSERT, UPDATE, DELETE, MERGE 중 가장 중요하고 튜닝 요소가 많은 것은 INSERT이다.

수행 빈도가 높은 것도 있지만, 채번 방식에 따른 성능 차이가 매우 크기 때문이다.

### 채번 테이블
순번을 채번하기 위해 별도의 테이블을 관리하는 방식이다.
채번 레코드를 읽어서 1을 더한 값으로 변경하고 그 값을 새로운 레코드를 입력하는 데 사용한다.

**장점**
- 범용성이 좋다.
- 채번 레코드를 변경하는 과정에서 액세스 직렬화가 이루어지므로 중복 값을 채번할 가능성을 방지해준다.
- INSERT 과정에서 결번을 방지할 수 있다.
- PK가 복합컬럼일 때도 사용할 수 있다.

**단점**
- 채번 레코드를 변경하기 위한 로우 Lock 경합으로 인해 성능이 좋지 않다. (자율 트랜잭션 기능을 활용하지 않은 경우)

> **자율 트랜잭션**
>
> PL/SQL의 자율 트랜잭션 기능을 이용하면 메인 트랜잭션에 영향을 주지 않고 서브 트랜잭션에서 일부 자원만 Lock을 해제할 수 있다.
> 
> PL/SQL 선언부에 `pragma autonomous_transaction`이라고 선언하기만 하면 된다.
> ```sql
> create or replace function seq_nextval(l_gubun number) return number
> as
>     pragma autonomous_transaction;
>     l_new_seq seq_tab.seq%type;
> begin
>     update seq_tab
>     set	seq = seq+1
>     where gubun = l_gubun;
>     
>     select seq into l_new_seq
>     from seq_tab
>     where gubun = l_gubun;
>     
>     commit;
>     return l_new_seq;
> end;
> ```
> 
> 메인 트랜잭션 INSERT 문에서 아래와 같이 채번 함수를 호출하고 최종적으로 커밋하지 전까지 다른 작업을 많이 수행하더라도 채번 테이블 로우 Lock은 이미 해제한 상태이므로 다른 트랜잭션을 블록킹하지 않는다.
> ```sql
> insert into target_tab values (seq_nextval(123), :x, :y, :z);
> ```

### 시퀀스 오브젝트
오라클에서 제공하는 시퀀스 오브젝트를 이용한 방식이다.

**장점**
- 성능이 빠르다.
- INSERT 과정에 중복레코드 발생에 대비한 예외처리에 크게 신경 쓰지 않아도 된다.

**단점**
- 기본적으로 PK가 단일컬럼일때만 사용 가능하다.
- INSERT 과정에서 결번이 생길 수 있다. (채번 이후 트랜잭션 롤백하는 경우, 시퀀스가 캐시에서 밀려나는 경우)

시퀀스 오브젝트는 오라클 내부에서 관리하는 채번 테이블이다.
SYS.SEQ$ 테이블을 말하며, DBA_SEQUENCES 뷰를 통해 조회할 수 있다.

시퀀스 오브젝트도 결국 테이블이므로 값을 읽고 변경하는 과정에서 Lock 메커니즘이 작동한다.
하지만 자율 트랜잭션 기능이 기본적으로 구현되어 있으며, 캐시 사이즈를 적절히 설정하면 빠른 성능을 제공한다.

> **순환옵션을 가진 시퀀스 활용**
> 
> PK가 복합 컬럼인데 동시 트랜잭션이 높아 시퀀스가 꼭 필요하다면 순환옵션을 가진 시퀀스 활용을 고려할 수 있다.
> 
> 하루에 도달할 수 없는 값으로 최댓값을 설정하고 그 값에 도달하면 1부터 다시 시작하도록 순환 옵션을 설정하는 것이다.

### MAX + 1 조회
대상 테이블의 최종 일련번호를 조회하고, 거기에 1을 더해서 INSERT하는 방식이다.

**장점**
- 별도의 채번 테이블을 관리하는 부담이 없다.
- 동시 트랜잭션에 의한 충돌이 많지 않으면, 성능이 매우 빠르다.
- PK가 복합컬럼인 경우에도 사용할 수 있다.

**단점**
- 레코드 중복에 대비한 세밀한 예외처리가 필요하다.
- 다중 트랜잭션에 의한 동시 채번이 심하면 시퀀스보다 성능이 많이 나빠질 수 있다.

---
**Reference**<br>
- 친절한 SQL 튜닝 6장
