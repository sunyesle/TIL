## Gradle Java Library Plugin

Java Library 플러그인은 Java 플러그인을 확장하여 추가적인 기능을 제공한다.
> build.gradle
```
plugins {
    id 'java-library'
}
```
Java 플러그인과 Java Library 플러그인의 주요 차이점은 소비자에게 노출되는 API 개념을 도입한다는 것이다. 종속성 구성 ``api``를 제공한다.
> build.gradle
```
dependencies {
    api 'org.apache.httpcomponents:httpclient:4.5.7'
    implementation 'org.apache.commons:commons-lang3:3.5'
}
```

## implentation과 api의 차이

``implementation``과 ``api`` 모두 compileClasspath와 runtimeClasspath에 추가된다. 하지만 둘 사이에는 전이 의존성(transitive dependency)의 컴파일 경로 노출 여부에서 차이가 있다.

``api``로 선언된 의존성은 **소비자에게 전이적으로 노출된다.** 소비자의 compileClasspath와 runtimeClasspath 모두에 포함된다.

``implemetation``로 선언된 의존성은 **소비자에게 노출되지 않는다.** 소비자의 runtimeClasspath에만 추가되고 compileClasspath에는 포함되지 않는다.

즉, ``api``는 의존성을 소비자의 컴파일 타임 종속성에 노출하고 ``implementation``은 노출하지 않는다.

### 예시

Project X가 있고, 이는 Project A와 Project B에 의존한다. 
> Project X의 build.gradle
```
dependencies {
    api 'a'
    implemetation 'b'
}
```
그리고 Project C는 Project X의 소비자이다.
> Project C의 build.gradle
```
dependencies {
    implemetation 'x'
}
```
이때 Project C의 classpath는 다음과 같다.

- compileClasspath: X, A
- runtimeClasspath: X, A, B

Project X에서 api를 통해 의존한 A는 compileClasspath에 추가되었다. (의존성 전이)

반면, implemetation을 통해 의존한 B는 compileClasspath에 추가되지 않았기 때문에 Project C의 코드에서 접근하려고 하면 컴파일 에러가 발생한다.

## implentation과 api의 사용

**가능하다면 api보다는 implementation을 사용하는 것이 좋다.**

implementation 사용은 다음과 같은 장점이 있다.
- 의존성이 소비자의 compileClasspath로 노출되지 않으므로, 실수로 전이적 의존성에 의존하는 일을 방지할 수 있다.
- compileClasspath 크기가 줄어 더 빨리 컴파일할 수 있다.
- 라이브러리 측에서 의존성을 변경해도 소비자는 재컴파일하지 않아도 된다.

api는 애플리케이션 바이너리 인터페이스(ABI, Application Binary Interface)로 다음과 같은 경우에 사용하면 좋다.

- 슈퍼클래스나 인터페이스에 사용되는 타입
- 제너릭 타입을 포함하여 public 메소드 파라미터로 사용하는 타입
- public 필드에 사용되는 타입
- public 어노테이션 타입

---
### Reference

https://docs.gradle.org/current/userguide/java_library_plugin.html

https://mangkyu.tistory.com/296

https://ttl-blog.tistory.com/1275

https://jongmin92.github.io/2019/12/07/Gradle/gradle-implementation/

https://reflectoring.io/gradle-pollution-free-dependencies/
