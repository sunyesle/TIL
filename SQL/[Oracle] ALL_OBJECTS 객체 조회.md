# [Oracle] ALL_OBJECTS 객체 조회
ALL_OBJECTS는 사용자가 접근 가능한 모든 객체에 대한 메타데이터 정보를 제공하는 데이터 딕셔너리 뷰이다.

## 주요 컬럼
|     컬럼명    |                   설명                  |
|---------------|-----------------------------------------|
| OWNER         | 객체를 소유한 사용자                    |
| OBJECT_NAME   | 객체의 이름                             |
| OBJECT_ID     | 내부적으로 부여되는 객체 ID             |
| OBJECT_TYPE   | 객체의 유형 (TABLE, VIEW, PROCEDURE 등) |
| CREATED       | 객체가 생성된 날짜                      |
| LAST_DDL_TIME | 마지막 DDL(정의어) 변경 시각            |
| STATUS        | VALID 또는 INVALID 상태                 |
| TEMPORARY     | 임시 객체 여부                          |
| GENERATED     | 시스템에 의해 생성된 객체 여부          |

## 시스템 뷰 접두어
시스템 뷰의 접두어를 변경해서 사용 가능하다.
| 뷰 이름       | 조회 대상               | 사용 권한     |
|--------------|-------------------------|--------------|
| USER_OBJECTS | 사용자가 소유한 객체      | 일반 사용자   |
| ALL_OBJECTS  | 사용자가 접근 가능한 객체 | 일반 사용자   |
| DBA_OBJECTS  | 데이터베이스 전체 객체    | DBA 권한 필요 |

## 예시
### INVALID 상태의 객체 조회
```sql
select owner,
       object_type,
       object_name,
       status
from ALL_OBJECTS
where status = 'INVALID'
order by owner, object_type, object_name; 
```

---
**Reference**<br>
- https://devhandbook.tistory.com/entry/Oracle-오라클-ALLOBJECTS-뷰-완전-정복-객체-관리와-분석의-핵심
- https://docs.oracle.com/en/database/oracle/oracle-database/23/refrn/ALL_OBJECTS.html
- https://gent.tistory.com/460
