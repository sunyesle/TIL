# [Oracle] START WITH CONNECT BY 계층형 쿼리

## 사용법
```sql
SELECT [컬럼]...
FROM [테이블]
WHERE [조건]
START WITH [시작 조건]
CONNECT BY [NOCYCLE] PRIOR [부모 코드] = [자식 코드]
ORDER SIBLINGS BY [같은 부모를 가진 형제 노드들의 정렬 조건];
```

```sql
SELECT empno
     , ename
     , mgr
     , LEVEL
FROM emp
START WITH mgr IS NULL
CONNECT BY PRIOR empno = mgr;
```

1. **START WITH 절에 시작 조건을 찾는다.**
   - mgr이 NULL인 행을 시작점으로 한다.
2. **CONNECT BY 절에 연결 조건을 찾는다.**
   - 자식컬럼(mgr)이 부모컬럼(empno)를 찾아서 하위에 붙는다.

## 계층형 쿼리에서 제공되는 의사 컬럼
| 의사 컬럼              | 설명                                                            |
|------------------------|---------------------------------------------------------------|
| **LEVEL**              | 현재 행의 계층 **깊이**(depth)를 나타낸다.                                 |
| **CONNECT_BY_ISLEAF**  | 현재 행이 **리프 노드**인지 여부를 나타낸다. `1`이면 리프 노드, `0`이면 리프 노드 아님       |
| **CONNECT_BY_ISCYCLE** | **순환 참조**가 발생하는 노드인지 여부를 나타낸다. `1`이면 순환 참조 발생, `0`이면 순환 참조 없음 |
| **CONNECT_BY_ROOT**    | **루트 노드**의 값을 반환한다. 현재 행에 대한 상위 계층에서 루트까지의 첫 번째 값을 반환         |

## 계층형 쿼리에서 제공되는 함수
### 경로 표시(SYS_CONNECT_BY_PATH)
SYS_CONNECT_BY_PATH는 루트 노드로부터의 경로를 반환하는 함수다.

*LTRIM을 이용해 앞에 출력되는 '>'를 제거하고 출력할 수 있다.*

```sql
SELECT ename
     , LTRIM(SYS_CONNECT_BY_PATH(ename, '>'), '>') AS path
FROM emp
START WITH mgr IS NULL
CONNECT BY PRIOR empno = mgr;
```

### 루트 노드 값(CONNECT_BY_ROOT)
CONNECT_BY_ROOT는 루트 노드의 값을 반환하는 함수다.
```sql
SELECT ename
     , CONNECT_BY_ROOT(ename) AS root_name
FROM emp
START WITH mgr IS NULL
CONNECT BY PRIOR empno = mgr;
```

## 예시
**샘플 데이터**
```sql
CREATE TABLE category (
    category_no    VARCHAR2(8)   NOT NULL PRIMARY KEY,
    category_name  VARCHAR2(100) NOT NULL,
    up_category_no VARCHAR2(8)   ,
    display_yn     CHAR(1)       NOT NULL,
    seq            NUMBER
);

INSERT INTO category (category_no, category_name, up_category_no, display_yn, seq) VALUES
-- 1Depth (최상위 카테고리)
('1100', '의류', NULL, 'Y', 1),
('1200', '식품', NULL, 'Y', 2),
('1300', 'PC/주변기기', NULL, 'Y', 3),

-- 2Depth
('1110', '남성 의류', '1100', 'Y', 1),
('1120', '여성 의류', '1100', 'Y', 2),
('1130', '아동 의류', '1100', 'Y', 3),
('1210', '신선 식품', '1200', 'Y', 1),
('1220', '가공 식품', '1200', 'Y', 2),
('1230', '음료/주류', '1200', 'Y', 3),
('1310', '노트북/데스크탑', '1300', 'Y', 1),
('1320', 'PC 주변기기', '1300', 'Y', 2),
('1330', '소프트웨어', '1300', 'Y', 3),

-- 3Depth
('1111', '정장', '1110', 'Y', 1),
('1112', '캐주얼', '1110', 'Y', 2),
('1121', '드레스', '1120', 'Y', 1),
('1122', '블라우스', '1120', 'Y', 2),
('1211', '과일', '1210', 'Y', 1),
('1212', '채소', '1210', 'Y', 2),
('1311', '게이밍 노트북', '1310', 'Y', 1),
('1312', '사무용 데스크탑', '1310', 'Y', 2);
```

**조회 쿼리**
```sql
SELECT LEVEL
     , CONNECT_BY_ISLEAF AS isleaf
     , LPAD(' ', (LEVEL-1) * 2) || category_name AS name
     , LTRIM(SYS_CONNECT_BY_PATH(category_name, '>'), '>') AS path
     , CONNECT_BY_ROOT(CATEGORY_NAME) AS root_name
     , category_no 
     , up_category_no
     , seq
FROM category
WHERE display_yn = 'Y'
START WITH up_category_no IS NULL
CONNECT BY PRIOR category_no = up_category_no
ORDER SIBLINGS BY seq;
```

**실행결과**

![실행결과](https://github.com/user-attachments/assets/fa718998-b6a2-4571-b7d8-e617e17ef68a)

---
**Reference**<br>
- https://blog.naver.com/l1523/221929073270
- https://kkongchii.tistory.com/entry/SQL-Oracle-%EA%B3%84%EC%B8%B5%ED%98%95-%EC%A7%88%EC%9D%98-START-WITH-CONNECT-BY-SYSCONNECTBYPATH
- https://gent.tistory.com/640
