# SQL 옵티마이저
## 옵티마이저 종류
**비용 기반 옵티마이저**(**CBO**)는 후보군이 될 만한 실행계획들을 도출하고,
데이터 딕셔너리에 미리 수집해 둔 통계정보를 이용해 각 실행계획의 예상 비용을 산정하고,
그중 가장 낮은 비용의 실행계획 하나를 선택하는 옵티마이저다.

CBO가 사용하는 **통계정보**로는 데이터양, 컬럼값의 수, 컬럼값 분포, 인덱스 높이, 클러스터링 팩터 등이 있다.

**통계정보 종류**
- 오브젝트 통계
  - 테이블 통계
  - 인덱스 통계
  - 컬럼 통계
- 시스템 통계

## 옵티마이저 모드
최적화 목표를 설정하는 기능이다.
- **ALL_ROWS**: 전체 쿼리속도 최적화
- **FIRST_ROWS**: 최초 응답속도 최적화 (deprecated. FIRST_ROWS_N 사용이 권장된다.)
- **FIRST_ROWS_N**: 최초 N건 응답속도 최적화

alter session 또는 alter system 명령어로 FIRST_ROWS_N 옵티마이저 모드를 설정할 때 지정할 수 있는 값은 1, 10, 100, 1000 네 가지다.
```sql
alter session set optimizer_mode = first_rows_1;
alter session set optimizer_mode = first_rows_10;
alter session set optimizer_mode = first_rows_100;
alter session set optimizer_mode = first_rows_1000;
```

FIRST_ROWS(N) 힌트로 지정할 때는 0보다 큰 정수값을 입력할 수 있다.
```sql
select /*+ first_rows(30) */ col1, col2, col3 from t where ...
```

## 옵티마이저에 영향을 미치는 요소
- SQL과 연산자 형태
- 인덱스, IOT, 클러스터, 파티션, MV 등 옵티마이징 팩터
- 제약 설정
- 통계 정보
- 옵티마이저 힌트
- 옵티마이저 관련 파라미터

## 옵티마이저의 한계
다음과 같은 한계와 제약으로 인해 옵티마이저는 불완전할 수밖에 없다. 

옵티마이저 행동에 가장 큰 영향을 미치는 통계정보를 필요한 만큼 충분히 확보하는 것부터 불가능한 일이다. 정보가 많을수록 좋지만, 수집하고 관리하는 데 시간과 비용이 들기 때문이다.

기본적으로 비용 기반으로 작동하지만, 내부적으로 여러 가정과 정해진 규칙을 이용해 기계적인 선택을 한다는 사실도 한계를 보이는 원인 중 하나이다.
또한 최적화에 허용되는 시간이 매우 짧은 것도 중요한 제약 중 하나이다.

## 개발자의 역할
### 필요한 최소 블록만 읽도록 쿼리 작성
데이터베이스 성능은 I/O 효율에 달려있으므로 동일한 레코드를 반복적으로 읽지 않고, 필요한 최소 블록만 읽도록 해야 한다.

페이징 쿼리 개선 사례와 함께 알아보자.
```sql
SELECT *
FROM (
     SELECT ROWNUM NO, 등록일자, 번호, 제목, 회원명, 게시판유형명, 질문유형명, 아이콘, 댓글개수
     FROM (
          SELECT A.등록일자, A.번호, A.제목, B.회원명, C.게시판유형명, D.질문유형명
               , GET_ICON(D.질문유형코드) 아이콘, (SELECT ... FROM ...) 댓글개수
          FROM 게시판 A, 회원 B, 게시판유형 C, 질문유형 D
          WHERE A.게시판유형 = :TYPE
          AND   B.회원번호 = A.작성자번호
          AND   C.게시판유형 = A.게시판유형
          AND   D.질문유형 = A.질문유형
          ORDER BY A.등록일자 DESC, A.질문유형, A.번호
     )
     WHERE ROWNUM <= (:PAGE * 10)
)
WHERE NO >= (PAGE-1)*10 + 1
```

최종 결과 집합에 대해서만 GET_ICON 함수를 호출하고, 댓글개수를 세는 스칼라 서브쿼리를 수행하도록 수정.<br>
최종 결과 집합에 대해서만 회원, 게시판유형, 질문유형 테이블과 NL 조인을 수행하도록 수정.
```sql
SELECT /*+ ORDERED USE_NL(B) USE_NL(C) USE_NL(D) */
       A.등록일자, A.번호, A.제목, B.회원명, C.게시판유형명, D.질문유형명
     , GET_ICON(D.질문유형코드) 아이콘, (SELECT ... FROM ...) 댓글개수
FROM (
     SELECT A.*, ROWNUM NO
     FROM (
          SELECT 등록일자, 번호, 제목
          FROM 게시판
          WHERE 게시판유형 = :TYPE
          ORDER BY 등록일자 DESC, 질문유형, 번호
     ) A
     WHERE ROWNUM <= (:PAGE * 10)
) A, 회원 B, 게시판유형 C, 질문유형 D
WHERE A.NO >= (PAGE-1)*10 + 1
AND   B.회원번호 = A.작성자번호
AND   C.게시판유형 = A.게시판유형
AND   D.질문유형 = A.질문유형
ORDER BY A.등록일자 DESC, A.질문유형, A.번호
```

### 최적의 옵티마이징 팩터 제공
- 전략적인 인덱스 구성
- 파티션, 클러스터, IOT, MV, Result Cache 등 DBMS가 제공하는 기능 활용
- 적절한 옵티마이저 모드 설정
- 정확하고 안정적인 통계정보 수집

### 필요하다면, 옵티마이저 힌트를 사용해 최적의 액세스 경로로 유도
옵티마이저가 생각하지 못한 최적의 경로를 찾아내고, 실행계획을 그 방식으로 유도할 수 있어야 한다.

---
**Reference**<br>
- 친절한 SQL 튜닝 7장
