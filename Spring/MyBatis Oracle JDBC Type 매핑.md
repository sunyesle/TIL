# MyBatis JDBC Type

## 오라클 JDBC Type 매핑
| Oracle Type                        | JDBC Type                      | Java Language Type            |
|---------------------------------------|---------------------------------|---------------------------|
| CHAR                                  | java.sql.Types.CHAR             | java.lang.String          |
| VARCHAR2                              | java.sql.Types.VARCHAR          | java.lang.String          |
| LONG                                  | java.sql.Types.LONGVARCHAR      | java.lang.String          |
| NUMBER                                | java.sql.Types.NUMERIC          | java.math.BigDecimal      |
| NUMBER                                | java.sql.Types.DECIMAL          | java.math.BigDecimal      |
| NUMBER                                | java.sql.Types.BIT              | boolean                   |
| NUMBER                                | java.sql.Types.TINYINT          | byte                      |
| NUMBER                                | java.sql.Types.SMALLINT         | short                     |
| NUMBER                                | java.sql.Types.INTEGER          | int                       |
| NUMBER                                | java.sql.Types.BIGINT           | long                      |
| NUMBER                                | java.sql.Types.REAL             | float                     |
| NUMBER                                | java.sql.Types.FLOAT            | double                    |
| NUMBER                                | java.sql.Types.DOUBLE           | double                    |
| RAW                                   | java.sql.Types.BINARY           | byte[]                    |
| RAW                                   | java.sql.Types.VARBINARY        | byte[]                    |
| LONGRAW                               | java.sql.Types.LONGVARBINARY    | byte[]                    |
| DATE                                  | java.sql.Types.DATE             | java.sql.Date             |
| DATE                                  | java.sql.Types.TIME             | java.sql.Time             |
| TIMESTAMP                             | java.sql.Types.TIMESTAMP        | java.sql.Timestamp        |
| BLOB                                  | java.sql.Types.BLOB             | java.sql.Blob             |
| CLOB                                  | java.sql.Types.CLOB             | java.sql.Clob             |
| NCLOB                                 | java.sql.Types.NCLOB            | java.sql.NClob            |
| NCHAR                                 | java.sql.Types.NCHAR            | java.lang.String          |

---
**Reference**<br>
- https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#%EC%A7%80%EC%9B%90%EB%90%98%EB%8A%94-jdbc-%ED%83%80%EC%9E%85
- https://docs.oracle.com/cd/A97336_01/buslog.102/a83724/basic3.htm
- https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/accessing-and-manipulating-Oracle-data.html#GUID-1AF80C90-DFE6-4A3E-A407-52E805726778
- https://bravesuccess.tistory.com/295
