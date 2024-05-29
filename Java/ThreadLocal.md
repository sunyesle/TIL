# ThreadLocal

스레드 지역 변수를 제공한다.

동일한 변수에 액세스하는 각각의 스레드에게 독립적인 저장공간을 제공함으로써, 동일한 스레드에서만 읽고 쓸 수 있는 변수를 생성할 수 있도록 도와준다.

ThreadLocal 인스턴스는 일반적으로 상태를 스레드(예: 사용자 ID 또는 트랜잭션 ID)와 연결하려는 클래스의 전용 정적 필드이다.


## 활용 예시

ThreadLocal은 싱글턴 패턴이나 전역 변수를 사용하는 구조에서 스레드 간에 변수가 임의로 공유되는 상황을 방지할 수 있다.

사용자 인증 정보, 트랜잭션 컨텍스트 등 스레드별로 독립적으로 유지해야 하는 정보를 저장하는 데 자주 사용된다.
- 웹 애플리케이션에서 사용자의 요청을 처리하는 각 스레드가 사용자별 세션 정보를 유지해야 할 때
- 데이터베이스 트랜잭션 관리에서 각 스레드가 트랜잭션 상태를 독립적으로 관리해야 할 때

ex) SecurityContextHolder, TransactionSynchronizationManager, RequestContextHolder

## 동작 방식
- 각 Thread 객체는 ThreadLocalMap을 필드로 가지고 있다.
- ThreadLocalMap은 ThreadLocal의 해시코드를 키로 하는 Map 구조를 가지고 있다.
- ThreadLocal은 현재 스레드의 ThreadLocalMap 객체를 가져와 현재 ThreadLocal를 키로 사용해 값을 찾는다.
![thread-local-class-in-java-2](https://github.com/sunyesle/TIL/assets/45172865/46261039-fb49-4793-b59a-92cc4bc5f29b)

## 주요 메서드
### Class ThreadLocal\<T>
- withInitial(Supplier<? extends S> supplier)
  - 람다식(lambda expression)을 통해 ThreadLocal 객체의 초기 값을 지정한다.
- get()
  - 현재 스레드의 지역 변수 값을 반환한다.
- set(T value)
  - 현재 스레드의 지역 변수 값을 설정한다.
- remove()
  - 현재 스레드의 지역 변수 값을 제거한다.

## 예제 코드

스프링에서 bean은 싱글턴으로 등록된다. 이렇게 하나만 있는 인스턴스의 필드에 여러 스레드가 동시에 접근하면 동시성 문제가 발생할 수 있다.

```java
public class FieldService {
    private static String nameStore;

    public String get() {
        return nameStore;
    }

    public void set(String name) {
        nameStore = name;
    }
}
```

```java
class FieldServiceTest {
    private FieldService service = new FieldService();

    @Test
    void fieldService_동시성_문제가_발생한다() {
        Thread threadA = new Thread(() -> {
            service.set("userA");
            System.out.println(Thread.currentThread().getName() + " : set userName=" + service.get());
            sleep(1000);
            System.out.println(Thread.currentThread().getName() + " : get userName=" + service.get());
        });
        threadA.setName("thread-A");

        Thread threadB = new Thread(() -> {
            service.set("userB");
            System.out.println(Thread.currentThread().getName() + " : set userName=" + service.get());
            sleep(1000);
            System.out.println(Thread.currentThread().getName() + " : get userName=" + service.get());
        });
        threadB.setName("thread-B");

        threadA.start();
        sleep(500);
        threadB.start();

        sleep(3000); //메인 쓰레드 종료 대기
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
> 실행결과
```
thread-A : set userName=userA
thread-B : set userName=userB
thread-A : get userName=userB
thread-B : get userName=userB
```
thread-A가 설정한 nameStore 값이 thread-B에 의해 덮어써져 동시성 문제가 발생한다.

<br>

ThreadLocal을 사용하여 동시성 문제를 해결해보자.
```java
public class ThreadLocalService {
    private ThreadLocal<String> nameStore = new ThreadLocal<>();

    public String get(){
        return nameStore.get();
    }

    public void set(String name) {
        nameStore.set(name);
    }
}
```

```java
class ThreadLocalTest {
    private ThreadLocalService threadLocalService = new ThreadLocalService();

    @Test
    void threadLocalService_동시성_문제가_발생하지_않는다() {

        Thread threadA = new Thread(() -> {
            threadLocalService.set("userA");
            System.out.println(Thread.currentThread().getName() + " : set userName=" + threadLocalService.get());
            sleep(1000);
            System.out.println(Thread.currentThread().getName() + " : get userName=" + threadLocalService.get());
        });
        threadA.setName("thread-A");

        Thread threadB = new Thread(() -> {
            threadLocalService.set("userB");
            System.out.println(Thread.currentThread().getName() + " : set userName=" + threadLocalService.get());
            sleep(1000);
            System.out.println(Thread.currentThread().getName() + " : get userName=" + threadLocalService.get());
        });
        threadB.setName("thread-B");

        threadA.start();
        sleep(500);
        threadB.start();

        sleep(3000); //메인 쓰레드 종료 대기
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
> 실행결과
```
thread-A : set userName=userA
thread-B : set userName=userB
thread-A : get userName=userB
thread-B : get userName=userB
```
동시성 문제가 발생하지 않는 것을 확인할 수 있다.

<br>

```java
public class NameContextHolder {
    private static ThreadLocal<String> nameStore = new ThreadLocal<>();

    public static String get(){
        return nameStore.get();
    }

    public static void set(String name) {
        nameStore.set(name);
    }
}
```

## ThreadLocal 사용 시 주의점

ThreadLocal과 ThreadPool을 함께 사용할 때는 특히 주의해야 한다.
ThreadPool을 사용하여 Thread를 재사용한다면, 이전에 ThreadLocal 객체를 통해 세팅한 데이터가 그대로 남아있게 된다.
반드시 ThreadLocal 사용이 끝난 후에 remove 메서드로 데이터를 제거해야 한다.

---


https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ThreadLocal.html

https://f-lab.kr/insight/understanding-and-utilizing-threadlocal

https://junhyunny.github.io/java/thread-local-class-in-java/

https://catsbi.oopy.io/3ddf4078-55f0-4fde-9d51-907613a44c0d

https://pamyferret.tistory.com/53
