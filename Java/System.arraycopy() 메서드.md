자주 사용하는 앱에서 바로 AI를 사용해 보세요 … Gemini를 사용하여 초안을 생성하고 콘텐츠를 다듬고, Google의 차세대 AI가 지원되는 Gemini Pro를 이용하세요.
# System.arraycopy() 메서드
Java 표준 라이브러리에 포함된 네이티브 메서드로, 한 배열에서 다른 배열로 요소를 복사하는 기능을 제공한다.

## 특징
- **네이티브 메서드**: C/C++로 구현되어 JVM의 최적화된 성능을 제공한다.
- **빠른 속도**: 자바의 다른 배열 복사 방법보다 일반적으로 빠르다.
- **메모리 효율성**: 새 배열을 생성하지 않고 기존 배열로 직접 복사 가능하다.
- **부분 복사**: 배열의 특정 부분만 선택적으로 복사할 수 있다.
- **다양한 타입 지원**: 기본 데이터 타입과 객체 배열 모두 지원한다. (객체 배열 복사는 얕은 복사)

## 사용 방법
```java
public static native void arraycopy(Object src,  int srcPos,
                                    Object dest, int destPos,
                                    int length);
```
- `src`: 원본 배열(복사할 데이터가 있는 배열)
- `srcPos`: 원본 배열에서 복사를 시작할 인덱스
- `dest`: 대상 배열(데이터가 복사될 배열)
- `destPos`: 대상 배열에서 복사를 시작할 인덱스
- `length`: 복사할 요소의 개수

## 예시
### 배열 전체 복사
```java
int[] src = {1, 2, 3, 4, 5};
int[] dest = new int[src.length];

System.arraycopy(src, 0, dest, 0, src.length);

// dest = {1, 2, 3, 4, 5}
```

### 배열 일부 복사
```java
int[] src = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
int[] dest = new int[5];

System.arraycopy(src, 3, dest, 0, 5);

// dest = {4, 5, 6, 7, 8}
```

### 동일한 배열 내에서 요소 이동
```java
int[] arr = {1, 2, 3, 4, 5};

System.arraycopy(arr, 0, arr, 1, 3);

// arr = {1, 1, 2, 3, 5}
```

### 배열의 특정 요소 삭제
```java
int[] arr = {1, 2, 3, 4, 5};
int removeIdx = 2;

System.arraycopy(arr, removeIdx + 1, arr, removeIdx, arr.length - removeIdx - 1);
arr[arr.length - 1] = 0;

// arr = {1, 2, 4, 5, 0}
```


## 다른 복사 방법과 비교
### System.arraycopy()
- **특징**
  - JVM 네이티브 메서드를 호출한다.
  - 배열을 부분적으로 복사할 수 있다. (srcPos, destPos, length 등 세밀한 제어 가능)
  - 이미 만들어둔 배열에 복사 가능하다.
- **선택 기준**
  - 이미 만들어둔 대상 배열에 덮어쓸 때
  - 세밀한 제어가 필요할 때
  - 성능이 중요할 때

### Arrays.copyOf(), Arrays.copyOfRange()
- **특징**
  - 내부적으로 System.arraycopy를 호출한다.
  - 항상 새 배열을 생성한다.
  - 가독성이 좋다.
  - 크기를 늘려서 복사도 가능하다. (부족한 부분은 기본값으로 채움)
- **선택 기준**
  - 새 배열을 만들고 싶을 때
  - 코드 가독성이 중요할 때
  - 배열 크기를 바꿔야 할 때

### clone()
- **특징**
  - 원본과 동일한 길이의 새 배열을 생성한다.
  - 가독성이 좋다.
- **선택 기준**
  - 원본과 동일한 복사본이 필요할 때
  - 코드 가독성이 중요할 때

---
**Reference**<br>
- https://eunplay.tistory.com/118
