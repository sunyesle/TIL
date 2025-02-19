# List

## List 생성
### new 연산자
```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");
```

```java
List<String> list = new ArrayList<>(3); // capacity 지정
list.add("A");
list.add("B");
list.add("C");
```

### Arrays.asList
고정된 크기의 리스트를 반환한다.<br>
리스트에 추가/삭제를 하는 경우 `UnsupportedOperationException` 예외를 발생시킨다.
```java
List<String> list = Arrays.asList("A", "B", "C");
```

### List.of
불변 리스트를 반환한다.
```java
List<String> list = List.of("A", "B", "C");
```

## List 복사
### Collections.unmodifiableList
원본 리스트에 대한 **불변 뷰**를 반환한다. 원본 리스트가 변경되면 영향을 받는다.
```java
List<String> list = Arrays.asList("A", "B", "C");
List<String> newList = Collections.unmodifiableList(list);

// 원본 리스트를 변경하면, 복사한 리스트도 변경된다.
list.set(0, "a");
System.out.println(newList); // [a, B, C]
```

### List.copyOf
원본 리스트와 독립적인 **새로운 불변 리스트**를 반환한다.
```java
List<String> list = Arrays.asList("A", "B", "C");
List<String> newList = List.copyOf(list);

// 원본 리스트를 변경해도, 복사한 리스트는 변경되지 않는다.
list.set(0, "a");
System.out.println(newList); // [A, B, C]
```

## 빈 List 생성
### Collections.emptyList
비어 있는 불변 리스트를 반환한다.
```java
List<String> list = Collections.emptyList();
```

## 단일 요소 List 생성
### Collections.singletonList
단일 요소의 불변 리스트를 반환한다.
```java
List<String> list = Collections.singletonList("A");
```

---
**Reference**<br>
- https://alwayspr.tistory.com/28#4.%20Collections.emptyList()-1
- https://ksh-coding.tistory.com/77#%F0%9F%8E%AF%204.%20List.copyOf%20-%20%EB%B0%A9%EC%96%B4%EC%A0%81%20%EB%B3%B5%EC%82%AC%20O%2C%20%EC%96%95%EC%9D%80%20%EB%B3%B5%EC%82%AC-1
