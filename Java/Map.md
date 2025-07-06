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

## compute(), computeIfPresent(), merge()
value 값을 업데이트하는 메서드이다.

### compute()
```java
default V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue = get(key);

    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue == null) {
        // delete mapping
        if (oldValue != null || containsKey(key)) {
            // something to remove
            remove(key);
            return null;
        } else {
            // nothing to do. Leave things as they were.
            return null;
        }
    } else {
        // add or replace old mapping
        put(key, newValue);
        return newValue;
    }
}
```
`remappingFunction.apply(key, oldValue)`

key 값의 존재 여부와 상관없이 remappingFunction을 호출한다.

- remappingFunction의 반환 값이 null인 경우
  - key 값이 존재하는 경우
    - 값을 삭제하고 null을 반환한다.
  - key 값이 존재하지 않는 경우
    - null을 반환한다.
- remappingFunction의 반환 값 null이 아닌 경우
  - 값을 업데이트하고, 새로운 값을 반환한다.

```java
Map<String, Integer> map = new HashMap<>();

// Key 값이 존재하는 경우
map.put("A", 1);
map.compute("A", (key, value) -> value + 1);
System.out.println(map.get("A")); // 2
System.out.println(map.containsKey("A")); // true

// Key 값이 존재하지 않는 경우
map.compute("B", (key, value) -> value == null ? 0 : value + 1); // NullPointerException 방지
System.out.println(map.get("B")); // 0
System.out.println(map.containsKey("B")); // true

// 람다식의 결과가 null 인 경우
map.put("C", 1);
map.compute("C", (key, value) -> null);
System.out.println(map.get("C")); // null
System.out.println(map.containsKey("C")); // false

System.out.println(map); // {A=2, B=0}
```

### computeIfPresent()
```java
default V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue != null) {
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}
```
`remappingFunction.apply(key, oldValue)`

- key 값이 존재하고, 값이 null이 아닌 경우
  - remappingFunction의 반환 값이 null인 경우
    - 값을 삭제하고 null을 반환한다.
  - remappingFunction의 반환 값이 null이 아닌 경우
    - 값을 업데이트한다.
- key 값이 존재하지 않거나, 값이 null인 경우
  - null을 반환한다.

```java
Map<String, Integer> map = new HashMap<>();

// Key 값이 존재하는 경우
map.put("A", 1);
map.computeIfPresent("A", (key, value) -> value + 1);
System.out.println(map.get("A")); // 2
System.out.println(map.containsKey("A")); // true

// Key 값이 존재하지 않는 경우
map.computeIfPresent("B", (key, value) -> value + 1);
System.out.println(map.get("B")); // null
System.out.println(map.containsKey("B")); // false

// 람다식의 결과가 null 인 경우
map.put("C", 1);
map.computeIfPresent("C", (key, value) -> null);
System.out.println(map.get("C")); // null
System.out.println(map.containsKey("C")); // false

System.out.println(map); // {A=2}
```

### merge()
```java
default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if (newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```
`remappingFunction.apply(oldValue, value)`

- key 값이 존재하지 않으면, value를 그대로 put한다.
- key 값이 존재하면 remappingFunction을 호출한다.
  - remappingFunction의 반환 값이 null인 경우
    - 값을 삭제한다.
  - remappingFunction의 결과가 null이 아닌 경우
    - 값을 업데이트한다.

```java
Map<String, Integer> map = new HashMap<>();

// Key 값이 존재하는 경우
map.put("A", 1);
map.merge("A", 0, (oldValue, value) -> oldValue + 1);
System.out.println(map.get("A")); // 2
System.out.println(map.containsKey("A")); // true

// Key 값이 존재하지 않는 경우
map.merge("B", 0, (oldValue, value) -> oldValue + 1);
System.out.println(map.get("B")); // 0
System.out.println(map.containsKey("B")); // true

// 람다식의 결과가 null 인 경우
map.put("C", 1);
map.merge("C", 0, (oldValue, value) -> null);
System.out.println(map.get("C")); // null
System.out.println(map.containsKey("C")); // false

System.out.println(map); // {A=2, B=0}
```

### 차이점
- `compute()`
  - `remappingFunction.apply(key, oldValue)`
  - key 값의 존재 여부와 상관없이 remappingFunction을 호출한다.
  - "있으면 갱신, 없으면 새로 생성" 같은 유연한 처리가 필요할 때 사용한다.
- `computeIfPresent()`
  - `remappingFunction.apply(key, oldValue)`
  - key 값이 존재하고, value가 null이 아닐 때만 remappingFunction을 호출한다.
  - 기존 데이터만 수정할 때 사용한다.
- `merge()`
  - `remappingFunction.apply(oldValue, value)`
  - key 값이 존재하고, value가 null이 아닐 때만 remappingFunction을 호출한다.
  - 기존값과 새로운 값을 병합할 때 사용한다.

## getOrDefault()
찾는 key가 존재한다면 찾는 key의 값을 반환하고 없다면 기본값을 반환한다.
```java
Map<String, Integer> map = new HashMap<>();

// Key 값이 존재하는 경우
map.put("A", 1);
Integer value1 = map.getOrDefault("A", 0);
System.out.println(value1); // 1

// Key 값이 존재하지 않는 경우
Integer value2 = map.getOrDefault("B", 0);
System.out.println(value2); // 0
```

---
**Reference**<br>
- https://bangpurin.tistory.com/213
- https://blog.advenoh.pe.kr/자바8-hashmap-보다-간결하고-효과적으로-작성하기/
