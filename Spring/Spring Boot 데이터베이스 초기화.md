## JPA를 사용하여 데이터베이스 초기화

```
spring:
  jpa:
    generate-ddl: true
    hibernate:
      ddl-auto: none
```

- `spring.jpa.generate-ddl=(boolean)`: DDL 생성 기능 사용 여부를 지정한다.

- `spring.jpa.hibernate.ddl-auto=(enum)`: Hibernate가 데이터베이스 스키마를 자동으로 생성하는 방법을 지정한다.

  - `create`: 기존 테이블을 삭제한 다음 새 테이블을 만든다.
  - `update`: 변경 사항을 업데이트한다. 애플리케이션에 더 이상 필요하지 않더라도 기존 테이블이나 열은 삭제되지 않는다.
  - `create-drop`: create와 유사하나 애플리케이션 종료 시점에 데이터베이스를 삭제한다. 일반적으로 단위 테스트에 사용된다.
  - `validate`: 엔티티와 테이블과 정상적으로 매핑되었는지 검증한다. 그렇지 않으면 예외가 발생한다.
  - `none`: 엔티티를 통한 자동 생성을 사용하지 않는다.

## SQL 스크립트를 사용하여 데이터베이스 초기화

프로젝트 루트의 `schema.sql`, `data.sql` 에서 SQL을 로드한다. 관념적으로 데이터 정의어(DDL)는 `schema.sql` 파일에 작성하고, 데이터 조작어(DML)은 `data.sql` 파일에 작성된다.

```
spring:
  sql:
    init:
      mode: always
```

- `spring.sql.init.mode=(enum)`

  - `embedded`: 내장형 데이터베이스(인메모리 데이터베이스)일때만 스크립트를 실행한다. (기본값)
  - `always`: 항상 스크립트를 실행한다.
  - `never` : 스크립트를 실행하지않는다.

Spring Boot 2.5 이전 버전을 사용하는 경우 `spring.datasource.initialization-mode`를 사용해야 한다.

### Hibernate 기반의 초기화와 함께 사용하는 경우

```
spring:
  jpa:
    defer-datasource-initialization: true
```

Spring Boot 2.5부터 스크립트 파일이 Hibernate 초기화 이전에 실행되도록 변경되었다. [Spring-Boot-2.5-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.5-Release-Notes#hibernate-and-datasql)

따라서, Hibernate 초기화를 통해 생성된 스키마를 수정하거나 데이터를 채우고싶다면 `spring.jpa.defer-datasource-initialization`을 `true`로 설정하여 Hibernate 초기화 이후 스크립트 파일이 실행되도록 해야한다.

---

### Reference

https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-initialization

https://www.baeldung.com/spring-boot-data-sql-and-schema-sql
