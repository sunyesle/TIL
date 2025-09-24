# System.arraycopy() 메서드
Java 표준 라이브러리에 포함된 네이티브 메서드로, 한 배열에서 다른 배열로 요소를 복사하는 기능을 제공한다.

## 특징
- **네이티브 메서드**: C/C++로 구현되어 JVM의 최적화된 성능을 제공한다.
- **빠른 속도**: 자바의 다른 배열 복사 방법보다 일반적으로 빠르다.
- **메모리 효율성**: 새 배열을 생성하지 않고 기존 배열로 직접 복사 가능하다.
- **부분 복사**: 배열의 특정 부분만 선택적으로 복사할 수 있다.
- **다양한 타입 지원**: 기본 데이터 타입과 객체 배역 모두 지원한다.

## 사용 방법
```java
public static native void arraycopy(Object src, int srcPos, Object dest, int destPos, int length);
```
- `src`: 원본 배열(복사할 데이터가 있는 배열)
- `srcPos`: 원본 배열에서 복사를 시작할 인덱스
- `dest`: 대상 배열(데이터가 복사될 배열)
- `destPos`: 대상 배열에서 복사를 시작할 인덱스
- `length`: 복사할 요소의 개수

### 예시
```java
int[] source = {1, 2, 3, 4, 5};
int[] destination = new int[5];

// source 배열의 내용을 destination 배열로 복사한다.
System.arraycopy(source, 0, destination, 0, source.length);

// 결과: destination = {1, 2, 3, 4, 5}
```

---
**Reference**<br>
- https://eunplay.tistory.com/118
