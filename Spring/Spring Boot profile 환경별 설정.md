# Profiles

배포 환경별(local, develop, production 등)로 DB, 외부 API 연동, 로그 레벨 등이 달라야 할 때가 있다.
Spring Boot의 Profile을 이용하여 환경별 설정값들을 미리 정의해놓고 환경에 따라 다른 설정이 적용되도록 할 수 있다.

## 환경별 설정 작성하기
Spring Boot 2.4부터는 물리적인 파일 하나를 여러 논리적 document로 분할 할 수 있다.
yaml 파일에서는 `---`, properties 파일에서는 `#---`을 구분자로 문서를 나눌 수 있다.

document는 위에서 아래 순으로 처리되기 때문에 뒤에 있는 document는 앞에 있는 document를 오버라이드한다.

- `spring.profiles.active` : 활성화할 profile을 지정한다.
- `spring.profiles.default` : 디폴트 profile을 지정한다. (기본값: default)
- `spring.profiles.group` : profile 그룹을 정의한다. 여러 profile을 하나의 profile로 만들 수 있다.
- `spring.config.activate.on-profile` : 특정 프로파일이 활성화될 때만 해당 설정이 적용되도록 할 수 있다.

```yaml
#application.yml
spring:
  profiles:
    #active: dev
    #default: dev
    group:
      dev:
      test: common
      prod: common
default:
  string: default property
---
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:~/test
    username: dev
    password:
server:
  port: 7777
---
spring:
  config:
    activate:
      on-profile: test
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test
    username: test
    password: test1234!
server:
  port: 8888
---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/prod
    username: prod
    password: prod1234!
server:
  port: 9999
---
spring:
  config:
    activate:
      on-profile: common
common:
  string: common property
```

---
**Reference**<br>
- https://docs.spring.io/spring-boot/reference/features/profiles.html
- https://colabear754.tistory.com/112
- https://jojoldu.tistory.com/547
