# Maven으로 의존성 jar 다운받기

**1. Maven을 다운받는다.**<br>
https://maven.apache.org/download.cgi

**2. 라이브러리를 다운받을 폴더를 생성하고, 폴더 안에 pom.xml 파일을 생성한다.**
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>
        <!-- 여기에 다운받을 의존성 추가 -->
    </dependencies>
</project>
```

**3. pom.xml 파일이 있는 곳에서 cmd로 명령어를 입력한다.**

> 의존성 포함된 모든 라이브러리를 다운로드
```
mvn dependency:copy-dependencies -DoutputDirectory=./libs
```

<br>

> 의존성 라이브러리를 제외하고 지정된 라이브러리만 다운로드
```
mvn dependency:copy-dependencies -DexcludeTransitive=true -DoutputDirectory=./libs
```

---
**Reference**
- https://atl.kr/dokuwiki/doku.php/maven%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC_jar_%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C
