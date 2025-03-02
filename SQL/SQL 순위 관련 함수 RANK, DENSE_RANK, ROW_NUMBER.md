# 순위(RANK) 관련 함수

## 요약
|함수명|설명|
|------|----|
|RANK|공동 순위를 건너뛴다. (ex. 1, 2, 2, 4, 5)|
|DENSE_RANK|공동 순위를 건너뛰지 않는다. (ex. 1, 2, 2, 3, 4)|
|ROW_NUMBER|중복 값과 상관없이 각 행에 고유한 순위를 부여한다. (ex. 1, 2, 3, 4, 5)|

## 사용 예시
### 예시 테이블과 데이터
```sql
CREATE TABLE product (
  `product_id` INT NOT NULL,
  `name` VARCHAR(45) NOT NULL,
  `price` INT NOT NULL,
  PRIMARY KEY (`product_id`));

INSERT INTO product (`product_id`, `name`, `price`) 
VALUES
(1, '새우깡', 1500),
(2, '오감자', 1000),
(3, '양파링', 1500),
(4, '홈런볼', 2000),
(5, '다이제', 1700);
```

## RANK
공동 순위를 건너뛴다. (ex. 1, 2, 2, 4, 5)
```sql
SELECT NAME, PRICE,
       RANK() OVER (ORDER BY PRICE) AS `RANK`
FROM PRODUCT;
```
```sql
NAME     PRICE  RANK
-------  -----  ------
오감자    1000   1
새우깡    1500   2
양파링    1500   2
다이제    1700   4
홈런볼    2000   5
```

## DENSE_RANK
공동 순위를 건너뛰지 않는다. (ex. 1, 2, 2, 3, 4)
```sql
SELECT NAME, PRICE,
       DENSE_RANK() OVER (ORDER BY PRICE) AS `RANK`
FROM PRODUCT;
```
```sql
NAME     PRICE  RANK
-------  -----  ------
오감자    1000   1
새우깡    1500   2
양파링    1500   2
다이제    1700   3
홈런볼    2000   4
```

## ROW_NUMBER
중복 값과 상관없이 각 행에 고유한 순위를 부여한다. (ex. 1, 2, 3, 4, 5)
```sql
SELECT NAME, PRICE,
       ROW_NUMBER() OVER (ORDER BY PRICE) AS `RANK`
FROM PRODUCT;
```
```sql
NAME     PRICE  RANK
-------  -----  ------
오감자    1000   1
새우깡    1500   2
양파링    1500   3
다이제    1700   4
홈런볼    2000   5
```
---
**Reference**<br>
- https://satisfactoryplace.tistory.com/193
- https://take-it-into-account.tistory.com/81
- https://moonpiechoi.tistory.com/128
