# P6Spy

### 기존 SQL 로깅의 문제점
JPA를 사용하고 있을 경우 외부 라이브러리 없이도 다음과 같은 설정을 통해 SQL문과 쿼리 파라미터의 로그를 남길 수 있다.

**application.yml** (Spring Boot 3 [Hibernate 6] 기준)
```yaml
spring:
  jpa:
    properties:
      hibernate:
        format_sql: true # sql 포매팅
logging:
  level:
    org.hibernate.SQL: DEBUG # sql 로그 출력
    org.hibernate.orm.jdbc.bind: TRACE # 쿼리 파라미터 로그 출력
```

```log
2024-07-26T13:25:29.098+09:00 DEBUG 31732 --- [           main] org.hibernate.SQL                        : 
    select
        p1_0.id,
        p1_0.company_id,
        p1_0.name,
        p1_0.product_type 
    from
        product p1_0 
    where
        p1_0.id=?
2024-07-26T13:25:29.100+09:00 TRACE 31732 --- [           main] org.hibernate.orm.jdbc.bind              : binding parameter (1:INTEGER) <- [1]
```
SQL에 바인딩 되는 파라미터가 `?`로 표시되기 때문에, 파라미터가 많아지면 실제 실행된 쿼리를 확인하기 불편하다.

<br>

**P6Spy 라이브러리를 사용하면 파라미터가 바인딩 된 SQL문을 로그로 남길 수 있다.**

## 의존성 추가
P6Spy 의존성을 추가한다. 1.9.0부터 Spring Boot 3를 지원하며, Spring Boot 2 애플리케이션의 경우 최신 호환 버전은 1.8.1이다.

**build.gradle**
```gradle
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.1'
````
Spring Boot 와 P6Spy 간의 자동 구성을 지원하기 때문에, 의존성 추가 후 애플리케이션을 실행해 보면 별다른 설정 없이도 SQL 로그가 남는 것을 확인할 수 있다.

```log
2024-07-26T13:11:35.394+09:00  INFO 28920 --- [           main] p6spy                                    : #1721968247644 | took 2ms | statement | connection 6| url jdbc:h2:mem:test
select p1_0.id,p1_0.company_id,p1_0.name,p1_0.product_type from product p1_0 where p1_0.id=?
select p1_0.id,p1_0.company_id,p1_0.name,p1_0.product_type from product p1_0 where p1_0.id=1;
```

## application.properties 설정
```yaml
decorator.datasource.p6spy.enable-logging=true # 로깅 사용 여부
```
`decorator.datasource.p6spy.enable-logging` 옵션을 통해 로깅을 사용 여부를 설정할 수 있다. 기본값은 `true`이다.
운영환경에서는 성능을 저하시킬 수 있기 때문에 사용하지 않는 것을 권장한다.

자세한 정보는 [Github](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)에서 확인할 수 있다.

## spy.properties 설정
프로젝트의 resources 디렉토리에 spy.properties 파일을 추가하여 P6Spy의 로깅 기능을 세밀하게 조정할 수 있다.

다음 설정은 로그 포맷을 변경하는 예시이다.

**spy.properties**
```
# spy.properties
appender=com.p6spy.engine.spy.appender.Slf4JLogger
logMessageFormat=com.p6spy.engine.spy.appender.CustomLineFormat
customLogMessageFormat=| %(executionTime) ms | %(sql)
```
```log
2024-07-26T13:12:07.582+09:00  INFO 23300 --- [           main] p6spy                                    : | 1 ms |select p1_0.id,p1_0.company_id,p1_0.name,p1_0.product_type from product p1_0 where p1_0.id=1
```
자세한 정보는 [공식 문서](https://p6spy.readthedocs.io/en/latest/configandusage.html)에서 확인할 수 있다.

## 로그 포맷 커스텀
쿼리를 포매팅해 주는 커스텀 포매터를 만들어보자. 쿼리 포매팅에는 Hibernate의 formatter를 사용했다.
```java
public class P6spyPrettySqlFormat implements MessageFormattingStrategy {
    @Override
    public String formatMessage(int connectionId, String now, long elapsed, String category, String prepared, String sql, String url) {
        sql = formatSql(category, sql);
        return now + " | " + elapsed + "ms | " + category + " | connection " + connectionId + " | " + P6Util.singleLine(prepared) + sql;
    }

    private String formatSql(String category, String sql) {
        if (sql == null || sql.isBlank()) return sql;

        if (Category.STATEMENT.getName().equals(category)) {
            String tempSql = sql.trim().toLowerCase(Locale.ROOT);
            if (tempSql.startsWith("create") || tempSql.startsWith("alter") || tempSql.startsWith("comment")) {
                sql = FormatStyle.DDL.getFormatter().format(sql);
            } else {
                sql = FormatStyle.BASIC.getFormatter().format(sql);
            }
        }
        return " |\nFormatSql(P6Spy sql,Hibernate format):" + sql;
    }
}
```
포매터는 다음과 같이 적용할 수 있다.
```java
@Configuration
public class P6spyConfig {
    
    @PostConstruct
    public void setLogMessageFormat() {
        P6SpyOptions.getActiveInstance().setLogMessageFormat(P6spyPrettySqlFormatter.class.getName());
    }
}
```

```log
2024-07-27T21:22:10.727+09:00  INFO 13816 --- [nio-8080-exec-1] p6spy                                    : p6spy                                    : 1721967338735 | 1ms | statement | connection 6 | select p1_0.id,p1_0.company_id,p1_0.name,p1_0.product_type from product p1_0 where p1_0.id=?|
HeFormatSql(P6Spy sql,Hibernate format):
    select
        p1_0.id,
        p1_0.company_id,
        p1_0.name,
        p1_0.product_type 
    from
        product p1_0 
    where
        p1_0.id=1
```

---
**Reference**<br>
- https://github.com/gavlyukovskiy/spring-boot-data-source-decorator?tab=readme-ov-file
- https://devkuka.tistory.com/303
- https://curiousjinan.tistory.com/entry/spring-boot-3-p6spy-jpa-logging
- https://jessyt.tistory.com/27
