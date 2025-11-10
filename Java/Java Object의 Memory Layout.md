# Java Object의 Memory Layout
일반적으로 런타임 데이터 영역의 메모리 레이아웃은 JVM 사양에 포함되지 않기 때문에 구현에 따라 다른 전략을 사용할 수 있다.

이 글에서는 HotSpot JVM 구현을 기준으로 JVM이 힙에 객체와 배열을 어떻게 저장하는지 알아보려고 한다.

## OOP (Ordinary Object Pointer)
JVM은 객체를 가리키기 위해 OOP라는 데이터 구조를 사용한다.

OOP는 힙에 존재하는 실제 객체 구조체(`oopDesc`)를 가리키는 포인터(참조)로, 자바 코드에서의 참조 변수에 해당한다.

### oopDesc
[`oopDesc`](https://github.com/openjdk/jdk21/blob/890adb6410dab4606a4f26a942aed02fb2f55387/src/hotspot/share/oops/oop.hpp#L52)는 힙 영역에 할당되는 객체를 표현하는 구조체로, 객체 헤더를 구성하는 두 가지 주요 필드를 포함한다.
- `mark word`: 객체별 메타데이터
- `klass word`: 클래스 메타데이터 포인터

```hpp
class oopDesc {
  friend class VMStructs;
  friend class JVMCIVMStructs;
 private:
  volatile markWord _mark;
  union _metadata {
    Klass*      _klass;
    narrowKlass _compressed_klass;
  } _metadata;
```

모든 자바 객체는 `oopDesc`를 기반으로 한다.
- [`instanceOopDesc`](https://github.com/openjdk/jdk21/blob/master/src/hotspot/share/oops/instanceOop.hpp): 단일 객체
- [`arrayOopDesc`](https://github.com/openjdk/jdk21/blob/master/src/hotspot/share/oops/arrayOop.hpp): 배열 객체

## 객체의 메모리 레이아웃
힙 영역에 할당된 자바 객체의 메모리 레이아웃은 다음과 같다.

<img width="700" height="324" alt="img (1)" src="https://github.com/user-attachments/assets/7f82730f-5e4f-498c-b2ad-69334dafac5a" />

### 마크 워드 (mark word)
마크 워드는 데이터를 담고 있는 bit field로, 객체 별로 고유하게 관리되는 메타데이터가 저장된다.
- Lock 정보: 경량 락, 모니터 락 등 동기화 상태
- GC Age: GC 구현을 위한 객체의 나이
- HashCode: 객체의 해시코드, `hashCode()`가 처음 호출될 때 저장된다.
- Unused: 미사용 구간

### 클래스 워드 (klass word)
클래스 워드는 객체가 어떤 클래스의 인스턴스인지 나타내는 포인터로, `Klass` 구조체를 가리킨다.

`Klass` 구조체에는 다음과 같은 정보가 포함되어 있으며, Metaspace 영역에 저장된다.
- 필드 정보 (오프셋, 타입 등)
- 메서드 정보 (가상 메서드 테이블 등)
- 정적 필드 정보

참고로 자바의 `Class` 객체는 이 정보를 기반으로 생성된 복제본이다.

### 배열의 길이
자바 배열의 경우 배열 길이도 객체 헤더에 저장된다.
따라서 객체 헤더의 길이는 배열 여부에 따라 달라질 수 있다.

### 인스턴스 데이터와 패딩
인스턴스 데이터는 객체가 실제로 담고 있는 정보다. 현재 클래스와 부모 클래스에서 정의한 필드 데이터가 여기에 저장된다.

JVM은 객체를 8byte 경계에 맞게 정렬하려고 하며, 이를 위해 패딩을 삽입하기도 한다.
필요한 패딩을 줄이기 위해 필드 순서를 변경할 수 있으며, 필드 사이에 정렬 간격이 어긋날 경우 필요한 만큼 내부 패딩이 삽입된다.
또한, 객체 전체 크기를 맞추기 위해 마지막에도 패딩이 추가될 수 있다.

---
**Reference**
- https://mangkyu.tistory.com/448
- https://medium.com/@AlexanderObregon/where-object-headers-are-stored-and-what-they-contain-in-java-memory-5002b9fb6ee4
- https://leeyh0216.github.io/posts/trino-slice/
