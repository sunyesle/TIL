# [Oracle] DBMS_JOB 스케줄러 잡
`DBMS_JOB` 패키지는 주기적인 작업(Job)을 등록하고 관리하는데 사용되는 패키지다.

> 10g 이후부터는 `DBMS_SCHEDULER`가 권장된다.

## 잡 조회
```sql
SELECT * FROM USER_JOBS;

SELECT JOB, WHAT, FAILURES, TOTAL_TIME, LAST_DATE, LAST_SEC, NEXT_DATE, NEXT_SEC, INTERVAL 
FROM USER_JOBS
ORDER BY NEXT_DATE;
```

## 잡 등록
```sql
DECLARE 
    X NUMBER; 
BEGIN 
DBMS_JOB.SUBMIT ( 
    JOB       => X, 
    WHAT      => 'MY_USER.MY_PROCEDURE;', 
    NEXT_DATE => TRUNC(SYSDATE, 'MI') + (1/1440),
    INTERVAL  => 'TRUNC(SYSDATE, ''MI'') + (1/1440)',
    NO_PARSE  => FALSE 
); 
END;
```
- **일주일에 1회**(자정 기준): `TRUNC(SYSDATE) + 7`
- **하루에 1회**(자정 기준): `TRUNC(SYSDATE) + 1`
- **1시간에 1회**(매 시 정각): `TRUNC(SYSDATE, 'HH24') + 1/24`
- **1분에 1회**(매 분 정각): `TRUNC(SYSDATE, 'MI') + 1/60/24`

## 잡 중지/재실행
```sql
--활성화
BEGIN
    DBMS_JOB.BROKEN([JOB번호], FALSE);
END;

-- 비활성화
BEGIN
    DBMS_JOB.BROKEN([JOB번호], TRUE);
END;
```

## 잡 실행
```sql
BEGIN
    DBMS_JOB.RUN([JOB번호]);
END;
```

## 잡 삭제
```sql
BEGIN
    DBMS_JOB.REMOVE([JOB번호]);
END;
```
