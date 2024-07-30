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


## 3.3 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
확장 함수는 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.
일반 메서드 본문에서 this를 사용할 때와 마찬가지로 확장함수 본문에서도 this를 쓸 수 있다. 또한 this를 생략할 수 있다.
확장 함수 안에서는 클래스 내부에서만 사용할 수 있는 private, protected 멤버를 사용할 수 없다.

```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1) // String: 수신 객체 타입, this: 수신객체

fun String.lastCharV2(): Char = get(length-1) // this를 생략할 수 있다.

println("Kotlin".lastChar()) // String이 수신 객체타입이고 "Kotlin"이 수신 객체다.
// 출력: n
```

### 3.3.1 임포트와 확장 함수

**개별 임포트**
```kotlin
import strings.lastChar
val c = "Kotlin".lastChar()
```
***을 사용한 임포트**
```kotlin
import strings.*
val c = "Kotlin".lastChar()
```
**`as` 키워드를 사용한 임포트**<br>
임포트한 클래스나 함수를 다른 이름으로 부를 수 있다.
확장 함수는 전체 이름(Fully Qualified Name) 사용할 수 없기 때문에 임포트시 이름을 바꾸는 것이 이름 충돌을 해결하는 유일한 방법이다.
```kotlin
import strings.lastChar as last
val c = "Kotlin".last()
```

### 3.3.2 자바에서 확장 함수 호출
내부적으로 확장 함수는 **수신객체를 첫 번째 인자로 받는 정적 메서드**다.
확장함수를 StringUtil.kt 파일에 정의했다면 다음과 같다.
```java
/* 자바 */
char c = StringUtilKt.lastChar("Java");
```

### 3.3.3 확장 함수로 유틸리티 함수 정의
**joinToString()을 확장으로 정의하기**
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```
```kotlin
var list = listOf(1, 2, 3)
println(list.joinToString(separator = "; ", prefix = "(", postfix = ")")) // (1, 2, 3)
```

확장 함수는 단지 정적 메소드 호출에 대한 문법적인 편의일 뿐이다.
그래서 클래스가 아닌 더 구체적인 타입을 수신 객체 타입으로 지정할 수도 있다.
```kotlin
fun Collection<String>.join(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
) = joinToString(separator, prefix, postfix)
```
```kotlin
println(listOf("one", "two", "eight").join(" ")) // one two eight
```

### 3.3.4 확장 함수는 오버라이드할 수 없다
다음과 같은 상속 관계가 있는 경우를 생각해보자
```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button : View() { // Button은 View를 확장한다.
    override fun click() = println("Button clicked")
}
```
```kotlin
val view: View = Button()
view.click() // view에 저장된 실제 타입에 따라 호출할 메서드가 결정된다.
// 출력: Button clicked
```
확장 함수는 첫 번째 인자가 수신객체인 정적 자바 메소드로 컴파일되며, 클래스의 일부가 아니다.
확장 함수를 호출할 때 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 함수가 호출될지 결정되지,
그 변수에 저장된 객체의 동적인 타입에 의해 결정되지 않는다.

```kotlin
fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")
```
```kotlin
val view: View = Button()
view.showOff() // 확장 함수는 정적으로 결정된다.
// 출력: I'm a view!
```

> 어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 멤버함수가 호출된다. (멤버 함수의 우선순위가 더 높다.)

### 3.3.5 확장 프로퍼티
기존 클래스에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다.
확장 프로퍼티는 아무 상태도 가질 수 없다. (기존 클래스의 인스턴스 객체에 필드를 추가할 방법은 없다.)
하지만 프로퍼티 문법으로 더 짧게 코드를 작성할 수 있어서 편한 경우가 있다.

**확장 프로퍼티 선언하기**
```kotlin
val String.lastChar: Char
    get() = get(length - 1)
```
```kotlin
println("Kotlin".lastChar) // n
```

**변경 가능한 확장 프로퍼티 선언하기**
```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
```
```kotlin
val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb) // Kotlin!
```

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

### 3.4.1 자바 컬렉션 API 확장
코틀린 표준 라이브러리는 수 많은 확장 함수를 포함한다.
```kotlin
val strings: List<String> = listOf("first", "second", "fourteenth")
println(strings.last()) // fourteenth

val numbers: Collection<Int> = setOf(1, 14, 2)
println(numbers.max()) // 14
```

### 3.4.2 가변 인자 함수: 인자 개수가 달라질 수 있는 함수 정의

```kotlin
val list = listOf(2, 3, 5, 7, 11)
```
라이브러리에서 이 함수의 정의를 보면 다음과 같다.

```kotlin
fun listOf<T>(vararg values: T): List<T> { ... }
```
파라미터 앞에 `vararg` 변경자를 붙여서 가변 길이 인자를 선언할 수 있다.

배열에 들어있는 원소를 가변길이 인자로 넘길때는 배열 앞에 `*`를 붙이면 된다.
```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args) // 스프레드 연산자가 배열의 내용을 펼쳐준다.
    println(list)
}
```

### 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```
이 코드는 **중위 호출**(infix call)이라는 특별한 방식으로 to라는 일반 메소드를 호출한 것이다.
```kotlin
1.to("one") // to 메서드를 일반적인 방식으로 호출함
1 to "one" // to 메서드를 중위 호출 방식으로 호출함
```
인자가 하나뿐인 일반메서드나 확장메서드에 중위 호출을 사용할 수 있다.
중위 호출을 사용하게 허용하고 싶다면 `infix` 변경자를 메서드 선언 앞에 추가 해야 한다.

다음은 to 함수의 정의를 간략하게 줄인 코드이다.
```kotlin
infix fun Any.to(other:Any) = Pair(this, other)
```
Pair는 코틀린 표준 라이브러리 클래스로, 두 원소로 이루어진 순서쌍을 표현한다.
```kotlin
fun main(){
    val (number, name) = 1 to "one" // to 함수를 통해 순서쌍을 만들고, 구조 분해를 통해 그 순서쌍을 푼다
    println("number: $number") // number: 1
    println("name: $name") // name: one
}
```
Pair의 내용으로 두 변수를 즉시 초기화 할 수 있다.
이런 기능을 **구조 분해 선언**(destructuring declaration) 이라고 부른다.

---
**Reference**<br>
- Kotlin in Action
