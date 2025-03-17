# Log4j2 설정

## build.gradle
**Log4j2 의존성 추가**
```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-log4j2'
}
```

**Logback 의존성 제거**<br>
Spring에서는 기본적으로 Logback을 이용해서 로깅을 하기 때문에 Log4j2를 적용하기 위해서는 Logback 라이브러리를 제거해야 한다.
```gradle
configurations {
	configureEach {
		exclude group: 'org.springframework.boot', module: 'spring-boot-starter-logging'
	}
}
```

## application.yml
log4j2 설정 파일 경로를 지정한다.
```yml
logging:
  config: classpath:log4j2.xml
```

## log4j2.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>

    <!-- Property를 설정하여, xml 파일 내부의 변수로 사용할 수 있다. 변수는 ${name} 형태로 사용할 수 있다. -->
    <Properties>
        <Property name="logPath">./logs</Property>
        <Property name="consoleLogPattern">%d{yyyy-MM-dd HH:mm:ss.SSS} %style{%-5p}{green} [%15t] %style{%-40c{1.}}{cyan} : %m%n</Property>
        <Property name="fileLogPattern">%d{yyyy-MM-dd HH:mm:ss.SSS} %-5p [%t] %c{1.} : %m%n</Property>
        <Property name="serviceName">application</Property>
    </Properties>

    <!-- Appender는 로그 메시지를 특정 위치에 전달해 주는 역할을 한다. -->
    <Appenders>
        <!-- <Console>: 콘솔에 로그를 출력하는 Appender를 설정한다. -->
        <!--    <PatternLayout>: 출력되는 로그의 형식을 설정한다. -->
        <Console name="console">
            <PatternLayout pattern="${consoleLogPattern}" disableAnsi="false" />
        </Console>

        <!-- <RollingFile>: 파일에 로그를 기록하는 Appender를 정의한다. Rolling file appender는 파일의 크기 또는 시간에 따라 새로운 파일을 생성한다. -->
        <!-- fileName: 로그 파일의 경로와 이름 -->
        <!-- filePattern: rolling file 패턴 -->
        <!--    <Policies>: 로그 파일의 롤오버 시점을 설정한다. -->
        <!--        <SizeBasedTriggeringPolicy>: 파일 크기 기반 롤오버 정책 -->
        <!--        <TimeBasedTriggeringPolicy>: 시간 기반 롤오버 정책 -->
        <!--    <DefaultRolloverStrategy>: 로그 파일의 롤오버 전략을 설정한다. -->
        <!--        <Delete>: 로그 파일 삭제 전략을 설정한다. -->
        <!--            <IfLastModified>: 파일의 마지막 수정일을 기준으로 파일을 삭제한다. -->
        <RollingFile name="file" append="true" fileName="${logPath}/${serviceName}.log" filePattern="${logPath}/${serviceName}.%d{yyyy-MM-dd}.%i.log.gz">
            <PatternLayout pattern="${fileLogPattern}" disableAnsi="true"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="10MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy>
                <Delete basePath="${logPath}">
                    <IfLastModified age="15d"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>
    </Appenders>

    <!-- 로그 레벨을 설정하고, 로그 이벤트를 특정 Appender에 연결한다. -->
    <Loggers>
        <Logger name="org.springframework" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
            <AppenderRef ref="file" />
        </Logger>
        <Logger name="com.sunyesle.spring_boot_logging" level="DEBUG" additivity="false" >
            <AppenderRef ref="console" />
            <AppenderRef ref="file" />
        </Logger>
        <Root level="INFO" additivity="false" >
            <AppenderRef ref="console" />
            <AppenderRef ref="file" />
        </Root>
    </Loggers>

</Configuration>
```

### PatternLayout
| 패턴                            | 설명                                                | garbage-free|
|-------------------------------|---------------------------------------------------|-------------|
| `%d`                          | 로그 이벤트가 발생한 시각 (기본 형식: `yyyy-MM-dd HH:mm:ss.SSS`) | O |
| `%p`                          | 로그 레벨 (`INFO`, `DEBUG`, `ERROR` 등)                | O |
| `%c`                          | 로거의 이름 (패키지명 포함)                                  | O |
| `%t`                          | 현재 실행 중인 스레드 이름                                   | O |
| `%m`                          | 로그 메시지                                            | O |
| `%n`                          | 줄 바꿈 (`\n`)                                       | O |
| `%r`                          | 애플리케이션 시작 이후 경과한 시간 (밀리초)                         | O |
| `%x`                          | NDC (Nested Diagnostic Context)                   | O |
| `%X{key}`                     | MDC (Mapped Diagnostic Context)에서 특정 key 값 출력     | O |
| `%F`                          | 소스 파일명                                            | X |
| `%l`                          | 로그 발생 위치 (`클래스명.메서드명(파일명:줄번호)`)                   | X |
| `%C`                          | 클래스명                                              | X |
| `%M`                          | 메서드명                                              | X |
| `%L`                          | 로그가 발생한 코드의 줄 번호                                  | X |
| `%style{pattern}{ANSI style}` | ANSI를 사용해 스타일링                                    | O |
| `%highlight{pattern}{style}`  | 특정 로그 레벨에 따라 색상을 적용                               | O |

---
**Reference**
- https://medium.com/@201924576/spring-boot-log4j2-%EC%84%A4%EC%A0%95-21f3b6da38c6
- https://logging.apache.org/log4j/2.x/manual/configuration.html
- https://logging.apache.org/log4j/2.x/manual/pattern-layout.html
