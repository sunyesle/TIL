# 5 람다로 프로그래밍
**람다 식**(lambda expression)은 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다.

## 5.1 람다 식과 멤버 참조
### 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기
무명 내부 클래스 대신 람다 식을 사용하여 코드를 간결하게 만들 수 있다.

### 5.1.2 람다와 컬렉션
```kotlin
data class Person(val name: String, val age: Int)
```

**컬렉션을 직접 검색하기**
```kotlin
fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}
```
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
findTheOldest(people) // Person(name=Bob, age=31)
```

**람다를 사용해 컬렉션 검색하기**
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.maxBy { it.age }) // Person(name=Bob, age=31)
```

**멤버 참조를 사용해 컬렉션 검색하기**
```kotlin
people.maxBy(Person::age)
```

### 5.1.3 람다 식의 문법
```kotlin
    val sum = { x: Int, y: Int -> x + y }
    println(sum(1, 2)) // 3
```
- `x: Int, y: Int` - 파라미터
- `x + y` - 본문
- 람다식은 항상 중괄호로 둘러싸여 있다.

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))

// 코드를 줄여 쓸 수 있게 제공했던 기능을 제거하고 정식으로 람다를 작성하면 다음과 같다.
people.maxBy({ p: Person -> p.age })

// 함수 호출 시 맨 뒤에 있는 인자가 람다 식이라면 그 람다를 괄호 밖으로 빼낼 수 있다.
people.maxBy() { p: Person -> p.age }

// 람다가 함수의 유일한 인자이고 괄호 뒤에 람다를 썼다면 빈 괄호를 없애도 된다.
people.maxBy { p: Person -> p.age }

// 컴파일러가 문맥으로부터 유추할 수 있는 인자 타입은 생략 가능하다.
people.maxBy { p -> p.age }

// 람다의 인자가 단 하나뿐이고 그 타입을 컴파일러가 추론할 수 있는 경우 it을 바로 쓸 수 있다.
people.maxBy { it.age } // it은 디폴트 파라미터 이름이다.
```
람다가 중첩되어 있거나 문맥에서 람다 파라미터의 의미나 타입을 쉽게 알 수 없는 경우,
`it`을 사용하는 것보다 파라미터를 명시적으로 선언하는게 좋다.

컴파일러가 타입을 추론하지 못하거나 타입 정보가 코드를 읽을 때 도움이 된다면
파라미터 타입을 표기한다.
```kotlin
// 람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 존재하지 않기 떄문에 타입을 명시해야 한다.
val getAge = { p: Person -> p.age }
people.maxBy(getAge)
```

본문이 여러 줄로 이뤄진 경우 본문의 맨 마지막에 있는 식이 람다의 결과 값이 된다.
```kotlin
val sum = { x: Int, y: Int ->
    println("Computing the sum of $x and $y...")
    x + y
}
println(sum(1, 2))
/* 출력
Computing the sum of 1 and 2...
3
*/
```

람다식을 만들자마자 호출하고 싶을 때
```kotlin
{ println(42) }() // 이런 식으로 람다 식을 직접 호출해도 되지만
run { println(42) } // run 함수를 사용하는게 읽기 쉽다.
```

### 5.1.4 현재 영역에 있는 변수에 접근
람다 안에서 사용하는 외부 변수를 **람다가 포획한 변수**라고 부른다.

**함수 파라미터를 람다 안에서 사용하기**
```kotlin
fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it") // 함수의 파라미터에 접근
    }
}
```
```kotlin
val errors = listOf("403 Forbidden", "404 Not Found")
printMessageWithPrefix(errors, "Error:")
```

**람다 안에서 바깥 함수의 로컬 변수 변경하기**
```kotlin
fun printProblemCounts(response: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    response.forEach {
        if (it.startsWith("4")) {
            clientErrors++ // 자바와 달리 final이 아닌 변수도 접근/변경이 가능하다.
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}
```
```kotlin
val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
printProblemCounts(responses) // 1 client errors, 1 server errors
```

> **변경 가능한 변수 포획하기: 자세한 구현 방법**<br>
> 람다가 final 변수(val)를 포획하면 그 변수의 값이 복사된다.<br>
> 람다가 변경 가능한 변수(var)를 포획하면 변수를 Ref 클래스 인스턴스에 넣는다.<br>
> 그 Ref 인스턴스에 대한 참조를 final로 만들어 람다로 포획하고, 람다 안에서는 Ref 인스턴스의 필드를 변경할 수 있다.<br>

람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있다.

### 5.1.5 멤버 참조
이중 콜론(`::`)을 사용하여 함수를 값으로 바꿀 수 있다.
`::`를 사용하는 식을 **멤버 참조**(member reference)라고 부른다.
멤버 참조는 프로퍼티나 메서드를 단 하나만 호출하는 함수 값을 만들어준다.
```kotlin
val getAge = Person::age
```

확장 함수도 멤버 함수와 동일한 방식으로 참조 할 수 있다.
```kotlin
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```

최상위에 선언된 함수나 프로퍼티를 참조 할 수도 있다.
클래스 이름을 생략하고 `::`로 참조를 바로 시작한다.
```kotlin
fun salute() = println("Salute!")

fun main() {
    run(::salute) // Salute!
}
```

**생성자 참조**(constructor reference)를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다.
`::`뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.
```kotlin
val createPerson = ::Person // Person의 인스턴스를 만드는 동작을 값으로 저장한다.
val p = createPerson("Alice", 29)
println(p) // Person(name=Alice, age=29)
```

**바운드 멤버 참조**(bound memeber reference)를 사용하면 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장한 다음 나중에 그 인스턴스에 대해 멤버를 호출해 준다.
```kotlin
val p = Person("Dmitry", 34)
val dmitrysAgeFunction = p::age // 바운드 멤버 참조
println(dmitrysAgeFunction())
```

---
**Reference**<br>
- Kotlin in Action
