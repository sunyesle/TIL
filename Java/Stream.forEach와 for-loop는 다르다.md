# Stream.forEach와 for-loop는 다르다

`for-loop`는 break를 사용해 반복을 종료시킬 수 있다.
```java
for (int i = 0; i < 100; i++) {
    // Do something
    if (i > 50) {
        break;
    }
    System.out.println(i);
}
```

<br>

`Stream.forEach`를 사용할 경우 100번 모두 반복하며 조건을 확인하게 된다.<br>
또한 최종 연산인 forEach 내에 로직이 포함되어 있는 것은 Stream의 의도에 맞지 않는다.
```java
IntStream.range(1, 100).forEach(i -> {
    // Do something
    if (i > 50) {
        return;
    }
    System.out.println(i);
});
```

<br>

`Stream.filter`를 사용한다고 하더라도 스트림의 중간 연산은 지연 연산이기 때문에 100번 모두 검사를 하게 된다.<br>
// Do something 부분에서 복잡한 연산이 포함되어 있거나, 반복 조건이 수만 건 수행되는 경우 성능에 직접적인 영향을 줄 수 있다.
```java
IntStream.range(1, 100)
        // Do something
        .filter(i -> i < 5)
        .forEach(System.out::println);
```

<br>

`Stream.takeWhile`을 사용하여 스트림에서 반복 작업을 중단할 수 있다.
- `takeWhile` : 조건을 만족하는 요소까지만 선택하고 나머지는 버린다.
```java
IntStream.range(1, 100)
        // Do something
        .takeWhile(i -> i < 5)
        .forEach(System.out::println);
```

<br>

**스트림은 단순히 for문의 대체제가 아니다.**

**종료 조건이 있는 로직**을 구현하고 있다면 스트림을 사용할 때 주의해야 한다.
for문을 사용하거나 Stream.takeWhile 사용하여 불필요한 반복을 줄일 수 있다.

---
**Reference**<br>
- https://tecoble.techcourse.co.kr/post/2020-05-14-foreach-vs-forloop/
- https://www.popit.kr/java8-stream은-loop가-아니다/
- https://ksabs.tistory.com/236
