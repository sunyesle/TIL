# [Oracle] DBMS_SCHEDULER 스케줄러 잡
`DBMS_SCHEDULER`는 기존 `DBMS_JOB` 패키지보다 확장된 기능을 제공하는 패키지로 Oracle 10g 이상에서 사용 가능하다. 이를 통해, 주기적인 작업(Job)을 생성하고 관리할 수 있다.

> 테스트에 사용한 테이블과 프로시저 생성 쿼리는 글 하단에 있다.

## 잡 생성
```sql
BEGIN
DBMS_SCHEDULER.CREATE_JOB (
    job_name        => 'MY_JOB',
    job_type        => 'PLSQL_BLOCK',
    job_action      => 'BEGIN MY_USER.MY_PROCEDURE; END;',
    start_date      => TRUNC(SYSTIMESTAMP, 'MI'),
    repeat_interval => 'FREQ=SECONDLY; INTERVAL=10',
    enabled         => TRUE
);
END;
```

## 등록된 잡 확인
```sql
-- 등록된 잡
SELECT * FROM USER_SCHEDULER_JOBS;

-- 현재 실행중인 잡 정보
SELECT * FROM USER_SCHEDULER_RUNNING_JOBS;

-- 잡 실행 로그
SELECT * FROM USER_SCHEDULER_JOB_LOG;

-- 잡 실행 로그 상세 정보
SELECT * FROM USER_SCHEDULER_JOB_RUN_DETAILS;
```

## 잡 활성화/비활성화
```sql
--활성화
BEGIN
    DBMS_SCHEDULER.ENABLE('MY_USER.MY_JOB');
END;

-- 비활성화
BEGIN
    DBMS_SCHEDULER.DISABLE('MY_USER.MY_JOB');
END;
```

<br>

> **테스트용 테이블 & 프로시저**
> ```sql
> CREATE TABLE TEMP (
>     CREATED_AT DATE
> );
> 
> CREATE OR REPLACE PROCEDURE MY_PROCEDURE
> IS
> BEGIN
>     INSERT INTO TEMP(CREATED_AT) VALUES(SYSDATE);
> END MY_PROCEDURE;
> ```

---
**Reference**
- https://coding-factory.tistory.com/462
- https://m.blog.naver.com/fromyongsik/222268984228
