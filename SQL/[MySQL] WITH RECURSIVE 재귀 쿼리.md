# [MySQL] WITH RECURSIVE 재귀 쿼리

## 문법
```sql
WITH RECURSIVE cte_count 
AS ( 
    -- Non-Recursive 문장(재귀의 첫 번째 부분으로, 쿼리에서 기준이 되는 초기값을 설정한다.)
    SELECT 1 AS num
    UNION ALL
    -- Recursive 문장(읽어 올 때마다 행의 위치가 기억되어 다음번 읽어 올 때 다음 행으로 이동한다.)
    SELECT num + 1
    FROM cte_count
    WHERE num < 10 -- 재귀 종료 조건
)
SELECT * FROM cte_count;
```

`WITH` 절은 쿼리 내에서 반복해서 사용할 수 있는 임시 테이블을 정의할 때 사용된다.
MySQL에서는 이렇게 생성되는 임시 테이블을 **CTE**(Common Table Expression) 라고 한다.

`WITH` 절의 CTE가 자신을 참조하는 경우 `WITH RECURSIVE`로 시작해야 한다.

## 예시
### 계층형 데이터 조회
계층형(트리) 구조의 카테고리 정보를 저장하기 위한 테이블이다.
`up_category_no` 컬럼에는 상위 카테고리의 `category_no` 값을 저장한다.
```sql
CREATE TABLE category ( 
    category_no    VARCHAR(8)   NOT NULL PRIMARY KEY, 
    category_name  VARCHAR(100) NOT NULL, 
    up_category_no VARCHAR(8)   NOT NULL, 
    display_yn     CHAR(1)      NOT NULL, 
    seq            INT 
);

INSERT INTO category (category_no, category_name, up_category_no, display_yn, seq) VALUES
-- 1Depth (최상위 카테고리)
('1100', '의류', 'M', 'Y', 1),
('1200', '식품', 'M', 'Y', 2),
('1300', 'PC/주변기기', 'M', 'Y', 3),

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

```sql
WITH RECURSIVE cte AS (
    SELECT category_no, category_name, up_category_no, seq,
           category_name AS path,
           CONVERT(seq, CHAR) AS ord,
           1 AS lv
    FROM category
    WHERE up_category_no = 'M'
    AND display_yn = 'Y'
    UNION ALL
    SELECT c.category_no, c.category_name, c.up_category_no, c.seq,
           CONCAT(cte.path, '>', c.category_name),
           CONCAT(cte.ord, '-', c.seq),
           cte.lv + 1
    FROM category c
    INNER JOIN cte ON c.up_category_no = cte.category_no
    AND display_yn = 'Y'
)
SELECT * FROM cte
ORDER BY ord;
```

> 실행결과

![실행결과](https://github.com/user-attachments/assets/03b1d6bc-4b4d-49f6-aa60-a859e9b0ad51)

---
**Reference**<br>
- https://yahwang.github.io/posts/49
- https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-RECURSIVE-%EC%9E%AC%EA%B7%80-%EC%BF%BC%EB%A6%AC
- https://velog.io/@nayu1105/SQL-%EC%9E%AC%EA%B7%80-%EC%BF%BC%EB%A6%AC
- https://www.datacamp.com/doc/mysql/mysql-with
