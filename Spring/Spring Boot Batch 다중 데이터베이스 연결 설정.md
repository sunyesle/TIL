# Spring Boot Batch 다중 데이터베이스 연결 설정

Spring Boot Batch의 메타 데이터를 저장하는 DB와 서비스 데이터를 저장하는 DB를 분리하기 위해,
다중 데이터베이스 설정 방법을 알아보자.

Spring Batch 5 기준으로 작성되었다. (Spring boot 3)

단일 데이터베이스를 사용할 경우 application.yml 파일에 DB 연결 정보만 작성하면 Spring Boot에서 자동 구성해 주지만,
다중 데이터베이스를 사용할 때는 직접 설정 파일을 작성해야 한다.

## DB 세팅
MySQL에서 데이터베이스를 생성한다.

meta_db에 Spring Boot Batch의 메타 데이터를 data_db에 서비스 데이터를 저장할 것이다.
```sql
CREATE DATABASE IF NOT EXISTS `meta_db`;
CREATE DATABASE IF NOT EXISTS `data_db`;
```

## application.properties
DB 연결 정보를 정의한다. url이 아니라 jdbc-url인 것에 주의하자.

> HikariCP에서는 jdbcUrl 옵션을 사용하기 때문에 DataSource를 수동으로 설정할 경우 jdbc-url으로 변경해야 한다.
[(HikariCP 문서)](https://github.com/brettwooldridge/HikariCP)

```yml
spring:
  datasource:
    meta-db:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://localhost:3306/meta_db
      username: test
      password: test1!
    data-db:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://localhost:3306/data_db
      username: test
      password: test1!
  batch:
    jdbc:
      initialize-schema: always
```

## 설정 파일 작성
### Batch 메타 데이터 DB 설정
배치 메타 데이터용 DataSource와 PlatformTransactionManager에 @Primary를 붙여준다.
```java
@Configuration
public class MetaDBConfig {

    @Primary
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.meta-db")
    public DataSource metaDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean
    public PlatformTransactionManager metaTransactionManager() {
        return new DataSourceTransactionManager(metaDataSource());
    }
}
```

### 서비스 데이터 DB 설정
JPA를 사용할 예정이기 때문에 관련 설정도 함께 진행한다.

EntityManagerFactory와 TransactionManager 빈을 생성하고, @EnableJpaRepositories 어노테이션에 지정해 준다.
```java
@Configuration
@EnableJpaRepositories(
        basePackages = "com.sunyesle.spring_boot_batch",
        entityManagerFactoryRef = "dataEntityManager",
        transactionManagerRef = "dataTransactionManager"
)
public class DataDBConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.data-db")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean dataEntityManager() {
        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();

        em.setDataSource(dataSource());
        em.setPackagesToScan("com.sunyesle.spring_boot_batch");
        em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());

        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto", "create");
        properties.put("hibernate.show_sql", "true");
        properties.put("hibernate.physical_naming_strategy", CamelCaseToUnderscoresNamingStrategy.class.getName());
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Bean
    public PlatformTransactionManager dataTransactionManager() {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(dataEntityManager().getObject());
        return transactionManager;
    }
}
```

---
**Reference**<br>
- https://charliezip.tistory.com/33
- https://milenote.tistory.com/171
- https://ride-dev.tistory.com/97
