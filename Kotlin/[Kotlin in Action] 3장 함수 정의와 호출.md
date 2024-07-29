# 3 함수 정의와 호출

## 3.1 코틀린에서 컬렉션 만들기
```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

// javaClass는 자바 getClass()에 해당하는 코틀린 코드다.
println(set.javaClass) // class java.util.HashSet
println(list.javaClass) // class java.util.ArrayList
println(map.javaClass) // class java.util.HashMap
```
코틀린은 자체 컬렉션을 제공하지 않고 자바 컬렉션을 사용한다.

## 3.2 함수를 호출하기 쉽게 만들기

자바 컬렉션에는 디폴트 toString 구현이 들어있다.
하지만 그 디폴트 toString의 출력 형식은 고정돼 있고 우리에게 필요한 형식이 아닐 수도 있다.
```kotlin
val list = listOf(1, 2, 3)

// toString() 호출
println(list) // [1, 2, 3]
```
자바 컬렉션의 toString 디폴트 구현과 달리 (1; 2; 3)가 출력되도록 하고 싶다면 어떻게 해야 될까?
이번 절에서는 직접 그런 함수를 구현해 보자.

**joinToString() 함수의 초기 구현**
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```

```kotlin
val list = listOf(1, 2, 3)
println(joinToString(list, ", ", "(", ")")) // (1, 2, 3)
```

### 3.2.1 이름 붙인 인자
```kotlin
println(joinToString(list, separator = ", ", prefix = "(", postfix = ")"))
```
함수에 전달하는 인자 중 일부(또는 전부)의 이름을 명시할 수 있다.
자바로 작성한 함수를 호출할 때는 이름 붙인 인자를 사용할 수 없다.

### 3.2.2 디폴트 파라미터 값
코틀린에서는 함수의 선언에서 파라미터의 디폴트 값을 지정할 수 있다.
자바에서 인자 중 일부가 생략된 오버로딩(overloading) 메서드들이 많아지는 문제를 피할 수 있다.

**디폴트 파라미터 값을 사용해 joinToString() 정의하기**
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ", // 디폴트 값을 지정한다.
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```

```kotlin
val list = listOf(1, 2, 3)

// separator, prefix, postfix 생략
println(joinToString(list)) // 1, 2, 3

// prefix, postfix 생략
println(joinToString(list, "; ")) //1; 2; 3

// 이름 붙은 인자를 사용하는 경우 순서에 관계없이 지정할 수 있다.
println(joinToString(list, postfix = ";", prefix = "# ")) // # 1, 2, 3;
```

### 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
#### 최상위 함수
객체지향 언어인 자바에서는 모든 코드를 클래스의 메서드로 작성해야 한다.
그 결과 Util 클래스처럼 다양한 정적 메서드를 모아두는 역할만 담당하며, 특별한 상태나 인스턴스 메서드는 없는 클래스가 생겨난다.

코틀린에서는 이런 무의미한 클래스가 필요 없다.
대신 함수를 소스 파일의 최상위 수준에 위치시키면 된다.
그런 함수들은 그 파일의 맨 앞에 정의된 패키지의 멤버 함수가 된다.

코틀린 컴파일러가 생성하는 클래스의 이름은 최상위 함수가 들어있던 코틀린 소스 파일의 이름과 대응한다.
> join.kt
```kotlin
package strings

fun joinToString(...): String { ... }
```

```java
/* 자바 */
package strings;

public class JoinKt {
    public static String joinToString(...) { ... }
}
```

#### 최상위 프로퍼티
함수와 마찬가지로 프로퍼티도 최상위 수준에 놓을 수 있다.
최상위 프로퍼티도 다른 프로퍼티처럼 _접근자 메서드를 통해_ 자바 코드에 노출된다.
```kotlin
var opCount = 0 // 최상위 프로퍼티를 선언한다.

fun performOperation() {
    opCount++ // 최상위 프로퍼티의 값을 변경한다.
}

fun reportOperationCount() {
    println("Operation performed $opCount times") // 최상위 프로퍼티 값을 읽는다.
}
```

**상수**를 추가해야 하는 경우 `const` 변경자를 추가하면 프로퍼티를 `public static final` 필드로 컴파일하게 만들 수 있다.
단 원시 타입과 String 타입의 프로퍼티만 const로 지정할 수 있다.

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```

```java
/* 자바 */
public static final String UNIX_LINE_SEPARATOR = "\n";
```

---
**Reference**<br>
- Kotlin in Action
