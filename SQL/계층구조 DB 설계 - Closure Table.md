# 계층구조 DB 설계 - Closure Table

## 클로저 테이블(Closure Table)
트리의 모든 경로를 데이터와 분리하여 별개의 테이블에 저장한다.

이 테이블에는 자기 자신을 참조하는 행을 포함한, 트리의 조상, 자손 관계를 가진 모든 노드 들의 관계를 한 행으로 저장한다.

## 샘플 데이터
```sql
CREATE TABLE category (
    id    INT          PRIMARY KEY,
    name  VARCHAR(100) NOT NULL
);

CREATE TABLE category_closure (
    ancestor_id   INT NOT NULL,
    descendant_id INT NOT NULL,
    depth         INT NOT NULL,
    PRIMARY KEY (ancestor_id, descendant_id)
);

INSERT INTO category (id, name) VALUES
(1, '식품'),
(2, '신선식품'),
(3, '채소'),
(4, '가공식품'),
(5, '음료'),
(6, '과자'),
(7, '건강식품');

INSERT INTO category_closure (ancestor_id, descendant_id, depth) VALUES
-- 식품
(1, 1, 0), -- 자기 자신
(1, 2, 1), -- 신선식품
(1, 3, 2), -- 채소
(1, 4, 1), -- 가공식품
(1, 5, 2), -- 음료
(1, 6, 2), -- 과자
(1, 7, 1), -- 건강식품

-- 신선식품
(2, 2, 0), -- 자기 자신
(2, 3, 1), -- 채소

-- 가공 식품
(4, 4, 0), -- 자기 자신
(4, 5, 1), -- 음료
(4, 6, 1), -- 과자

-- 리프노드
(3, 3, 0), -- 채소
(5, 5, 0), -- 음료
(6, 6, 0), -- 과자
(7, 7, 0); -- 건강식품
```

## 예제
**모든 상위 카테고리 조회**
```sql
SELECT c.* FROM category c
JOIN category_closure cc ON c.id = cc.ancestor_id
WHERE cc.descendant_id = 4;
```

**모든 하위 카테고리를 찾아오는 쿼리**
```sql
SELECT c.* from category c
JOIN category_closure cc ON c.id = cc.descendant_id 
WHERE cc.ancestor_id = 4;
```

**삽입**
```sql
INSERT INTO category (id, name)
VALUES (8, '과일');

INSERT INTO category_closure (ancestor_id, descendant_id, depth)
SELECT ancestor_id , 8, depth + 1 
FROM category_closure 
WHERE descendant_id = 2 -- 부모 id
UNION ALL
SELECT 8, 8, 0
;
```

**삭제**
```sql
DELETE FROM category
WHERE id = 4 
OR id IN (
    SELECT descendant_id
    FROM category_closure
    WHERE ancestor_id = 4
);

DELETE FROM category_closure
WHERE descendant_id IN (
    SELECT descendant_id FROM (
        SELECT descendant_id
        FROM category_closure
        WHERE ancestor_id = 4
    ) AS subquery
);
```

---
**Reference**
- https://ibocon.tistory.com/204
- https://izzy.tistory.com/500
- https://wildeveloperetrain.tistory.com/150
- https://blog.yevgnenll.me/posts/save-tree-in-mysql-closure-pattern-hierarchy-structure
