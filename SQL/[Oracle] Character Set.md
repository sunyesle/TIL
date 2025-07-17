# [Oracle] Character Set

## Charset 확인 쿼리
```sql
SELECT *
  FROM nls_database_parameters
 WHERE parameter IN ('NLS_CHARACTERSET', 'NLS_NCHAR_CHARACTERSET');
 
PARAMETER              VALUE
---------------------- ---------
NLS_CHARACTERSET       AL32UTF8
NLS_NCHAR_CHARACTERSET AL16UTF16
```

### 문자셋(NLS_CHARACTERSET)
데이터베이스의 기본 Character Set이다.<br>
CHAR, VARCHAR2, CLOB, LONG 데이터 타입의 인코딩을 정의한다.

### 국제 문자셋(NLS_NCHAR_CHARACTERSET)
데이터베이스의 기본 Character Set이 유니코드를 지원하지 않을 때, 유니코드를 지원하기 부가적으로 설정할 수 있는 Character Set이다.<br>
NCHAR, NVARCHAR2, NCLOB 데이터 타입의 인코딩을 정의한다.

---
**Reference**<br>
- https://071217.tistory.com/124
- https://hrjeong.tistory.com/276
- https://smarttechways.com/2023/05/29/nls_nchar_characterset-and-nls_characterset-define-in-oracle/
- https://rastalion.dev/oracle-character-set-%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC/
