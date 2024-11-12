# Stream

Java의 Stream API는 일련의 데이터 흐름을 함수형 연산(람다)을 통해 표준화된 방법으로 처리할 수 있도록 지원한다.

스트림은 "**생성 -> 가공 -> 결과 생성**"의 구조로 구성되어 있다.

## 목차
1. **[스트림 생성](#스트림-생성)**
   - [배열 스트림](#배열-스트림)
   - [컬렉션 스트림](#컬렉션-스트림)
   - [비어있는 스트림](#비어있는-스트림)
   - [Stream.builder()](#streambuilder)
   - [Stream.generate()](#streamgenerate)
   - [기본 타입형 스트림](#기본-타입형-스트림)
   - [문자열 스트림](#문자열-스트림)
   - [파일 스트림](#파일-스트림)
   - [스트림 연결하기](#스트림-연결하기)
   - [병렬 스트림](#병렬-스트림)

2. **[스트림 가공](#스트림-가공)**
   - [Filtering](#filtering)
   - [Mapping](#mapping)
   - [Sorting](#sorting)
   - [Iterating](#iterating)

3. **[결과 생성](#결과-생성)**
   - [Calculating](#calculating)
   - [Reduction](#reduction)
   - [Collecting](#collecting)
   - [Matching](#matching)
   - [Iterating](#iterating)

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

### Sorting
`sorted`는 스트림 내 요소들을 정렬해 준다.
```java
IntStream sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```
```java
List<String> lang = Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");

// 인자 없이 호출할 경우 오름차순으로 정렬
lang.stream()
        .sorted()
        .toList();
// [Go, Groovy, Java, Python, Scala, Swift]

// 역순 정렬
lang.stream()
        .sorted(Comparator.reverseOrder())
        .toList();
// [Swift, Scala, Python, Java, Groovy, Go]

// 문자열 길이를 기준으로 정렬
lang.stream()
        .sorted(Comparator.comparingInt(String::length))
        .toList();
// [Go, Java, Scala, Swift, Groovy, Python]

lang.stream()
        .sorted((s1, s2) -> s2.length() - s1.length())
        .toList();
// [Groovy, Python, Scala, Swift, Java, Go]
```

### Iterating
`peek`은 스트림 내 각 요소들을 대상으로 특정 작업을 수행한다.<br>
주로 디버깅을 위해 사용한다.
```java
int sum = IntStream.of(1, 3, 5, 7, 9)
        .peek(System.out::println)
        .sum();
```

## 결과 생성
### Calculating
최소, 최대, 합, 평균 등의 결과를 만들어 낼 수 있다.
```java
long count = IntStream.of(1, 3, 5, 7, 9).count();
long sum = IntStream.of(1, 3, 5, 7, 9).sum();
```

최소, 최대, 평균은 스트림이 비어있는 경우 표현할 수 없기 때문에 Optional을 이용해 리턴한다.
```java
OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();
OptionalDouble everage = IntStream.of(1, 3, 5, 7, 9).average();
```

스트림에서 `ifPresent` 메서드를 이용해서 Optional을 처리할 수 있다.
```java
DoubleStream.of(1.1, 2.2, 3.3, 4.4, 5.5)
        .average()
        .ifPresent(System.out::println);
```

### Reduction
`reduce`는 스트림의 요소를 단일 값으로 결합하는 데 사용된다.
```java
Optional<T> reduce(BinaryOperator<T> accumulator);
T reduce(T identity, BinaryOperator<T> accumulator);
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
```
**reduce 메서드 파라미터**
- `accumulator`: 각 요소를 처리하는 계산로직. 각 요소가 올때마다 중간결과를 생성하는 로직
- `identity`: 계산을 위한 초기값 스트림이 비어서 계산할 내용이 없더라도 이 값은 리턴한다.
- `combiner`: 병렬 스트림에서 부분 결과들을 결합하는데 사용

```java
OptionalInt sum = IntStream.of(1, 2, 3)
        .reduce((a, b) -> {
            return a + b;
        });
// 6

int sum2 = IntStream.of(1, 2, 3)
        .reduce(10, Integer::sum);
// 16

Integer reducedParallel = Stream.of(1, 2, 3)
        .parallel()
        .reduce(10,
                Integer::sum,
                (a, b) -> {
                    System.out.println("combiner was called");
                    return a + b;
                });
System.out.println("result: " + reducedParallel);
// combiner was called
// combiner was called
// result: 36
```

### Collecting 
`collect` 메서드는 스트림의 요소를 컬렉션, 리스트, 세트 또는 맵과 같은 다른 형태로 변환하는데 사용된다.<br>
Collector 타입의 인자를 받으며, 자주 사용하는 작업은 Collectors 클래스에서 제공하고 있다.

예제에서 사용할 리스트는 다음과 같다.
```java
List<Product> products = Arrays.asList(
        new Product("apple", 1000),
        new Product("orange", 2000),
        new Product("lemon", 1000),
        new Product("bread", 3000),
        new Product("sugar", 2000)
);
```

#### Collectors.toList()
스트림에서 작업한 결과를 담을 리트스로 반환한다.
```java
List<String> collectorCollection = products.stream()
        .map(Product::getName)
        .collect(Collectors.toList()); // 'toList()' 로 간결하게 표현 가능하다.
```

#### Collectors.joining()
스트림에서 작업한 결과를 하나의 String으로 이어 붙여 반환한다.
```java
public static Collector<CharSequence, ?, String> joining();
public static Collector<CharSequence, ?, String> joining(CharSequence delimiter);
public static Collector<CharSequence, ?, String> joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix);
```
**Collectors.joining 메서드 파라미터**
- `delimiter` : 각 요소 중간에 들어가 요소를 구분시켜주는 구분자
- `prefix` : 결과 맨 앞에 붙는 문자
- `suffix` : 결과 맨 뒤에 붙는 문자

```java
String listToString = products.stream()
        .map(Product::getName)
        .collect(Collectors.joining());
// appleorangelemonbreadsugar

String listToString2 = products.stream()
        .map(Product::getName)
        .collect(Collectors.joining(", ", "<", ">"));
// <apple, orange, lemon, bread, sugar>
```

#### Collectors.averagingInt()
숫자 값의 평균을 반환한다.
```java
Double averageAmount = products.stream()
        .collect(Collectors.averagingInt(Product::getPrice));
// 2700.0
```

#### Collectors.summingInt()
숫자 값의 합을 반환한다.
```java
Integer summingAmount = products.stream()
        .collect(Collectors.summingInt(Product::getPrice));
// 13500
```

IntStream으로 바꿔주는 mapToInt 메서드를 사용해서 더 간단하게 표현할 수 있다.
```java
Integer summingAmount2 = products.stream()
        .mapToInt(Product::getPrice)
        .sum();
```

#### Collectors.summarizingInt()
통계 정보들을 반환한다.
```java
IntSummaryStatistics statistics = products.stream()
        .collect(Collectors.summarizingInt(Product::getPrice));
// IntSummaryStatistics{count=5, sum=13500, min=1000, average=2700.000000, max=4000}
```

위에서 살펴본 averaging, summing, summarizing 메소드는 각 기본 타입(int, long, double)별로 제공된다.

#### Collectors.partitioningBy()
평가 함수 리턴 값(true, false)을 기준으로 스트림내 요소들을 두개의 그룹으로 분할 할 수 있다. (스트림 2분할)
```java
Map<Boolean, List<Product>> mapPartitioned = products.stream()
        .collect(Collectors.partitioningBy(el -> el.getPrice() < 2000));
// {false=[Product(name=orange, price=2000), Product(name=bread, price=3000), Product(name=sugar, price=2000)],
//  true=[Product(name=apple, price=1000), Product(name=lemon, price=1000)]}

#### Collectors.groupingBy()
특정한 기준으로 요소들을 그룹지을 수 있다. (스트림 n분할)
```java
Map<Integer, List<Product>> collectorMapOfLists = products.stream()
        .collect(Collectors.groupingBy(Product::getPrice));
// {2000=[Product(name=orange, price=2000), Product(name=sugar, price=2000)],
//  3000=[Product(name=bread, price=3000)],
//  1000=[Product(name=apple, price=1000), Product(name=lemon, price=1000)]}
```

#### Collectors.collectingAndThen()
collect한 결과를 대상으로 추가 작업을 진행할 수 있다.
```java
Set<Product> unmodifiableSet = products.stream()
        .collect(Collectors.collectingAndThen(Collectors.toSet(), Collections::unmodifiableSet));
```

#### Collector.of()
직접 Collector를 만들 수도 있다.
```java
public static<T, R> Collector<T, R, R> of(
        Supplier<R> supplier,
        BiConsumer<R, T> accumulator,
        BinaryOperator<R> combiner,
        Collector.Characteristics... characteristics) {
```
**Collector.of 메서드 파라미터**
- `supplier`: 결과 객체 생성(초기화)
- `accumulator`: 결과 객체에 요소를 추가하는 동작을 정의
- `combiner`: 병렬 스트림에서 부분 결과들을 결합하는데 사용

```java
Collector<Product, ?, LinkedList<Product>> toLinkedList =
        Collector.of(LinkedList::new,
                LinkedList::add,
                (l1, l2) -> {
                    l1.addAll(l2);
                    return l1;
                });
LinkedList<Product> linkedList = products.stream()
        .collect(toLinkedList);
```

### Matching
스트림의 요소들이 특정 조건에 만족하는지 조사할 수 있다.
```java
boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
```

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

boolean anyMatch = names.stream()
        .anyMatch(name -> name.contains("a")); // true
boolean allMatch = names.stream()
        .allMatch(name -> name.length() > 3 ); // true
boolean noneMatch = names.stream()
        .noneMatch(name -> name.endsWith("s")); // true
```

### Iterating
`forEach`는 요소를 돌면서 실행되는 스트림 연산의 최종 작업이다.<br>
스트림 계산 결과 보고(주로 print)할 때만 사용하는 것이 좋다.<br>
`forEach`에 로직이 들어가 있다면 forEach가 아니라 스트림의 중간 연산(filter,map)을 활용하거나, 반복문을 사용하는 것이 올바른 방향이다.

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

names.stream().forEach(System.out::println);
// EricElenaJava
```

---
**Reference**<br>
- https://futurecreator.github.io/2018/08/26/java-8-streams
- https://www.elancer.co.kr/blog/detail/255
