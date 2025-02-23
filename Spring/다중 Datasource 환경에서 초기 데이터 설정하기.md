# 다중 Datasource 환경에서 초기 데이터 설정하기

`DataSourceInitializer` 빈을 등록하여 `DataSource`를 초기화할 수 있다.

`@Qualifier` 어노테이션으로 초기화할 `DataSource`를 지정한다.

```java
@Bean
@Profile("test") // test 프로파일에서만 활성화되도록 한다.
public DataSourceInitializer dataSourceInitializer(@Qualifier("primaryDataSource") DataSource datasource) {
    ResourceDatabasePopulator resourceDatabasePopulator = new ResourceDatabasePopulator();
    resourceDatabasePopulator.addScript(new ClassPathResource("sql/primary/schema.sql"));
    resourceDatabasePopulator.addScript(new ClassPathResource("sql/primary/data.sql"));

    DataSourceInitializer dataSourceInitializer = new DataSourceInitializer();
    dataSourceInitializer.setDataSource(datasource);
    dataSourceInitializer.setDatabasePopulator(resourceDatabasePopulator);
    return dataSourceInitializer;
}
```

---
**Reference**<br>
- https://jaehoney.tistory.com/294
- https://revi1337.tistory.com/248
- https://github.com/arawn/spring-examples/blob/master/data-access/datasource-initializer/README.md
