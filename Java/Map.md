# Map

## putIfAbsent(), computeIfAbsent()
key의 존재 여부에 따라서 새로운 key와 value 값을 추가하는 메서드이다.

### putIfAbsent()
```java
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}
```
key 값이 존재하는 경우, map의 value의 값을 반환한다.<br>
key 값이 존재하지 않거나 null일 경우, key와 value를 map에 저장하고 null을 반환한다.

```java
Map<String, Integer> map = new HashMap<>();

// Key 값이 존재하는 경우
map.put("A", 1);
Integer value1 = map.putIfAbsent("A", 2);
System.out.println(value1); // 1

// Key 값이 존재하지 않는 경우
Integer value2 = map.putIfAbsent("B", 2);
System.out.println(value2); // null

System.out.println(map); // {A=1, B=2}
```

### computeIfAbsent()
```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) {
            put(key, newValue);
            return newValue;
        }
    }

    return v;
}
```
key 값이 존재하는 경우 map의 value의 값을 반환한다.<br>
key 값이 존재하지 않거나 null인 경우, mappingFunction의 반환 값을 map에 저장하고 반환한다.

```java
Map<String, Integer> map = new HashMap<>();

// Key 값이 존재하는 경우
map.put("A", 1);
Integer value1 = map.computeIfAbsent("A", (key) -> 2);
System.out.println(value1); // 1

// Key 값이 존재하지 않는 경우
Integer value2 = map.computeIfAbsent("B", (key) -> 2);
System.out.println(value2); // 2

System.out.println(map); // {A=1, B=2}
```

### 차이점
- `putIfAbsent()`
  - 이미 준비된 값을 Map에 넣는다.
  - 단순히 키가 없을 때 특정 값을 추가하고 싶을 때 사용한다.
- `computeIfAbsent()`
  - 값이 필요할 때만 mappingFunction을 호출해서 값을 계산하고 넣는다.
  - 값 생성 비용이 크거나, 키에 따라 동적으로 값을 계산해야 할 때 사용한다.

---
**Reference**<br>
- https://bangpurin.tistory.com/213
- https://blog.advenoh.pe.kr/자바8-hashmap-보다-간결하고-효과적으로-작성하기/
