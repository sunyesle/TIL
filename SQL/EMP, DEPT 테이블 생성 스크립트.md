# EMP, DEPT 테이블 생성 스크립트

## Oracle
```sql
CREATE TABLE DEPT (
  DEPTNO NUMBER(2),
  DNAME  VARCHAR2(14),
  LOC    VARCHAR2(13)
);

INSERT INTO DEPT VALUES (10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO DEPT VALUES (20, 'RESEARCH',   'DALLAS');
INSERT INTO DEPT VALUES (30, 'SALES',      'CHICAGO');
INSERT INTO DEPT VALUES (40, 'OPERATIONS', 'BOSTON');

CREATE TABLE EMP (
  EMPNO    NUMBER(4),
  ENAME    VARCHAR2(10),
  JOB      VARCHAR2(9),
  MGR      NUMBER(4),
  HIREDATE DATE,
  SAL      NUMBER(7,2),
  COMM     NUMBER(7,2),
  DEPTNO   NUMBER(2)
);

INSERT INTO EMP VALUES (7369, 'SMITH',  'CLERK',     7902, TO_DATE('17-12-1980','DD-MM-YYYY'),  800, NULL, 20); 
INSERT INTO EMP VALUES (7499, 'ALLEN',  'SALESMAN',  7698, TO_DATE('20-02-1981','DD-MM-YYYY'), 1600,  300, 30);
INSERT INTO EMP VALUES (7521, 'WARD',   'SALESMAN',  7698, TO_DATE('22-02-1981','DD-MM-YYYY'), 1250,  500, 30);
INSERT INTO EMP VALUES (7566, 'JONES',  'MANAGER',   7839, TO_DATE('02-04-1981','DD-MM-YYYY'), 2975, NULL, 20);
INSERT INTO EMP VALUES (7654, 'MARTIN', 'SALESMAN',  7698, TO_DATE('28-09-1981','DD-MM-YYYY'), 1250, 1400, 30);
INSERT INTO EMP VALUES (7698, 'BLAKE',  'MANAGER',   7839, TO_DATE('01-05-1981','DD-MM-YYYY'), 2850, NULL, 30);
INSERT INTO EMP VALUES (7782, 'CLARK',  'MANAGER',   7839, TO_DATE('09-06-1981','DD-MM-YYYY'), 2450, NULL, 10);
INSERT INTO EMP VALUES (7788, 'SCOTT',  'ANALYST',   7566, TO_DATE('09-12-1982','DD-MM-YYYY'), 3000, NULL, 20);
INSERT INTO EMP VALUES (7839, 'KING',   'PRESIDENT', NULL, TO_DATE('17-11-1981','DD-MM-YYYY'), 5000, NULL, 10);
INSERT INTO EMP VALUES (7844, 'TURNER', 'SALESMAN',  7698, TO_DATE('08-09-1981','DD-MM-YYYY'), 1500,    0, 30);
INSERT INTO EMP VALUES (7876, 'ADAMS',  'CLERK',     7788, TO_DATE('23-05-1987','DD-MM-YYYY'), 1100, NULL, 20);
INSERT INTO EMP VALUES (7900, 'JAMES',  'CLERK',     7698, TO_DATE('03-12-1981','DD-MM-YYYY'),  950, NULL, 30);
INSERT INTO EMP VALUES (7902, 'FORD',   'ANALYST',   7566, TO_DATE('03-12-1981','DD-MM-YYYY'), 3000, NULL, 20);
INSERT INTO EMP VALUES (7934, 'MILLER', 'CLERK',     7782, TO_DATE('23-01-1982','DD-MM-YYYY'), 1300, NULL, 10);
```

## MySQL
```sql
CREATE TABLE DEPT (
  DEPTNO DECIMAL(2),
  DNAME  VARCHAR(14),
  LOC    VARCHAR(13) 
);

INSERT INTO DEPT VALUES (10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO DEPT VALUES (20, 'RESEARCH',   'DALLAS');
INSERT INTO DEPT VALUES (30, 'SALES',      'CHICAGO');
INSERT INTO DEPT VALUES (40, 'OPERATIONS', 'BOSTON');

CREATE TABLE EMP (
  EMPNO    DECIMAL(4),
  ENAME    VARCHAR(10),
  JOB      VARCHAR(9),
  MGR      DECIMAL(4) ,
  HIREDATE DATE,
  SAL      DECIMAL(7,2),
  COMM     DECIMAL(7,2),
  DEPTNO   DECIMAL(2) 
);

INSERT INTO EMP VALUES (7369, 'SMITH',  'CLERK',     7902, STR_TO_DATE('17-12-1980','%d-%m-%Y'),  800, NULL, 20);
INSERT INTO EMP VALUES (7499, 'ALLEN',  'SALESMAN',  7698, STR_TO_DATE('20-02-1981','%d-%m-%Y'), 1600,  300, 30);
INSERT INTO EMP VALUES (7521, 'WARD',   'SALESMAN',  7698, STR_TO_DATE('22-02-1981','%d-%m-%Y'), 1250,  500, 30);
INSERT INTO EMP VALUES (7566, 'JONES',  'MANAGER',   7839, STR_TO_DATE('02-04-1981','%d-%m-%Y'), 2975, NULL, 20);
INSERT INTO EMP VALUES (7654, 'MARTIN', 'SALESMAN',  7698, STR_TO_DATE('28-09-1981','%d-%m-%Y'), 1250, 1400, 30);
INSERT INTO EMP VALUES (7698, 'BLAKE',  'MANAGER',   7839, STR_TO_DATE('01-05-1981','%d-%m-%Y'), 2850, NULL, 30);
INSERT INTO EMP VALUES (7782, 'CLARK',  'MANAGER',   7839, STR_TO_DATE('09-06-1981','%d-%m-%Y'), 2450, NULL, 10);
INSERT INTO EMP VALUES (7788, 'SCOTT',  'ANALYST',   7566, STR_TO_DATE('09-12-1982','%d-%m-%Y'), 3000, NULL, 20);
INSERT INTO EMP VALUES (7839, 'KING',   'PRESIDENT', NULL, STR_TO_DATE('17-11-1981','%d-%m-%Y'), 5000, NULL, 10);
INSERT INTO EMP VALUES (7844, 'TURNER', 'SALESMAN',  7698, STR_TO_DATE('08-09-1981','%d-%m-%Y'), 1500,    0, 30);
INSERT INTO EMP VALUES (7876, 'ADAMS',  'CLERK',     7788, STR_TO_DATE('23-05-1987','%d-%m-%Y'), 1100, NULL, 20);
INSERT INTO EMP VALUES (7900, 'JAMES',  'CLERK',     7698, STR_TO_DATE('03-12-1981','%d-%m-%Y'),  950, NULL, 30);
INSERT INTO EMP VALUES (7902, 'FORD',   'ANALYST',   7566, STR_TO_DATE('03-12-1981','%d-%m-%Y'), 3000, NULL, 20);
INSERT INTO EMP VALUES (7934, 'MILLER', 'CLERK',     7782, STR_TO_DATE('23-01-1982','%d-%m-%Y'), 1300, NULL, 10);
```
