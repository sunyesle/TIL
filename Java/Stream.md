# Stream

Java의 Stream API는 일련의 데이터 흐름을 함수형 연산(람다)을 통해 표준화된 방법으로 처리할 수 있도록 지원한다.

스트림은 "**생성 -> 가공 -> 결과 생성**"의 구조로 구성되어 있다.

## 스트림 생성

### 배열 스트림
배열의 경우 `Arrays.stream` 메서드를 통해 스트림을 만들 수 있다.
```java
String[] arr = new String[]{"a", "b", "c"};
Arrays.stream(arr);
```

### 컬렉션 스트림
컬렉션 타입의 경우(List, Set 등) Collection 인터페이스에 추가된 디폴트 메서드 `stream`을 통해 스트림을 만들 수 있다.
```java
List<String> list = Arrays.asList("a", "b", "c");
list.stream();
```

### 비어있는 스트림
```java
Stream.empty();
```

### Stream.builder()
`builder` 메서드를 사용하면 스트림에 직접적으로 값을 넣을 수 있다.
```java
Stream.<String>builder()
        .add("a").add("b").add("c")
        .build();
```

### Stream.generate()
generate() 메서드를 사용하면 Supplier<T> 람다로 값을 넣을 수 있다.<br>
이때 생성되는 스트림은 크기가 무한하기 때문에 최대 크기를 제한해야 한다.
```java
Stream.generate(() -> Math.random()).limit(5);
```

### 기본 타입형 스트림
제네릭을 사용하지 않고 직접적으로 해당 타입의 스트림을 다룰 수도 있다.<br>
불필요한 오토박싱(auto-boxing)이 일어나지 않는다. 만약 필요하다면 `boxed` 메서드를 이용해서 박싱할 수 있다.
```java
IntStream.range(1, 5);
LongStream.rangeClosed(1, 5).boxed();
```
Random 클래스로 쉽게 난수 스트림을 생성할 수 있다.(IntStream, LongStream, DoubleStream)
```java
new Random().ints(3);
```

### 문자열 스트림
String을 이용해서 스트림을 생성할 수도 있다.
```java
IntStream charsStream = "abc".chars();
Stream<String> stringStream = Pattern.compile(", ").splitAsStream("a, b, c");
```

### 파일 스트림
`Files.lines` 메서드는 해당 파일의 각 라인을 String 타입의 스트림으로 만들어준다.
```java
Files.lines(Paths.get("file.txt"), Charset.forName("UTF-8"));
```

### 스트림 연결하기
`Stream.concat` 메서드를 이용해 두 개의 스트림을 연결해서 새로운 스트림을 만들 수 있다.
```java
Stream<String> stream1 = Stream.of("a", "b", "c");
Stream<String> stream2 = Stream.of("d", "e", "f");
Stream<String> concat = Stream.concat(stream1, stream2);
```

### 병렬 스트림
`parallelStream`, `parallel` 메서드를 통해 병렬 스트림을 만들 수 있다.
```java
List<String> list = Arrays.asList("a", "b", "c");
list.parallelStream();

String[] arr = new String[]{"a", "b", "c"};
Arrays.stream(arr).parallel();

IntStream.range(1, 5).parallel();
```
`sequential` 메서드로 병렬 스트림을 순차 스트림으로 변경할 수 있다.
```java
IntStream.range(1, 5).parallel().sequential();
```

## 스트림 가공
### Filtering
`filter`는 스트림 내 요소를 평가해서 걸러내준다.
```java
Stream<T> filter(Predicate<? super T> predicate);
```

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");
Stream<String> stream = names.stream()
        .filter(name -> name.contains("a"));
// [Elena, Java]
```
### Mapping
`map`은 스트림 내 요소들을 하나씩 특정 값으로 변환해 준다.
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");
Stream<String> stream = names.stream()
        .map(String::toUpperCase);
// [ERIC, ELENA, JAVA]
```

`flatMap`은 중첩 구조를 한단계 제거하고 단일 컬렉션으로 만들어준다.<br>
Stream을 반환하는 람다를 인자로 받는다.

다음과 같이 중첩된 리스트가 있다. flatMap을 사용해서 중첩 구조를 제거할 수 있다.
```java
List<List<String>> list =
        Arrays.asList(Arrays.asList("a", "b", "c"),
                      Arrays.asList("d", "e", "f"));

List<String> flatList = list.stream()
        .flatMap(Collection::stream)
        .toList();
// [a, b, c, d, e, f]
```

다음 예제에서는 학생들의 점수 평균을 구한다.
```java
List<Student> students = Arrays.asList(
        new Student(95, 73, 82),
        new Student(80, 55, 96),
        new Student(64, 95, 90));

students.stream()
        .flatMapToInt(student -> IntStream.of(student.getKor(), student.getEng(), student.getMath()))
        .average().ifPresent(avg -> System.out.println(avg));
```

---
**Reference**<br>
- https://futurecreator.github.io/2018/08/26/java-8-streams
- https://www.elancer.co.kr/blog/detail/255