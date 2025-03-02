# SQL 옵티마이저 힌트

## SQL 옵티마이저
SQL 옵티마이저는 사용자가 원하는 작업을 가장 효율적으로 수행할 수 있도록 최적의 데이터 액세스 경로를 선택해 주는 DBMS의 핵심 엔진이다.

옵티마이저의 최적화 단계를 요약하면 다음과 같다.
1. 사용자로부터 전달받은 쿼리를 수행하는 데 후보군이 될 만한 실행계획들을 찾아낸다.
2. 데이터 딕셔너리에 미리 수집해 둔 통계정보를 이용해서 각 실행계획의 예상 비용을 산정한다.
3. 최저 비용을 나타내는 실행계획을 선택한다.

## 옵티마이저 힌트
옵티마이저 힌트를 이용해 데이터 엑세스 경로를 바꿀 수 있다.

### 힌트 사용법
```sql
SELECT /*+ INDEX(A 고객_PK) */
    고객명, 연락처, 주소, 가입일시
FROM 고객 A
WHERE 고객ID = '00000001'
```

힌트 안에 인자를 나열할 때는 ,(콤마)를 사용한다.<br>
힌트와 힌트 공백을 사용한다.
```sql
/*+ INDEX(A A_X01) INDEX(B, B_X03) */
```

FROM 절 테이블명 옆에 ALIAS를 지정했다면, 힌트에도 반드시 ALIAS를 사용해야 한다.
```sql
SELECT /*+ FULL(E) */
FROM EMP E
```

### 자주 사용하는 힌트
| 분류           | 힌트               | 설명                                                                                             |
|--------------|------------------|------------------------------------------------------------------------------------------------|
| **최적화 목표**   | ALL_ROWS         | 전체 처리속도 최적화                                                                                    |
|              | FIRST_ROWS(N)    | 최초 N건 응답속도 최적화                                                                                 |
| **엑세스 방식**   | FULL             | Table Full Scan으로 유도                                                                           |
|              | INDEX            | Index Scan으로 유도                                                                                |
|              | INDEX_DESC       | Index를 역순으로 스캔하도록 유도                                                                           |
|              | INDEX_FFS        | Index Fast Full Scan으로 유도                                                                      |
|              | INDEX_SS         | Index Skip Scan으로 유도                                                                           |
| **조인 순서**    | ORDERED          | FROM 절에 나열된 순서대로 조인                                                                            |
|              | LEADING          | LEADING 힌트 괄호에 기술한 순서대로 조인 <br>ex) `LEADING(T1, T2)`                                           |
|              | SWAP_JOIN_INPUTS | 해시 조인 시, BUILD INPUT을 명시적으로 선택 <br>ex) `SWAP_JOIN_INPUTS(T1)`                                  |
| **조인 방식**    | USE_NL           | NL 조인으로 유도                                                                                     |
|              | USE_MERGE        | 소트 머지 조인으로 유도                                                                                  |
|              | USE_HASH         | 해시 조인으로 유도                                                                                     |
|              | NL_SJ            | NL 세미조인으로 유도                                                                                   |
|              | MERGE_SJ         | 소트 머지 세미조인으로 유도                                                                                |
|              | HASH_SJ          | 해시 세미조인으로 유도                                                                                   |
| **서브쿼리 팩토링** | MATERIALIZE      | WITH 문으로 정의한 집합을 물리적으로 생성하도록 유도 <br>ex) `WITH /*+ MATERIALIZE */ T AS (SELECT ... )`           |
|              | INLINE           | WITH 문으로 정의한 집합을 물리적으로 생성하지 않고 INLINE 처리하도록 유도 <br>ex) `WITH /*+ INLINE */ T AS (SELECT ... )` |
| **쿼리 변환**    | MERGE            | 뷰 머징 유도                                                                                        |
|              | NO_MERGE         | 뷰 머징 방지                                                                                        |
|              | UNNEST           | 서브쿼리 Unnesting 유도                                                                              |
|              | NO_UNNEST        | 서브쿼리 Unnesting 방지                                                                              |
|              | PUSH_PRED        | 조인조건 Pushdown 유도                                                                               |
|              | NO_PUSH_PRED     | 조인조건 Pushdown 방지                                                                               |
|              | USE_CONCAT       | OR 또는 IN 리스트 조건을 OR-Expansion으로 유도                                                             |
|              | NO_EXPAND        | OR 또는 IN 리스트 조건에 대한 OR-Expansion 방지                                                            |
| **병렬 처리**    | PARALLEL         | 테이블 스캔 또는 DML을 병렬 방식으로 처리하도록 유도 <br>ex) `PARALLEL(T1 2) PARALLEL(T2 2)`                        |
|              | PARALLEL_INDEX   | 인덱스 스캔을 병렬 방식으로 처리하도록 유도                                                                       |
|              | PQ_DISTRIBUTE    | 병렬 수행 시 데이터 분배 방식 결정 <br>ex) `PQ_DISTRIBUTE(T1 HASH HASH)`                                     |
| **기타**       | APPEND           | Direct-Path Insert로 유도                                                                         |
|              | DRIVING_SITE     | DB Link Remote 쿼리에 대한 최적화 및 실행 주체 지정(Local 또는 Remote)                                          |
|              | PUSH_SUBQ        | 서브쿼리를 가급적 빨리 필터링하도록 유도                                                                         |
|              | NO_PUSH_SUBQ     | 서브쿼리를 가급적 늦게 필터링하도록 유도                                                                         |

---
**Reference**
- 친절한 SQL 튜닝 1장
