# ConcurrentHashMap
`ConcurrentHashMap`은 자바의 동시성 컬렉션 클래스 중 하나로, 멀티스레드 환경에서 안전하게 사용할 수 있는 해시 맵이다.

## Slipped Conditions 피하기
`ConcurrentHashMap`의 메서드는 스레드에 안전하지만, 잘못된 방법으로 사용할 경우 동시성 문제가 발생할 수 있다.

```java
ConcurrentMap map = new ConcurrentHashMap();

if (!map.containsKey("key1")) {
    map.put("key1", "value1");
}
```
`containsKey()`와 `put()` 메서드가 스레드로부터 안전하더라도, 위의 if문 구조는 스레드로부터 안전하지 않다.

예를 들어 두 개의 스레드가 동시에 `map.containsKey("key1")`를 호출할 경우, 두 스레드 모두 false를 반환받을 수 있다.<br>
그 결과 두 스레드 모두 if문 블록으로 진입해 `map`에 값을 넣게 되고, 두 번째로 값을 넣은 스레드는 첫 번째 스레드가 넣은 값을 덮어써 버리게 된다.

`putIfAbsent()` 또는 `computeIfAbsent()`같은 원자적(atomic) 메서드를 사용하여 이 문제를 해결할 수 있다.
```java
ConcurrentMap map = new ConcurrentHashMap();

map.putIfAbsent("key1", "value1");
```
```java
ConcurrentMap map = new ConcurrentHashMap();

map.computeIfAbsent("key2", key ->
    "Value for key " + key + " : " + Thread.currentThread().getName()
);
```

---
**Reference**
- https://jenkov.com/tutorials/java-util-concurrent/concurrentmap.html
- https://devel-repository.tistory.com/92
- https://curiousjinan.tistory.com/entry/java-concurrent-hash-map-cas
