# [Kotlin] 범위 지정 함수 let, with, run, apply, also

## 범위 지정 함수(Scope Funciton)
범위 지정 함수란 특정 객체(수신 객체)에 대한 작업을 블록 안에서 실행할 수 있도록 하는 함수이다.

Kotlin에서는 `let`, `run`, `with`, `apply`, `also` 등의 범위 지정 함수를 제공한다.

이 함수들은 객체의 컨텍스트 내에서 코드 블록을 실행한다는 공통점이 있지만,
객체가 블록 내에서 사용되는 방식과 블록의 반환값에 차이가 있다.

## 차이점
각각의 범위 지정 함수들은 다음과 같은 차이가 있다.

**범위 지정 함수 호출 시 수신 객체가 어떻게 전달되는가**
- 확장 함수의 수신 객체로 제공: `let`, `run`, `apply`, `also`
- 수신 객체 없이 호출: `run { ... }`
- 매개변수로 제공: `with`

**수신 객체가 수신 객체 지정 람다에 어떠한 형식으로 전달되는가**
- 람다의 매개변수(`it`)로 제공: `let`, `also`
- 람다의 수신 객체(`this`)로 제공: `run`, `with`, `apply` 

**반환값**
- 람다의 결과를 반환: `let`, `run`, `with`
- 수신 객체 자신을 반환: `apply`, `also`

| 함수      | 수신 객체 전달 방식      | 람다 내부 참조 | 반환값  |
|---------|------------------|----------|-------|
| `let`   | 확장 함수의 수신 객체     | `it`     | 람다 결과 |
| `run`   | 확장 함수의 수신 객체     | `this`   | 람다 결과 |
| `run`   | 수신 객체 없음 (일반 함수) | 없음       | 람다 결과 |
| `with`  | 매개변수             | `this`   | 람다 결과 |
| `apply` | 확장 함수의 수신 객체     | `this`   | 수신 객체 |
| `also`  | 확장 함수의 수신 객체     | `it`     | 수신 객체 |

## 종류
### let
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R
```
null이 될 수 없는 객체에서 람다를 수행하고자 하는 경우<br>
로컬 범위 변수로 표현식을 소개하는 경우

### with
```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R
```
객체에 대한 함수 호출 그룹화

### run
```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R
```
객체 설정과 계산 결과 반환 시

```kotlin
public inline fun <R> run(block: () -> R): R
```
표현식이 필요한 명령문 실행

### apply
```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```
객체 설정

### also
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T
```
객체를 사용하는 추가 작업이 필요한 경우# [Kotlin] 범위 지정 함수 let, with, run, apply, also

## 범위 지정 함수(Scope Funciton)
범위 지정 함수란 특정 객체(수신 객체)에 대한 작업을 블록 안에서 실행할 수 있도록 하는 함수이다.

Kotlin에서는 `let`, `run`, `with`, `apply`, `also` 등의 범위 지정 함수를 제공한다.

이 함수들은 객체의 컨텍스트 내에서 코드 블록을 실행한다는 공통점이 있지만,
객체가 블록 내에서 사용되는 방식과 블록의 반환값에 차이가 있다.

## 차이점
각각의 범위 지정 함수들은 다음과 같은 차이가 있다.

**범위 지정 함수 호출 시 수신 객체가 어떻게 전달되는가**
- 확장 함수의 수신 객체로 제공: `let`, `run`, `apply`, `also`
- 수신 객체 없이 호출: `run { ... }`
- 매개변수로 제공: `with`

**수신 객체가 수신 객체 지정 람다에 어떠한 형식으로 전달되는가**
- 람다의 매개변수(`it`)로 제공: `let`, `also`
- 람다의 수신 객체(`this`)로 제공: `run`, `with`, `apply` 

**반환값**
- 람다의 결과를 반환: `let`, `run`, `with`
- 수신 객체 자신을 반환: `apply`, `also`

| 함수      | 수신 객체 전달 방식      | 람다 내부 참조 | 반환값  |
|---------|------------------|----------|-------|
| `let`   | 확장 함수의 수신 객체     | `it`     | 람다 결과 |
| `run`   | 확장 함수의 수신 객체     | `this`   | 람다 결과 |
| `run`   | 수신 객체 없음 (일반 함수) | 없음       | 람다 결과 |
| `with`  | 매개변수             | `this`   | 람다 결과 |
| `apply` | 확장 함수의 수신 객체     | `this`   | 수신 객체 |
| `also`  | 확장 함수의 수신 객체     | `it`     | 수신 객체 |

## 종류
### let
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R
```
null이 될 수 없는 객체에서 람다를 수행하고자 하는 경우<br>
로컬 범위에서 변수로 표현식을 사용하는 경우

### with
```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R
```
객체에 대해 여러 함수를 호출할 때 코드 블록으로 그룹화

### run
```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R
```
객체 설정 및 계산 결과 반환 시

```kotlin
public inline fun <R> run(block: () -> R): R
```
반환값이 필요한 코드 블록 실행 시

### apply
```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```
객체 설정

### also
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T
```
객체를 대상으로 추가적인 작업이 필요한 경우

---
**Reference**<br>
- https://kotlinlang.org/docs/scope-functions.html
- https://velog.io/@sjlim/Kotlin-%EB%B2%94%EC%9C%84-%EC%A7%80%EC%A0%95-%ED%95%A8%EC%88%98-Scope-Funciton-let-with-run-apply-also#with
- https://devocean.sk.com/blog/techBoardDetail.do?ID=165527&boardType=techBlog
- https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29
