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

---
**Reference**<br>
- Kotlin in Action
