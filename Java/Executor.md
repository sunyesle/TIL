# Executor

## 관련 개념
- `Runnable`: 결과를 반환하지 않는 작업
- `Callable`: 결과를 반환하는 작업
- `Future`: 미래에 완료된 Callable의 실행 결과를 받기 위해 사용된다.

## Executor 인터페이스 계층 구조
![Executor](https://github.com/user-attachments/assets/52738237-0548-49e5-b19c-168890e06a68)

- `Executor`: 작업 실행을 위한 인터페이스
- `ExecutorService`: 작업 등록과 실행을 위한 인터페이스
- `ScheduledExecutorService`: 특정 시간 이후 또는 주기적으로 작업을 실행할 수 있는 ExecutorService
- `Executors`: ExecutorService와 ScheduledExecutorService의 팩토리의 메서드 제공

---
**Reference**<br>
- https://mangkyu.tistory.com/259
- https://yangbox.tistory.com/28
