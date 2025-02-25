# ThreadPoolExecutor
`ThreadPoolExecutor`는 Java에서 제공하는 Thread Pool 구현체이다.

## 동작 방식
`ThreadPoolExecutor`는 내부의 `BlockingQueue`에 작업을 등록해 둔다.
각각의 스레드는 작업을 할당받아 처리하고, 사용할 수 있는 스레드가 없다면 작업은 큐에서 대기하게 된다.

![ThreadPoolExecutor](https://github.com/user-attachments/assets/fc09056a-c3eb-45cd-9969-89b99bb40dd5)

`ThreadPoolExecutor`에는 두 가지의 저장 공간이 제공된다.
1. Thread 대기를 위한 큐
2. Thread Pool 저장 공간

`ThreadPoolExecutor`의 주요 옵션은 다음과 같다. 
- **corePoolSize**
  - `ThreadPoolExecutor`가 동시에 실행할 수 있는 스레드 수
- **maximumPoolSize**
  - `ThreadPoolExecutor`가 최대 수행할 수 있는 스레드 수
- **keepAliveTime**
  - corePoolSize보다 더 많은 스레드가 생성될 경우 추가된 Thread를 제거하기 위한 대기 시간
- **timeUnit**
  - keepAliveTime 옵션값을 위한 시간 단위(second, millisecond, ...)
- **workQueue**
  - `ThreadPoolExecutor`의 실행할 수 있는 Thread Pool이 없을 경우 스레드가 대기하기 위한 큐 지정

corePoolSize 만큼의 스레드가 수행 중이라면 추가적인 요청은 workQueue에 저장된다.<br>
workQueue가 다 차면 maximumPoolSize 만큼 스레드 풀 크기가 증가하게 된다.

---
**Reference**<br>
- https://blog.naver.com/bumsukoh/222175557879
