# Spring Boot MyBatis 다중 데이터베이스 연결 설정

Spring Boot MyBatis의 다중 데이터베이스 설정 방법을 알아보자.

단일 데이터베이스를 사용할 경우 application.yml 파일에 DB 연결 정보만 작성하면 Spring Boot에서 자동 구성해 주지만,
다중 데이터베이스를 사용할 때는 직접 설정 파일을 작성해야 한다.

## application.yml
DB 연결 정보를 정의한다. url이 아니라 jdbc-url인 것에 주의하자.

> HikariCP에서는 jdbcUrl 옵션을 사용하기 때문에 DataSource를 수동으로 설정할 경우 jdbc-url으로 변경해야 한다.
[(HikariCP 문서)](https://github.com/brettwooldridge/HikariCP)

```yml
spring:
  datasource:
    primary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://localhost:13306/primary
      username: test
      password: test1!
    secondary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://localhost:13306/secondary
      username: test
      password: test1!
```

## 설정 파일 작성
### DataSource 설정
기본으로 사용할 `DataSource`에 `@Primary` 어노테이션을 붙여준다.
```java
@Configuration
public class DataSourceConfig {

    @Primary
    @Bean(name = "primaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

### Primary DB Mybatis 설정
기본으로 사용할 `SqlSessionFactory`와 `SqlSessionTemplate`에 `@Primary` 어노테이션을 붙여준다.
```java
@Configuration
@MapperScan(
        basePackages = "com.sunyesle.spring_boot_mybatis",
        sqlSessionFactoryRef = "primarySqlSessionFactory",
        annotationClass = PrimaryDB.class // MapperScan 시 사용할 어노테이션 지정
)
public class PrimaryMyBatisConfig {

    @Primary
    @Bean(name = "primarySqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("primaryDataSource") DataSource dataSource, ApplicationContext applicationContext) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setMapperLocations(applicationContext.getResources("classpath:mapper/*.xml"));
        sessionFactory.setTypeAliasesPackage("com.sunyesle.spring_boot_mybatis.**.vo");
        return sessionFactory.getObject();
    }

    @Primary
    @Bean(name = "primarySqlSessionTemplate")
    public SqlSessionTemplate primarySqlSessionTemplate(@Qualifier("primarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

#### 커스텀 어노테이션 추가
MyBatis Mapper 인터페이스에서 `@Mapper` 대신 사용할 어노테이션을 정의한다.
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface PrimaryDB {
}
```

다음과 같이 Mapper 인터페이스에서 `@PrimaryDB` 어노테이션을 사용하여, Primary DB에 매핑되도록 설정할 수 있다.
```java
@PrimaryDB
public interface OrderMapper {
}
```

### Secondary DB Mybatis 설정
```java
@Configuration
@MapperScan(
        basePackages = "com.sunyesle.spring_boot_mybatis",
        sqlSessionFactoryRef = "secondarySqlSessionFactory",
        annotationClass = SecondaryDB.class
)
public class SecondaryMyBatisConfig {

    @Bean(name = "secondarySqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("secondaryDataSource") DataSource dataSource, ApplicationContext applicationContext) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setMapperLocations(applicationContext.getResources("classpath:mapper/*.xml"));
        sessionFactory.setTypeAliasesPackage("com.sunyesle.spring_boot_mybatis.**.vo");
        return sessionFactory.getObject();
    }

    @Bean(name = "secondarySqlSessionTemplate")
    public SqlSessionTemplate secondarySqlSessionTemplate(@Qualifier("secondarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

#### 커스텀 어노테이션 추가
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface SecondaryDB {
}
```

---
**Reference**<br>
- https://tweety1121.tistory.com/entry/Spring-boot-multiple-database-%EC%84%A4%EC%A0%95-mybatishikari
