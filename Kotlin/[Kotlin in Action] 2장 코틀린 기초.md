# 2 코틀린 기초

## 2.1 기본 요소: 함수와 변수

### 2.1.1 Hello, world!
```kotlin
fun main(args: Array<String>) {
    println("Hello, world!")
}
```
- 함수를 선언할 때 fun 키워드를 사용한다.
- 파라미터 이름 뒤에 그 파라미터의 타입을 쓴다.
- 함수를 최상위 수준에서 선언할 수 있다.
- 배열은 Array 클래스로 표현된다.
- 자바 표준 라이브러리 함수를 간결하게 사용할 수 있게 감싼 래퍼(wrapper)를 제공한다.
- 줄 끝에 세미콜론(;)을 붙이지 않아도 된다.

### 2.1.2 함수

**블록이 본문인 함수**<br>
```kotlin
fun max(a: Int, b: Int): Int {
return if (a > b) a else b
}
```

**식이 본문인 함수**<br>
리턴타입을 생략할 수 있다.
```kotlin
fun max2(a: Int, b: Int) = if (a > b) a else b
```
코틀린의 if는 문이 아닌 **식**이다.
- 식(expression): 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여할 수 있음.
- 문(statement): 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재하며 아무런 값을 만들어내지 않음.

### 2.1.3 변수
```kotlin
val answer = 42 // 타입 지정 생략 가능
val answer2: Int = 42 // 변수명 뒤에 타입 명시
val answer3: Int // 초기화식 없이 변수를 선언하려면 타입을 명시해야한다.
answer3 = 42

val a = 1 // value. 변경 불가능한(immutable) 참조. 자바의 final 변수
var b = 1 // variable. 변경 가능한(mutable) 참조. 자바의 일반 변수
```
기본적으로 모든변수를 val로 선언하고 필요할때만 var로 변경하자.
변경 불가능한 참조와 변경 불가능한 객체를 부수 효과가 없는 함수와 조합해 사용하면 코드가 함수형 코드에 가까워진다.

### 2.1.4 문자열 템플릿
변수이름 앞에 $를 붙여서 문자열 템플릿을 사용할 수 있다. 자바의 문자열 접합 연산과 동일하지만 좀 더 간결하다.
```kotlin
val name = "Kotlin"
println("Hello, $name!") // "Hello, " + name + "!"와 동일
println("Hello, ${name}!")
```

중괄호(`{}`)로 둘러싸서 식을 문자열 템플릿 안에 넣을 수 있다.
```kotlin
fun main(args: Array<String>) {
    println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
}
```

## 2.2 클래스와 프로퍼티

### 2.2.1 프로퍼티
```kotlin
class Person(
    val name: String, // 읽기 전용 프로퍼티로, 코틀린은 (비공개)필드, (공개)getter를 만들어낸다.
    var isMarried: Boolean // 쓸 수 있는 프로퍼티로, 코틀린은 (비공개)필드, (공개)getter, (공개)setter를 만들어 낸다.
)

fun main() {
    val person = Person("Bob", true)
    println(person.name) // getter 사용
    person.isMarried = false // setter 사용
}
```

### 2.2.2 커스텀 접근자
```kotlin
class Rectangle(
    val height: Int,
    val width: Int
) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

fun main() {
    val rectangle = Rectangle(41, 43)
    println(rectangle.isSquare) // false
}
```

### 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지
파일 안에 있는 모든 선언(클래스, 함수, 프로퍼티 등)이 해당 패키지에 들어간다.

자바와 달리 패키지 구조와 디렉터리 구조가 맞아 떨어질 필요는 없지만, 대부분의 경우 자바와 같이 패키지별로 디렉터리를 구성하는 편이 낫다.
특히 자바와 코틀린을 같이 사용하는 프로젝트에서는 자바의 양식을 따르는게 중요하다.
하지만 여러 클래스를 한 파일에 넣는 것을 주저해서는 안된다.
```kotlin
package geometry.shapes  // 패키지 선언

import java.util.Random

class Rectangle(val height:Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

### 2.3.1 enum 클래스 정의
```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

```kotlin
enum class Color(
    var r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238); // 반드시 세미콜론을 사용해야 한다.

    fun rgb() = (r * 256 + g) * 256 + b;
}

fun main() {
    println(Color.BLUE.rgb()) // 255
}
```

### 2.3.2 when으로 enum 클래스 다루기
if와 마찬가지로 when도 값을 만들어내는 식이다.
```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Rechard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }

fun main() {
    println(getMnemonic(Color.BLUE)) // Battle
}
```

한 분기에 여러 값을 매치 패턴으로 사용할 경우 값 사이를 콤마(`,`)로 분리한다.
```kotlin
fun getWarmth(color: Color) = when (color) {
    Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
    Color.GREEN -> "neutral"
    Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
}

fun main() {
    println(getMnemonic(Color.BLUE)) // cold
}
```

enum 상수 값을 임포트해서 코드를 더 간단하게 만들 수 있다.
```kotlin
import geometry.ch02.Color // 다른 패키지에서 정의한 Color 클래스를 임포트한다.
import geometry.ch02.Color.* // 짧은 이름으로 사용하기 위해 enum 상수를 모두 임포트한다.

fun getWarmth(color: Color) = when (color) {
    RED, ORANGE, YELLOW -> "warm"
    GREEN -> "neutral"
    BLUE, INDIGO, VIOLET -> "cold"
}
```

### 2.3.3 when과 임의의 객체를 함께 사용
```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")
    }

fun main() {
    println(mix(BLUE, YELLOW)) // GREEN
}
```

### 2.3.4 인자가 없는 when 사용
when에 아무 인자도 없으려면 각 분기의 조건이 boolean 결과를 계산하는 식이어야 한다.
추가 객체를 만들지 않는다는 장점이 있지만 가독성은 더 떨어진다.
```kotlin
fun mixOptimized(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
        (c1 == YELLOW && c2 == BLUE) || (c1 == BLUE && c2 == YELLOW) -> GREEN
        (c1 == BLUE && c2 == VIOLET) || (c1 == VIOLET && c2 == BLUE) -> INDIGO
        else -> throw Exception("Dirty color")
    }
```

### 2.3.5 스마트 캐스트: 타입 검사과 타입 캐스트를 조합
예제로 (1 + 2) + 4 와 같은 간단한 산술식을 계산하는 함수를 만들어보자.
```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```
이제 식의 값을 어떻게 계산하는지 살펴보자. 살펴본 예를 평가한 값은 7이어야 한다.
```kotlin
println(eval(Sum(Sum(Num(1), Num(2)), Num(4))))
```

**if 연쇄를 사용해 식을 계산하기**
```kotlin
fun eval(e: Expr): Int {
    if (e is Num) {
        val n = e as Num // 불필요한 타입 캐스팅
        return n.value
    }
    if (e is Sum) {
        return eval(e.left) + eval(e.right)
    }
    throw IllegalArgumentException("Unknown expression")
}

fun main() {
    println(eval(Sum(Sum(Num(1), Num(2)), Num(4)))) // 7
}
```
코틀린에서는 `is`를 사용해 변수타입을 검사한다.
`is`로 검사하고나면 컴파일러가 캐스팅을 수행해준다.
이를 **스마트 캐스트**(smart cast)라고 부른다.

스마트캐스트는 `is`로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다.
클래스의 프로퍼티에 대해 스마트 캐스트를 사용한다면 그 프로퍼티는 반드시 `val`이어야 하며 커스텀 접근자를 사용하는 것이어도 안된다.

원하는 타입으로 명시적으로 타입캐스팅하려면 `as`키워드를 사용한다.

### 2.3.6 리팩토링: if를 when으로 변경

**값을 만들어내는 if 식**
```kotlin
fun evalV1(e: Expr): Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.left) + eval(e.right)
    } else{
        throw IllegalArgumentException("Unknown expression")
    }
```

**if 중첩 대신 when 사용하기**
```kotlin
fun evalV2(e: Expr): Int =
    when (e) {
        is Num ->
            e.value
        is Sum ->
            eval(e.left) + eval(e.right)
        else ->
            throw IllegalArgumentException("Unknown expression")
    }
```

### 2.3.7 if와 when의 분기에서 블록 사용
블록의 맨 마지막에 그 분기의 결과값을 위치시키면 된다.
```kotlin
fun evalWithLogging(e: Expr): Int =
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value // 블록의 맨 마지막에 그 분기의 결과값을 위치시키면 된다.
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left + $right")
            eval(e.right) + eval(e.left)
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }

fun main() {
    println(evalWithLogging(Sum(Sum(Num(1), Num(2)), Num(4))))
    /* 출력
    num: 1
    num: 2
    sum: 1 + 2
    num: 4
    sum: 3 + 4
    7
    */
}

```

## 2.4 대상을 이터레이션: while과 for 루프

### 2.4.1 while 루프
자바와 동일하다.

### 2.4.2 수에 대한 이터레이션: 범위와 수열
for은 자바의 for-each 루프에 해당하는 형태만 존재한다.
코틀린에서는 초깃값, 증가 값, 최종 값을 사용한 루프를 대신하기 위해 범위를 사용한다.

```kotlin
val oneToTen = 1..10
```
범위(range)는 기본적으로 두 값으로 이루어진 구간이다.
코틀린의 범위는 폐구간(양 끝을 포함하는 구간)이다.

```kotlin
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "FizzBuzz "
    i % 3 == 0 -> "Fizz "
    i % 5 == 0 -> "Buzz "
    else -> "$i "
}

fun main() {
    for (i in 1..100) {
        print(fizzBuzz(i))
    }
    /* 출력
    1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz 16 17 ...
    */

    // downTo 역방향 수열을 만든다
    // step 증가값의 절댓값이 2로 바뀐다
    for (i in 100 downTo 1 step 2) {
        print(fizzBuzz(i))
    }
    /* 출력
    Buzz 98 Fizz 94 92 FizzBuzz 88 86 Fizz 82 Buzz Fizz 76 74 ...
    */
}
```

### 2.4.3 맵에 대한 이터레이션
```kotlin
fun main5() {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F') { // A부터 F까지 문자의 범위를 사용해 이터레이션한다.
        val binary = Integer.toBinaryString(c.toInt()) // 아스키(ASCII) 코드를 2진 표현으로 바꾼다.
        binaryReps[c] = binary // c를 키로 c의 2진 표현을 맵에 넣는다.
    }
    for ((letter, binary) in binaryReps) { // 맵에 대해 이터레이션 한다. 맵의 키와 값을 두 변수에 각각 대입한다.
        println("$letter = $binary")
    }
    /* 출력
    A = 1000001
    B = 1000010
    C = 1000011
    D = 1000100
    E = 1000101
    F = 1000110
    */

    val list = arrayListOf("10", "11", "1001")
    for ((index, element) in list.withIndex()) { // 인덱스와 함께 컬렉션을 이터레이션한다.
        println("$index: $element")
    }
    /* 출력
    0: 10
    1: 11
    2: 1001
    */
}
```

### 2.4.4 in으로 컬렉션이나 범위의 원소 검사
어떤 값이 범위나 컬렉션에 들어있는지 알고 싶을 때도 in을 사용한다.
```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

fun main() {
    println(isLetter('q')) // true
    println(isNotDigit('x')) // true
}
```
`c in 'a'..'z'`는 `'a' <= c && c <= 'z'`로 변환된다.

when에서 in 사용하기
```kotlin
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know..."
}

fun main() {
    println(recognize('8')) // It's a digit!
}
```

비교가능한 클래스(java.lang.Comparable 인터페이스를 구현한 클래스) 그 클래스의 인스턴스 객체를 사용해 범위를 만들 수 있다.
```kotlin
fun main() {
    println("Kotlin" in "Java".."Scala") // true
    println("Kotlin" in setOf("Java", "Scala")) // false
}
```

### 2.5 코틀린의 예외 처리
```kotlin
if(percentage !in 0..100) {
    throw IllegalArgumentException("A percentage value must be between 0 and 100: $percentage")
}
```
자바와 달리 코틀린의 throw는 식이므로 다른 식에 포함될 수 있다.
```kotlin
val percentage =
    if (number in 0..100)
        number
    else
        throw IllegalArgumentException("A percentage value must be between 0 and 100: $number")
```

### 2.5.1 try, catch, finally
```kotlin
fun readNumber(reader: BufferedReader): Int? { // 함수가 던질 수 있는 예외를 명시할 필요가 없다.
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) { // 예외 타입을 :의 오른쪽에 쓴다.
        return null
    } finally {
        reader.close()
    }
}

fun main() {
    val reader = BufferedReader(StringReader("239"))
    println(readNumber(reader)) // 239
}
```
코틀린은 체크 예외와 언체크 예외를 구별하지 않는다.
함수가 던지는 예외를 지정하지 않고 발생한 예외를 잡아내도 되고 잡아내지 않아도 된다.

### 2.5.2 try를 식으로 사용
코틀린의 try 키워드는 if나 when과 마찬가지로 식이다.
```kotlin
fun readNumber3(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null // catch에서 값 반환하기
    }
    println(number)
}

fun main() {
    val reader = BufferedReader(StringReader("not a number"))
    readNumber(reader) // null
}
```

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return // 함수 리턴
    }
    println(number)
}

fun main() {
    val reader = BufferedReader(StringReader("not a number"))
    readNumber2(reader) // 아무것도 출력되지 않는다.
}
```

---
**Reference**<br>
- Kotlin in Action
