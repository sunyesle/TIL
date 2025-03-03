# ThreadPoolExecutor
`ThreadPoolExecutor`는 Java에서 제공하는 Thread Pool 구현체이다.

## 스레드 풀(Thread Pool)
스레드 풀이란 스레드를 미리 생성하고, 작업 요청이 발생할 때마다 미리 생성된 스레드로 해당 작업을 처리하는 방식을 의미한다.

스레드의 빈번한 생성 및 파괴로 인한 오버헤드를 줄일 수 있다.

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

corePoolSize 만큼의 스레드가 수행 중이라면 추가적인 요청에 대한 스레드는 workQueue에 저장된다.<br>
workQueue가 가득 차면 새로운 스레드가 생성되어 maximumPoolSize 까지 증가시킨다.<br>
corePoolSize를 초과하여 생성된 스레드는 keepAliveSeconds 만큼 idle 상태가 지속될 경우 자동으로 제거된다.

> ### 서비스 특성에 따른 설정 팁
> 일정한 양의 처리가 필요한 업무인 경우,
> corePoolSize와 maximumPoolSize를 업무 처리량에 맞춰서 동일하게 설정하고 대신 Queue를 적절하게 설정하여 처리 대기를 시키는 것이 좋다.
> 
> 반대로 처리량이 시간대별로 증감이 있는 경우,
> maximumPoolSize를 다르게 설정하고 대신 Queue의 크기를 작게 설정하여 추가적인 Thread 실행이 원활하게 이루어지도록 하는 것이 좋다.


---
**Reference**<br>
- https://blog.naver.com/bumsukoh/222175557879
- https://velog.io/@vanillacake369/Async-Size-%EC%84%A4%EC%A0%95-%EA%B8%B0%EC%A4%80%EC%97%90-%EB%8C%80%ED%95%B4-%EA%B3%A0%EB%AF%BC%ED%95%B4%EB%B3%B4%EC%9E%90-feat.ThreadPoolQueue
- https://hudi.blog/java-thread-pool/
- https://tecoble.techcourse.co.kr/post/2021-09-18-java-thread-pool/

