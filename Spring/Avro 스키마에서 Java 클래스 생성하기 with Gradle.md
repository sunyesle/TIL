# Avro 스키마에서 Java 클래스 생성하기 with Gradle

## Apache Avro
Avro는 Apache에서 만든 프레임워크로 데이터 직렬화 기능을 제공한다.
JSON(*.avsc) 형식의 스키마를 통해 데이터 구조를 정의할 수 있다.

## 스키마 정의
```json
{
  "namespace": "example.avro",
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "name", "type": "string"},
    {"name": "active", "type": "boolean"},
    {"name": "lastUpdatedBy", "type": ["null", "string"], "default": null}
  ]
}
```

## Gradle 설정
`generateAvro` Gradle 태스크를 실행하면 `src/main/resources/avro` 폴더 내의 `.avsc` 파일이 Java 코드로 변환된다.
```groovy
import com.github.davidmc24.gradle.plugin.avro.GenerateAvroJavaTask

plugins {
    id 'java'
    id 'com.github.davidmc24.gradle.plugin.avro-base' version '1.9.1'
}

dependencies {
    // Avro 형식을 처리하는 데 필요한 의존성
    implementation 'org.apache.avro:avro:1.11.4'
}

def generateAvro = tasks.register("generateAvro", GenerateAvroJavaTask) {
    source("src/main/resources/avro") // Avro 스키마 파일 위치 (*.avsc)
    outputDir = file("build/generated") // 생성된 Java 파일 출력 디렉토리
}

sourceSets {
    getByName("main") {
        // 생성된 Java 파일 디렉토리를 컴파일 대상에 추가
        java.srcDir("$buildDir/generated")
    }
}

tasks.named("compileJava") {
    dependsOn(generateAvro)
}
```

---
**Reference**<br>
- https://github.com/davidmc24/gradle-avro-plugin?tab=readme-ov-file#alternate-usage
- https://avro.apache.org/docs/1.12.0/getting-started-java/
- https://www.baeldung.com/java-gradle-avro-schema
- https://velog.io/@itbuddy/Avro-Gradle-source-outputDir-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A7%80%EC%A0%95-h7s09r21
