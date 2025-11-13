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

<img width="700" alt="object_memory_layout" src="https://github.com/user-attachments/assets/4290e3dc-8a3f-4fb8-8e0e-c25e7d4b8a0d" />

### 마크 워드 (mark word)
마크 워드는 데이터를 담고 있는 bit field로, 객체 별로 고유하게 관리되는 메타데이터가 저장된다.
- Lock 정보: 경량 락, 모니터 락 등 동기화 상태
- GC Age: GC 구현을 위한 객체의 나이
- HashCode: 객체의 해시코드, `hashCode()`가 처음 호출될 때 저장된다.
- Unused: 미사용 구간

마크 워드의 크기는 64비트 JVM에서는 64비트, 32비트 JVM에서는 32비트이다.
64비트 기준 마크 워드의 구조는 다음과 같다.
```
 ┌───────────────────────────────┬─────────────────────────┬────┬────┐
 │          Unused               │         Hash Code       │Age │Lock│
 │          (27b)                │           (31b)         |(4b)│(2b)│
 └───────────────────────────────┴─────────────────────────┴────┴────┘
```
64비트 내에 런타임 데이터를 다 담을 수 없기때문에, 최대한 효율적으로 사용해야 한다.
이를 위해 마크 워드의 데이터 구조는 동적으로 의미가 달라진다.

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

**HotSpot JVM 기본 할당 전략**
- 상속 관계를 따라 **부모 클래스의 필드** → **자식 클래스 필드** 순으로 배치된다.
- 같은 클래스 내에서는 **큰 타입** → **작은 타입** → **참조 타입** 순서로 배치된다.
  - `long`, `double` → `int` → `short`, `char` → `byte`, `boolean` → `참조 타입`
- 8바이트 경계에 맟추기 위해, 필드 사이에 패딩을 삽입한다.
- 객체 전체 크기가 8바이트의 정수배가 되도록, 마지막에 패딩을 삽입한다.

## 동적으로 변하는 마크 워드
객체 헤더는 객체 생성 이후 고정되는 것이 아니라, JVM의 동작에 따라 변경된다.
동기화, 가비지 컬렉션 등에 의해 마크 워드가 변경될 수 있다.

다음과 같이 헤더의 마지막 2비트 값에 따라 의미가 달라진다.

| 주요 정보 저장 영역                   | Lock bits | Lock State       | 의미                    |
|-------------------------------|----------|------------------|-----------------------|
| 해시코드, GC용 age             | 01       | Unlocked         | 락 없음                 |
| 경량 락 포인터(스레드 스택의 Lock Record) | 00       | Lightweight Lock | 경량 락          |
| 중량 락 포인터(모니터 객체 주소)           | 10       | Heavyweight Lock | 중량 락 |
| 포워딩 포인터                 | 11       | GC Flag          | GC 관련 플래그             |

### synchronized
`synchronized`가 사용될 때, JVM은 어떤 스레드가 잠금을 가지고 있는지 추적해야 한다.
이때 최적화를 위해 **경량 락** → **중량 락**으로 계층화된 락 매커니즘을 제공한다.

#### 1. Unlocked
객체 헤더의 마지막 2비트(Lock bits)가 `01`일 경우 락이 걸려있지 않은 상태이다.
이때 마크 워드에는 `객체의 해시코드, GC age 등 메타데이터`가 저장되어 있다.

#### 2. 경량 락(Lightweight Lock)
락이 없는 상태에서 `synchronized` 블록에 진입하면 JVM은 경량 락을 시도한다.
1. 현재 스레드의 스택 프레임에 `Lock Record`를 생성한다. `Lock Record`에는 잠금을 걸 객체의 원래 마크 워드 값을 복사해 둔다.
2. CAS(Compare-And-Swap) 연산을 통해 객체의 마크 워드를 `Lock Record의 포인터 + 00 (Lock bits)`로 변경하려고 시도한다.

만약 변경에 성공하면 현재 스레드가 락을 획득하게 된다, 실패했다면 다른 스레드가 이미 동일 객체를 잠그려고 했다는 것을 의미한다.

#### 3. 중량 락(Heavyweight Lock)
경량 락 획득에 실패하면, JVM은 잠시 스핀(Spin)을 돌면서 락이 풀리길 기다린다. 스핀 중에도 락이 해제되지 않으면, 중량 락으로 전환된다.

JVM은 `ObjectMonitor` 객체를 생성하고, 마크 워드를 `ObjectMonitor 객체의 포인터 + 10 (lock bits)`로 교체한다.
이후에는 OS 수준의 모니터(커널 mutex)로 락을 관리한다.

---
**Reference**
- https://mangkyu.tistory.com/448
- https://medium.com/@AlexanderObregon/where-object-headers-are-stored-and-what-they-contain-in-java-memory-5002b9fb6ee4
- https://leeyh0216.github.io/posts/trino-slice/
- https://medium.com/@AlexanderObregon/where-object-headers-are-stored-and-what-they-contain-in-java-memory-5002b9fb6ee4
- https://velog.io/@aplbly/JAVA-%EB%B0%94%EC%9D%B4%ED%8A%B8%EC%BD%94%EB%93%9C
- https://velog.io/@tlarbals824/Deep-Dive-into-Synchronized-Biased-lock-Lightweight-lock-heavyweight-lock
