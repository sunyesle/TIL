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

---
**Reference**<br>
- 친절한 SQL 튜닝 6장
