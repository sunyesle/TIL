# Gradle Java Plugin

자바 프로젝트를 빌드하고 관리하기 위한 작업(task)들을 제공하는 그레이들 내장 플러그인이다.

## 사용법
빌드 스크립트에 다음 내용을 추가하여 사용할 수 있다.

> build.gradle
```
plugins {
    id 'java'
}
```

## 의존성 관리

의존성은 컴파일 시에만 필요할 수도 있고, 컴파일과 런타임 시점에 모두 필요할 수도 있다. 또는 테스트를 실행할 때만 필요할 수도 있다.

Gradle에서는 **의존성 구성**을 통해 의존성들의 범위(scope)를 설정할 수 있다.

### Classpath
JVM 혹은 자바 컴파일러가 Java 프로그램을 컴파일하거나 실행할 때, 필요한 클래스를 탐색하는데 기준이 되는 경로를 말한다.

- ``compileClasspath``: 프로젝트의 소스코드를 컴파일할 때 필요한 클래스 경로
- ``runtimeClasspath``: 프로젝트를 실행할 때 필요한 클래스 경로

### 의존성 구성

- ``implementation``: compileClasspath와 runtimeClasspath에 추가
- ``compileOnly``: compileClasspath에 추가. 빌드 결과물에는 포함되지 않는다.
- ``runtimeOnly``: runtimeClasspath에 추가


#### 그 외
- ``annotationProcessor``: 어노테이션 기반 라이브러리를 컴파일러가 인식할 수 있도록 추가 (Lombok, QueryDSL, ConfigurationProperties 등)
- ``testImplementation``, ``testCompileOnly``, ``testRuntimeOnly``:
테스트 소스에 대한 의존성 구성도 별도로 존재한다.

> build.gradle
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.mysql:mysql-connector-j'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'com.h2database:h2'
}
```

---
### Reference

https://docs.gradle.org/current/userguide/java_plugin.html

https://docs.gradle.org/current/userguide/java_library_plugin.html

https://mangkyu.tistory.com/296

https://ttl-blog.tistory.com/1271

https://freestrokes.tistory.com/108