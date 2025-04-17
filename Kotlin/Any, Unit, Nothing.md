# [Kotlin] Any, Unit, Nothing

## Any
- Java의 Object에 대응되는 클래스이다.
- 모든 코틀린 클래스의 조상이다.
- toString(), equals(), hashCode() 함수를 가지고 있다.
- Any는 확장 함수를 통해 특별한 메서드들을 제공한다.
  - Pair를 만들기 위한 to() 함수
  - let(), run(), apply(), also() 등

## Unit
- Java의 void와 대응되는 개념이다.
- 자바에서 함수의 리턴 타입이 없을 경우 void를 사용하듯이 코틀린에서는 Unit을 지정하게 되면 리턴 값이 없는 함수가 된다.
```java
fun returnsUnit(): Unit {
}

fun returnsUnitExplicitly1(): Unit {
    return
}

fun returnsUnitExplicitly2(): Unit {
    return Unit
}
```
- Unit은 싱글톤 인스턴스이다. 코틀린에서 Unit이라는 키워드는 타입이면서도 동시에 객체이기도 하다.
```java
public object Unit {
    override fun toString() = "kotlin.Unit"
}
```
- 코틀린은 함수에 리턴이 없으면 Unit 타입으로 추론한다.
- void가 아니라 Unit 객체를 리턴하기 때문에 모든 함수가 표현식이 될 수 있다.


## Nothing
- Unit 타입처럼 값을 반환하지 않는 함수를 나타낼 때 사용한다.
- Unit과의 차이점은 Unit은 객체를 return 하지만, Nothing은 return 자체를 하지 않는다.
```java
fun infinityLoop(): Nothing{
	while(true){
    	println("HELLO WORLD!")
    }
}

fun throwException(): Nothing{
	throw Exception()
}
```
- Noting은 어떠한 값도 포함하지 않는 타입이다. 생성자도 접근할 수 없고 어떠한 값도 얻을 수 없다.
```java
public class Nothing private constructor()
```
- 모든 클래스의 자식 클래스이다. Nothing은 모든 클래스로 대체될 수 있다.

---
**Reference**<br>
- https://readystory.tistory.com/143
- https://sangyoon98.tistory.com/30
- https://velog.io/@min0505/Kotlin-Any-Unit-Nothing-Java-void
