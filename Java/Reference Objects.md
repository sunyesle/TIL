# Reference Objects
## StrongReference
기본적인 참조 타입이다.
`StrongRefence`가 있는 객체는 GC 대상이 되지 않는다.
```java
Object strong = new Object();
```

## SoftReference
메모리 부족(OutOfMemory 직전) 상태에서만 GC 대상이 된다.
메모리가 허용되는 한 데이터를 보존해야 할 때 적합하다.
주로 캐시 구현에 사용된다.
```java
Object o = new Object();
SoftReference<Object> ref = new SoftReference<>(o);

// GC 전
System.out.println(ref.get()); // java.lang.Object@4517d9a3

o = null; // Strong reference 제거
System.gc(); // GC 실행

// GC 후 (메모리 여유가 있는 경우)
System.out.println(ref.get()); // java.lang.Object@4517d9a3
```
`SoftReference`는 GC가 발생해도 메모리가 충분하면 객체는 유지되며,
메모리가 부족할 때 GC가 객체를 수거한다.

## WeakReference
GC가 돌 때마다 회수된다.
메모리가 여유 있어도 모두 회수되며, 메모리 누수를 방지하는데 유용하다.
`WeakHashMap`, 메모리 누수 방지용 캐시 등에 사용된다. 캐시 적중률은 낮지만, 메모리 효율이 높다.
```java
Object o = new Object();
WeakReference<Object> ref = new WeakReference<>(o);

// GC 전
System.out.println(ref.get()); // java.lang.Object@4517d9a3

o = null; // Strong reference 제거
System.gc(); // GC 실행

// GC 후
System.out.println(ref.get()); // null
```
`WeakReference`는 객체 생존을 보장하지 않는다.
필요할 때만 일시적으로 참조해 두는 용도로 사용된다.

## PhantomReference
GC 대상이 되었을 때, 생성 시 파라미터로 넘긴 `ReferenceQueue`에 `enqueue`된다.
`DirectBuffer`처럼 JVM에서 관리하지 않는 네이티브 리소스를 정리할 때 유용하다.
```java
Object o = new Object();
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> ref = new PhantomReference<>(o, queue);

// GC 전
System.out.println(ref.get()); // null
System.out.println(queue.poll()); // null

o = null; // StrongReference 제거
System.gc(); // GC 실행

// GC 후
System.out.println(ref.get()); // null
System.out.println(queue.poll()); // java.lang.ref.PhantomReference@4517d9a3
```
`PhantomReference`는 객체의 실제 참조를 유지하지 않으며, `get()` 메서드는 항상 `null`을 반환한다.
객체 접근 목적이 아니라 GC 시점을 감지하는 용도로 사용된다.

---
**Reference**
- https://jonghoonpark.com/2024/05/05/java-reference-type
- https://liltdevs.tistory.com/182
