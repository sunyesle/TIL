# Java Object의 Memory Layout
일반적으로 런타임 데이터 영역의 메모리 레이아웃은 JVM 사양에 포함되지 않기 때문에 구현에 따라 다른 전략을 사용할 수 있다.

이 글에서는 HotSpot JVM 구현을 기준으로 JVM이 힙에 객체와 배열을 어떻게 저장하는지 알아보려고 한다.

## OOP
JVM은 객체를 가리키기 위해 위해 Ordinary Object Pointer(OOP)라는 데이터 구조를 사용한다.

HotSpot 내부에서는 다음과 같이 정의되어 있다.
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
[`oopDesc`](https://github.com/openjdk/jdk21/blob/890adb6410dab4606a4f26a942aed02fb2f55387/src/hotspot/share/oops/oop.hpp#L52)는 C++로 작성되어 있으며,
객체 헤더의 정의를 포함하고 있다.

`oopDesc`는 두 가지 주요 필드(객체 헤더)로 구성되어 있다.
- `mark word`: 객체별 메타데이터
- `klass word`: 클래스 메타데이터 포인터

모든 객체는 모두 `oopDesc`라는 클래스를 기반으로 한다.
- [`instanceOopDesc`](https://github.com/openjdk/jdk21/blob/master/src/hotspot/share/oops/instanceOop.hpp): 단일 객체
- [`arrayOopDesc`](https://github.com/openjdk/jdk21/blob/master/src/hotspot/share/oops/arrayOop.hpp): 배열 객체

## 객체의 메모리 레이아웃
<img width="700" height="324" alt="img (1)" src="https://github.com/user-attachments/assets/7f82730f-5e4f-498c-b2ad-69334dafac5a" />

### 마크 워드 (mark word)
마크 워드는 데이터를 담고 있는 bit field로, 객체 별로 고유하게 관리되는 메타데이터가 저장된다.
- Lock 정보: 경량 락, 모니터 락 등 동기화 상태
- GC Age: GC 구현을 위한 객체의 나이
- HashCode: 객체의 해시코드, `hashCode()`가 처음 호출될 때 저장된다.
- Unused: 미사용 구간

### 클래스 워드 (klass word)
클래스 워드는 포인터로, 해당 객체의 클래스 메타데이터가 존재하는 메모리 주소를 가리킨다.
- 클래스 정보 (InstanceKlass 구조체 주소): 해당 객체가 어떤 클래스의 인스턴스인지 나타낸다.
- VTable (Virtual Method Table) 포인터: 가상 메서드 테이블 주소. 다형성을 지원하기 위한 메서드 테이블
- 필드 정보: 필드의 오프셋, 타입 정보
- 정적 필드 정보: 해당 클래스의 정적 필드에 대한 정보

Klass는 JVM 내부적으로 빌드를 하고 클래스를 로딩할 때 필요한 메타데이터이다.
모든 클래스가 공유하는 메타데이터 정보이기 때문에 JVM Metaspace 영역에 저장 및 관리된다.

`Class` 객체는 이 정보를 바탕으로 만든 복제본이다.

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
