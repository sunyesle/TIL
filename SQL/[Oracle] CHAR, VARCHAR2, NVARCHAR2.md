# [Oracle] CHAR, VARCHAR, NVARCHAR

Oracle의 문자 데이터 타입

## CHAR
길이가 n byte인 고정길이 문자열 데이터 타입이다.<br>
컬럼 정의 시 지정된 길이보다 짧은 길이의 문자열을 저장하는 경우 나머지 공간은 공백으로 채워진다.<br>
최대 사이즈: 2000 bytes

## VARCHAR2, VARCHAR
길이가 n byte인 가변길이 문자열 데이터 타입이다.<br>
문자열의 크기에 따라 저장 공간이 자동으로 조정된다.<br>
최대 사이즈: 4000 bytes

## NVARCHAR2, NVARCHAR
길이가 n자인 가변길이 문자열 데이터 타입이다.<br>
국제 문자셋(National Character Set)을 사용한다.(`NLS_NCHAR_CHARACTERSET`)<br>
최대 사이즈: 4000 bytes

> Oracle에서 `VARCHAR`와 `VARCHAR2`, `NVARCHAR`와 `NVARCHAR2`는 현재 동의어지만 추후 호환성을 위해 `VARCHAR2`, `NVARCHAR2` 사용을 권장한다.

---
**Reference**
- https://071217.tistory.com/124
- https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/datatype-limits.html
