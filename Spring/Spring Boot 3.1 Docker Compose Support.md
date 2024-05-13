
Spring Boot 3.1부터 Docker Compose에 대한 지원을 시작했다. 프로젝트의 docker compose 파일을 감지하고, 애플리케이션이 시작되거나 중지 될 때 컨테이너를 자동으로 시작/중지해준다.

## 필요사항
- Spring Boot 3.1 이상
- Docker Compose 2.2.0 이상

## Docker Compose 파일

애플리케이션 디렉토리에 compose.yml 파일을 추가한다.

```
services:
  database:
    image: 'mysql:8.0.33'
    ports:
      - '13306:3306'
    environment:
      - 'MYSQL_ROOT_PASSWORD=rootsecret'
      - 'MYSQL_USER=myuser'
      - 'MYSQL_PASSWORD=secret'
      - 'MYSQL_DATABASE=mydatabase'
```

## Dependency

Docker Compose 지원은 'spring-boot-docker-compose' 종속성을 추가하여 활성화할 수 있다. 개발환경에서만 활성화되도록 developmentOnly를 적용시킨다.

>gradle

```
dependencies {
    developmentOnly "org.springframework.boot:spring-boot-docker-compose"
}
```

이 모듈이 종속성으로 포함되면 Spring Boot는 다음을 수행한다.

- 애플리케이션 디렉터리에서 compose.yml 또는 docker-compose.yml 파일을 찾는다.
- 애플리케이션이 실행되면 발견한 docker compose 파일에 대해 docker compose up을 호출한다.
- 지원되는 각 컨테이너에 대한 서비스 연결 Bean 생성한다.
- 애플리케이션이 종료되면 docker compose stop을 호출한다.

만약, 애플리케이션을 시작할 때 Docker Compose 서비스가 이미 실행 중인 경우 Spring Boot는 지원되는 각 컨테이너에 대한 서비스 연결 Bean만 생성하고, 애플리케이션이 종료될 때 docker compose stop을 호출하지 않는다.

<br/>

> 테스트 환경에서 사용할 때
 
spring boot 3.1 사용시developmentOnly만 추가 시 테스트코드에서 사용 불가능한 문제가 있다. testImplementation도 함께 추가하거나, Implementation로 변경하면 작동한다.

    dependencies {
        developmentOnly "org.springframework.boot:spring-boot-docker-compose"
        testImplementation "org.springframework.boot:spring-boot-docker-compose"
    }


 Spring Boot 3.2부터는 Gradle 종속성 구성 testAndDevelopmentOnly를 사용하면 된다.

    dependencies {
        testAndDevelopmentOnly "org.springframework.boot:spring-boot-docker-compose"
    }

## application.yml(properties)

```
spring.docker.compose.file=./docker/compose.yaml
```
compose 파일경로를 지정할 수 있다. 설정하지 않을경우 프로젝트 루트의 compose.yaml(yml), docker-compose.yaml(yml)

<br/>


```
spring.docker.compose.skip.in-tests=true
```
기본적으로 테스트를 실행할 때 Spring Boot의 Docker Compose 지원은 비활성화된다. 이를 활성화하려면 spring.docker.compose.skip.in-tests를 false로 설정해야한다.

<br/>

```
spring.docker.compose.lifecycle-management=start-and-stop
```
수명주기 제어. 다음 값이 지원된다.
- none : docker compose를 시작하거나 중지하지 않는다.
- start-only: 애플리케이션이 시작되면 docker compose를 시작한다.
- start-and-stop :애플리케이션이 시작되면 docker compose를 시작하고, JVM이 종료될때 중지한다.

<br/>

```
spring.docker.compose.start.command=up
```
docker compose up 또는 docker compose start 사용여부 변경

<br/>

```
spring.docker.compose.stop.command=stop
```
docker compose down 또는 docker compose stop 사용여부 변경
