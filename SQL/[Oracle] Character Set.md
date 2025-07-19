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

## 문자셋(NLS_CHARACTERSET)
데이터베이스의 기본 Character Set이다.<br>
CHAR, VARCHAR2, CLOB, LONG 데이터 타입의 인코딩을 정의한다.

## 국제 문자셋(NLS_NCHAR_CHARACTERSET)
데이터베이스의 기본 Character Set이 유니코드를 지원하지 않을 때, 유니코드를 지원하기 부가적으로 설정할 수 있는 Character Set이다.<br>
NCHAR, NVARCHAR2, NCLOB 데이터 타입의 인코딩을 정의한다.

## Charset 종류

<img width="901" height="409" alt="Image" src="https://github.com/user-attachments/assets/140b2033-e6d4-4ea3-ac16-9fd7a5f21032" />

### 권장 Charset
대한민국에서만 사용되는 시스템 : **KO16MSWIN949**
- 한글 Windows에서 한글IME를 통해 입력할 수 있는 문자는 모두 지원 가능하다. (한글, 한자, 영문, 숫자 등)
- 한글과 한자는 2 byte로 저장되고, 영문과 숫자는 1 byte로 저장된다.

다국어 문자를 저장해야 하는 시스템 : **AL32UTF8**
- AL32UTF8은 최신 Unicode에서 추가되고 있는 문자들도 모두 지원 가능하다. (중국 한자, 서유럽 문자, 동남아 문자 등)
- 영문과 숫자는 1 byte로 저장되고, 나머지 문자들은 대부분 3 byte로 저장된다. (일부 4 byte로 저장되는 문자가 있으나 거의 사용되지 않는 문자이다.)

---
**Reference**<br>
- https://071217.tistory.com/124
- https://hrjeong.tistory.com/276
- https://smarttechways.com/2023/05/29/nls_nchar_characterset-and-nls_characterset-define-in-oracle/
- https://rastalion.dev/oracle-character-set-%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC/
