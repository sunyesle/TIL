# 4 클래스, 객체, 인터페이스
## 4.1 클래스 계층 정의
### 4.1.1 코틀린 인터페이스
```kotlin
interface Clickable { // 인터페이스 정의
    fun click()
    fun showOff() = println("I'm clickable!") // 디폴트 구현
}

interface Focusable {
    fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}

class Button : Clickable, Focusable { // 클래스 이름 뒤에 콜론을 붙이고 구현할 인터페이스를 적는다.
    override fun click() = println("I was clicked")

    override fun showOff() { // 여러 개의 디폴트 구현이 있을경우 반드시 명시적으로 구현을 제공해야 한다. 누락 시 컴파일 에러가 발생한다.
        super<Clickable>.showOff() // 꺾쇠 괄호로 기반 타입을 지정한다.
        super<Focusable>.showOff()
    }
}
```
- 클래스 이름 뒤에 콜론(`:`)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리한다.
- 메서드를 오버라이드 하는 경우 override 변경자를 반드시 붙여야 한다.
- 인터페이스 디폴트 구현이 필요하다면 메서드 본문을 시그니처 뒤에 추가하면 된다. 자바와 달리 default 키워드를 붙이지 않아도 된다.
- 이름과 시그니처가 같은 멤버 메서드에 둘 이상의 디폴트 구현 메서드가 있다면 반드시 오버라이딩 메서드를 제공해야 한다.
- 꺾쇠 괄호 사이에 기반 타입을 지정해서 어떤 상위타입의 멤버 메서드를 호출할지 지정할 수 있다.

### 4.1.2 open, final, abstract 변경자: 기본적으로 final

#### open, final
코틀린의 클래스와 메서드는 **기본적으로 final**이고 오버라이드를 허용하고 싶은 경우 클래스와 메서드, 프로퍼티 앞에 `open` 변경자를 붙여야 한다.
> 상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라 -Effective Java

```kotlin
open class RichButton : Clickable { // 이 클래스는 열려있다. 다른 클래스가 이 클래스를 상속할 수 있다.
    fun disable() {} // 이 함수는 파이널이다. 하위 클래스가 이 메서드를 오버라이드 할 수 없다.
    open fun animate() {} // 이 함수는 열려있다. 하위 클래스에서 이 메서드를 오버라이드해도 된다.
    override fun click() {} // 이 함수는 열려있는 메서드를 오버라이드 한다. 오버라이드한 메서드는 기본적으로 열려있다.
}

open class RichButton2 : Clickable {
    final override fun click() {} // 오버라이드하는 메서드의 구현을 하위클래스에서 오버라이드 하지 못 하게 하려면 final을 명시한다.
}
```
**열린 클래스와 스마트 캐스트**<br>
스마트 캐스트는 타입 검사 뒤에 변경 될 수 없는 변수에만 적용 가능하다.
이 요구사항은 또한 프로퍼티가 final이어야만 한다는 뜻이기도 하다.
클래스의 기본적인 상속 가능 상태를 final로 함으로써 대부분의 프로퍼티를 스마트 캐스트에 활용할 수 있다.

#### abstract
추상 클래스는 인스턴스화 할 수 없으며, 하위 클래스에서 추상 멤버를 반드시 오버라이드해야 한다.
추상 멤버는 항상 열려있기 때문에 open 변경자를 명시할 필요가 없다.
```kotlin
abstract class Animated { // 추상 클래스 선언. 이 클래스의 인스턴스를 만들 수 없다.
    abstract fun animate() // 이 함수는 추상 함수다. 구현이 있으면 안된다. 하위 클래스에서는 이 함수를 반드시 오버라이드해야 한다.
    open fun stopAnimating() {} // 추상 클래스에 속했더라도 비추상 함수는 기본적으로 final이다. open으로 오버라이드를 허용할 수 있다.
    fun animateTwice() {}
}
```

**클래스 내에서 상속 제어 변경자의 의미**<br>
- `final`: 오버라이드할 수 없다. 클래스 멤버의 기본 변경자다
- `open`: 오버라이드할 수 있다. 반드시 open을 명시해야 오버라이드할 수 있다.
- `abstract`: 반드시 오버라이드해야 한다. 추상클래스의 멤버에만 붙일 수 있다. 추상 멤버에는 구현이 있으면 안 된다.
- `override`: 상위 클래스나 상위 인스턴스의 멤버를 오버라이드하는 중. 오버라이드하는 멤버는 기본적으로 열려있다. 하위클래스의 오버라이드를 금지하려면 final을 명시해야 한다.

### 4.1.3 가시성 변경자: 기본적으로 공개
코틀린의 **기본 가시성은 public**이다.

자바의 기본 가시성인 package-private는 코틀린에 없다.
패키지 전용 가시성에 대한 대안으로 `internal`이라는 가시성 변경자를 도입했다.
internal 가시성은 모듈(한꺼번에 컴파일되는 코틀린 파일들) 내부에서만 볼 수 있다.

코틀린에서는 최상위 선언에 대해 `private` 가시성을 허용한다.
최상위 선언에는 클래스, 함수, 프로퍼티 등이 포함된다.
비공개 가시성인 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다.

자바에서는 같은 패키지 안에서 `protected` 멤버에 접근할 수 있지만
코틀린에서는 선언된 클래스 또는 하위 클래스에서만 접근할 수 있다.

코틀린에서는 외부 클래스가 내부 클래스나 중첩된 클래스의 `private` 멤버에 접근할 수 없다.

### 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
클래스 안에 다른 클래스를 선언하면 도우미 클래스를 캡슐화하거나 코드 정의를 그 코드를 사용하는 곳 가까이 두고 싶을 때 유용하다.
자바와 달리 코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.

View 요소를 하나 만든다고 상상해보자. 그 View의 상태를 직렬화해야 한다.
```kotlin
interface State : Serializable
interface View {
    fun getCurrentState(): State
    fun restoreState(state: State)
}
```

```java
/* 자바 */
public class Button implements View {
    @Override
    public State getCurrentState(){
        return new ButtonState();
    }
    @Override
    public void restoreState(State state) { /*...*/ }
    public class ButtonState implements State { /*...*/ }
}
```
위 코드에서 ButtonState를 직렬화하려고 하면 에러가 발생한다.
자바에서 클래스 안에 정의한 클래스는 자동으로 내부 클래스(inner class)가 된다.
내부 클래스는 바깥 클래스에 대한 참조를 묵시적으로 포함한다.
바깥쪽 클래스인 Button은 직렬화할 수 없기 때문에 에러가 발생한 것이다.
이 문제를 해결하려면 ButtonState를 static 중접 클래스(nested class)로 선언하여 바깥쪽 클래스에 대한 참조를 없애야 한다.

코틀린에서는 아무런 변경자가 붙지 않으면 자바 static 중첩 클래스와 같다.
내부 클래스로 변경하고 싶다면 `inner` 변경자를 붙이면 된다.

**코틀린 중첩 클래스**
```kotlin
class Button : View {
    override fun getCurrentState(): State = ButtonStatus()
    override fun restoreState(state: State) { /*...*/ }
    class ButtonStatus : State { /*...*/ }
}
```

**코틀린 내부 클래스**
```kotlin
// 코틀린 내부 클래스
class Outer{
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```
내부 클래스 Inner 안에서 바깥쪽 Outer의 참조에 접근하려면 `this@Outer`라고 써야 한다.

**중첩 클래스**<br>
- 바깥쪽 클래스에 대한 참조를 저장하지 않는다.
- 자바  : static class A
- 코틀린: class A

**내부 클래스**<br>
- 바깥쪽 클래스에 대한 참조를 저장한다.
- 자바  : class A
- 코틀린: inner class A

### 4.1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한
```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when(e) {
        is Num -> e.value
        is Sum -> eval(e.left) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression") // 꼭 else 분기가 있어야 한다.
    }
```
항상 디폴트 분기를 추가해야 하고,
클래스 계층에 새로운 하위 클래스를 추가하더라도 컴파일러가 when이 모든 경우를 처리하는지 제대로 검사할 수 없다.

`sealed` 클래스로 이런 문제를 해결할 수 있다.

sealed 클래스란 추상 클래스(abstract class)로 상속받는 하위 클래스의 종류를 제한한다.
sealed 클래스의 하위 클래스들은 반드시 같은 패키지 내에 선언되어야 한다.
```kotlin
sealed class Expr{ // 기반 클래스를 sealed로 봉인한다.
    class Num(val value:Int) : Expr() // 하위 클래스
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
    when(e) { // when 식에서 sealed 클래스의 모든 하위 클래스를 처리한다면 디폴트 분기가 필요 없다.
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.left) + eval(e.left)
    }
```

## 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
코틀린은 주(primary) 생성자와 부(secondary) 생성자를 구분한다.
주 생성자는 클래스 본문 밖에서, 부 생성자는 클래스 본문 안에서 정의한다.

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록
```kotlin
class User(val nickname: String)
```
위 코드를 명시적으로 풀어서 보면 다음과 같다.
```kotlin
class User constructor(val _nickname: String) { // 파라미터가 하나만 있는 주 생성자
    val nickname: String
    init { // 초기화 블록
        nickname = _nickname
    }
}
```
- `constructor`: 생성자 정의를 시작할 때 사용한다. 주 생성자 앞에 어노테이션이나 가시성 변경자가 없다면 `constructor`를 생략해도 된다.
- `init`: 초기화 블록을 시작한다. 초기화 블록은 주 생성자와 함께 사용된다. 클래스의 객체가 만들어질 때 실행될 초기화 코드가 들어간다.

모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라미터 없는 생성자를 만들어준다.

**기반 클래스 생성자 호출**<br>
기반 클래스를 초기화하려면 기반 클래스 이름 뒤에 괄호를 치고 생성자 인자를 넘긴다.
```kotlin
open class User(val nickname: String) {}
class TwitterUser(nickname: String) : User(nickname) { // 기반클래스의 생성자 호출
}

open class Button // 컴파일러가 자동으로 인자가 없는 디폴트 생성자를 만들어준다.
class RadioButton: Button() // 하위 클래스에서 반드시 기반클래스의 생성자를 호출해야 한다.
```
> 클래스 정의에 있는 상위 클래스 및 인터페이스 목록에서 이름 뒤에 괄호 유무로 클래스와 인터페이스를 구분할 수 있다.

**가시성 변경**<br>
```kotlin
class Secretive private constructor() {} // 주 생성자의 가시성 변경
```
> **비공개 생성자에 대한 대안**<br>
> 정적 유틸리티 함수 대신 최상위 함수, 싱글턴을 사용하고 싶으면 객체 선언

### 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화
```kotlin
open class View {
    constructor(ctx: Context) { // 부 생성자
        // ...
    }
    constructor(ctx: Context, attr: AttributeSet){
        // ...
    }
}

class MyView : View {
    constructor(ctx: Context) : this(ctx, MY_STYLE) { // this() 키워드로 이 클래스의 다른 생성자에게 위임한다.
        // ...
    }
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) { // super() 키워드로 상위 클래스의 생성자를 호출한다.
        // ...
    }
}
```
클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야한다.

### 4.2.3 인터페이스에 선언된 프로퍼티 구현
```kotlin
interface User {
    val nickname: String
}

// 주 생성자에 있는 프로퍼티
class PrivateUser(override val nickname: String) : User 

// 커스텀 접근자에서 매번 값을 계산하는 프로퍼티
// 호출될 때마다 결과를 계산해서 돌려준다.
class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 커스텀 게터
}

// 값을 저장하는 프로퍼티
// 객체 초기화시 계산한 데이터를 뒷받침하는 필드에 저장했다가 불러온다.
class FacebookUser(val accountId: Int) : User {
    override val nickname: String = getFacebookName(accountId) // 커스텀 초기화 식

    private fun getFacebookName(accountId: Int): String {
        return accountId.toString()
    }
}
```

인터페이스에 게터/세터가 있는 프로퍼티도 선언 가능하다.
인터페이스에는 상태를 저장할 수 없기때문에 뒷받침하는 필드를 참조할 수는 없다.
오버라이드 하지 않고 상속할 수 있다.
```kotlin
interface User {
    val email: String
    val nickname: String
        get() = email.substringBefore('@')
}
```

### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근
접근자의 본문에서 `field` 식별자를 통해 뒷받침하는 필드에 접근할 수 있다.
```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println(
                """
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent() // 뒷받침하는 필드 값 읽기
            )
            field = value // 뒷받침하는 필드 값 변경하기
        }
}
```
```kotlin
val user = User("Alice")
user.address = "Elsenheimerstrasse 47, 807687 Muenchen" // 내부적으로 세터를 호출한다.
/* 출력
Address was changed for Alice:
"unspecified" -> "Elsenheimerstrasse 47, 807687 Muenchen".
*/
```

### 2.4.5 접근자의 가시성 변경
get/set 앞에 가시성 변경자를 추가해서 접근자의 가시성을 변경할 수 있다.
```kotlin
class LengthCounter {
    var counter: Int = 0
        private set

    fun addWord(word: String) {
        counter += word.length
    }
}
```
```kotlin
val lengthCounter = LengthCounter()
lengthCounter.addWord("Hi!")
println(lengthCounter.counter) // 3
```

## 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임
### 4.3.1 모든 클래스가 정의해야 하는 메서드
자바와 마찬가지로 모든 코틀린 클래스는 toString, equals, hashCode 등을 오버라이드해야 한다.
```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString(): String = "Client(name='$name', postalCode=$postalCode)"

    override fun equals(other: Any?): Boolean { // Any는 java.lang.Object에 대응하는 클래스로 코틀린의 모든 클래스의 최상위 클래스다.
        if (other == null || other !is Client)
            return false
        return name == other.name && postalCode == this.postalCode
    }

    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```
#### 문자열 표현 toString()

#### 객체의 동등성 equals()
코틀린에서는 동등성 연산에 `==`을 사용한다. 내부적으로는 `equals`를 호출해서 객체를 비교한다.

#### 해시 컨테이너 hashCode()
`equals`가 true를 반환하는 두 객체는 반드시 같은 `hashCode`를 반환해야 한다.
해시 컨테이는 원소를 비교할 때 비용을 줄이기 위해 먼저 객체의 해시 코드를 비교하고(`hashCode`) 같은 경우에만 실제 값을 비교한다(`equals`).

### 4.3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메서드 자동 생성
어떤 클래스가 데이터를 저장하는 역할만을 수행한다면 toString, equals, hashCode를 반드시 오버라이드해야 한다.
`data` 변경자를 클래스 앞에 붙이면 이런 메서드를 컴파일러가 자동으로 만들어준다.
이때 주 생성자 밖에 정의된 프로퍼티는 equals나 hashCode를 계산할 때 고려의 대상이 아니다.
```kotlin
data class Client(val name: String, val postalCode: Int)
```

#### 데이터 클래스와 불변성: copy() 메서드
데이터 클래스의 모든 프로퍼티를 읽기 전용(val)로 만들어서 불변(immutable) 클래스로 만들라고 권장한다.
HashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성이 필수적이다.

코틀린 컴파일러는 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 copy메서드를 제공한다.
데이터 클래스 인스턴스를 불변 객체로 더 쉽게 활용할 수 있다.
```kotlin
val lee = Client2("이계영", 4122)
println(lee.copy(postalCode = 4000)) // Client2(name=이계영, postalCode=4000)
```

### 4.3.3 클래스 위임: by 키워드 사용
상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때가 있다.
이럴 때 사용하는 일반적인 방법이 데코레이터 패턴이다.
```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()

    override val size: Int
        get() = innerList.size

    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun iterator(): Iterator<T> = innerList.iterator()
    override fun containsAll(elements: Collection<T>): Boolean = innerList.containsAll(elements)
    override fun contains(element: T): Boolean = innerList.contains(element)
}
```
코틀린에서는 인터페이스를 구현할 때 `by` 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

by 키워드를 사용해 앞의 예제를 재작성한 코드이다.
```kotlin
class DelegatingCollection2<T>(
    innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {

}
```

**클래스 위임 사용하기**
```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet { // MutableCollection의 구현을 innerSet에게 위임한다.
    var objectAdded = 0

    override fun add(element: T): Boolean { // 위임하지 않고 새로운 구현을 제공한다.
        objectAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        objectAdded += elements.size
        return innerSet.addAll(elements)
    }
}
```

## 4.4 Object 키워드: 클래스 선언과 인스턴스 생성
코틀린에서는 `object`키워드를 다양한 상황에서 사용하지만 모든 경우 클래스를 정의하면서 동시에 인스턴스를 생성한다는 공통점이 있다.
- **객체 선언**(object declaration)은 싱글턴을 정의하는 방법 중 하나다.
- **동반 객체**(companion object)는 인스턴스 메서드는 아니지만 어떤 클래스와 관련있는 메서드와 팩토리 메서드를 담을 때 쓰인다.
- **객체 식**은 자바의 무명 내부 클래스대신 쓰인다.

### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기
코틀린은 **객체 선언** 기능을 통해 싱글턴을 언어에서 기본 지원한다.
객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.

객체 선언은 `object` 키워드로 시작한다.
객체 선언 안에도 프로퍼티, 메서드, 초기화 블록 등이 들어갈 수 있다. 하지만 생성자는 객체 선언에 쓸 수 없다.
싱글턴 객체는 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다.

구체적인 예제로 두 파일 경로를 대소문자 관계없이 비교해주는 Comparator을 구현해보자

**객체 선언을 사용해 Comparator 구현하기**
```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path, ignoreCase = true)
    }
}
```
```kotlin
println(CaseInsensitiveFileComparator.compare(File("/User"), File("/user"))) // 0

// 일반 객체(인스턴스)를 사용할 수 있는 곳에서는 항상 싱글턴 객체를 사용할 수 있다.
var files = listOf(File("/Z"), File("/a"))
println(files.sortedWith(CaseInsensitiveFileComparator)) // [\a, \Z]
```

클래스 안에서 객체를 선언할 수도 있다.

**중첩 객체를 사용해 Comparator 구현하기**
```kotlin
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int {
            return p1.name.compareTo(p2.name)
        }
    }
}
```
```kotlin
val persons = listOf(Person("Bob"), Person("Alice"))
println(persons.sortedWith(Person.NameComparator)) // [Person(name=Alice), Person(name=Bob)]
```

#### 코틀린 객체를 자바에서 사용하기
코틀린 객체 선언은 유일한 인스턴스에 대한 정적인 필드가 있는 자바 클래스로 선언된다.
인스턴스 필드의 이름은 항상 INSTANCE다.
```java
/* 자바 */
CaseInsensitiveFileComparator.INSTANCE.compare(file1, file2);
```

### 4.4.2 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소
코틀린 클래스 안에는 정적인 멤버가 없다. 코틀린 언어는 자바 static 키워드를 지원하지 않는다.
그 대신 패키지 수준의 최상위 함수와 객체 선언을 활용한다.
- **최상위 함수**: 정적 메서드를 거의 대신할 수 있다. 대부분의 경우 최상위 함수 활용을 더 권장한다.
- **객체 선언**: 최상위 함수가 대신할 수 없는 역할이나 정적 필드를 대신할 수 있다.

하치만 최상위 함수는 private으로 표시된 클래스는 비공개 멤버에 접근할 수 없다.
클래스 내부 정보에 접근해야 할 경우 **클래스에 중첩된 객체 선언의 멤버 함수**로 정의해야 한다.(ex.팩토리 메서드)

클래스 안에 정의된 객체 중 하나에 `companion`이라는 표시를 붙이면 그 클래스의 동반 객체로 만들 수 있다.
동반 객체의 프로퍼티나 메서드에 접근하려면 그 동반가 정의된 클래스의 이름을 사용한다.
그 결과 동반객체의 멤버를 사용하는 구문은 자바의 정적 메서드, 정적 필드 사용 구문과 같아진다.

```kotlin
class A {
    companion object { // 객체의 이름을 따로 지정할 필요 없다.
        fun bar() {
            println("Companion object called")
        }
    }
}
```
```kotlin
A.bar() // Companion object called
```

#### 동반 객체로 팩토리 패턴을 구현하기
부 생성자가 2개 있는 클래스를 동반 객체 안에서 팩토리 클래스를 정의하는 방식으로 변경해보자.

**부 생성자가 여럿 있는 클래스**
```kotlin
class UserOld {
    val nickname: String

    constructor(email: String) {
        nickname = email.substringBefore('@')
    }

    constructor(facebookAccountId: Int) {
        nickname = getFacebookName(facebookAccountId)
    }

    private fun getFacebookName(facebookAccountId: Int): String {
        return ""
    }
}
```

**부 생성자를 팩토리 메서드로 대신하기**
```kotlin
class User private constructor(val nickname: String) { // 주 생성자를 비공개로 만든다.
    companion object { // 동반 객체를 선언한다.
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```
```kotlin
val subscribingUser = User.newSubscribingUser("bob@gmail.com")
val facebookAccountId = User.newFacebookUser(4)
println(subscribingUser.nickname) // bob
```
클래스를 확장해야만 하는 경우에는 동반 객체 멤버를 하위 클래스에서 오버라이드 할 수 없으므로 여러 생성자를 사용하는 편이 더 나은 해법이다.

### 4.4.3 동반 객체를 일반 객체처럼 사용
동반 객체는 클래스 안에 정의된 일반 객체다.

#### 동반 객체에 이름 붙이기
이름을 지정하지 않으면 동반 객체의 이름은 자동으로 Companion이 된다.
```kotlin
class Person(val name: String) {
    companion object Loader { // 동반 객체에 이름을 붙인다.
        fun fromJSON(jsonText: String): Person = ...
    }
}
```
```kotlin
Person.Loader.fromJSON("{name: 'Dmitry'}")
Person.fromJSON("{name: 'Dmitry'}")
```

#### 동반 객체에서 인터페이스 구현
다른 객체 선언과 마찬가지로 동반 객체도 인터페이스를 구현할 수 있다.
인터페이스를 구현하는 동반 객체를 참조할 때 객체를 둘러싼 클래스의 이름을 바로 사용할 수 있다.
```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}
class Person(val name: String) {
    companion object: JSONFactory<Person>{ // 동반 객체가 인터페이스를 구현한다.
        override fun fromJSON(jsonText: String): Person = ...
    }
}
```
```kotlin
fun loadFromJSON<T>(factory: JSONFactory<T>): T {
    ...
}
loadFromJSON(Person) // 동반 객체의 인스턴스를 함수에 넘긴다.
```
동반 객체가 구현한 JSONFactory의 인스턴스를 넘길 때 Person 클래스의 이름을 사용했다.

#### 동반 객체 확장
클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다.

Person 클래스는 비즈니스 모듈의 일부이다.
관심사를 명확히 분리하기위해 역직렬화 함수를 비즈니스 모듈이 아니라 클라이언트/서버 통신을 모듈안에 선언하고 싶다.
이럴 때 확장 함수를 사용하여 구조를 잡을 수 있다.
```kotlin
// 비즈니스 로직 모듈
class Person(val firstName: String, val lastName: String) {
    companion object { // 비어있는 동반 객체를 선언한다.
    }
}

// 클라이언트/서버 통신 모듈
fun Person.Companion.fromJSON(json: String): Person {
    ...
}

// 마치 동반 객체 안에서 fromJSON 함수를 정의한 것처럼 호출할 수 있다.
val p = Person.fromJSON(json)
```

### 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성
무명 객체(anonymous object)를 지정할 때도 `object` 키워드를 쓴다.
무명 객체는 자바의 무명 내부 클래스를 대신한다.

객체 선언과 달리 무명 객체는 싱글턴이 아니다. 객체식이 쓰일 때마다 새로운 인스턴스가 생성된다.

자바의 무명 내부 클래스와 달리 코틀린의 무명 클래스는 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.
또한 객체 식 내부에서 final이 아닌 외부 변수에도 접근할 수 있다.

객체 식은 무명 객체 안에서 여러 메서드를 오버라이드해야 하는 경우 유용하다.
메서드가 하나뿐인 인터페이스라면 무명 객체대신 람다를 활용하는 편이 낫다.
```kotlin
interface Clickable { // 인터페이스 정의
    fun click()
}
```
```kotlin
var clickCount = 0
val myButton = object : Clickable {
    override fun click() {
        clickCount++ // 식이 포함된 함수의 변수에 접근할 수 있다.
        println("myButton click! clickCount: $clickCount")
    }
}
myButton.click() // myButton click
```

---
**Reference**<br>
- Kotlin in Action
