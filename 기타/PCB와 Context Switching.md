# Process Control Block(프로세스 제어 블록)
CPU는 한 번에 하나의 프로세스만 실행할 수 있다.
그러나 운영체제는 CPU 스케줄링을 통해 여러 프로세스를 번갈아 실행함으로써 다수의 프로세스가 동시에 실행되는 것처럼 보이게 한다.

이때 운영체제가 프로세스를 관리하고 제어하기 위해서는 프로세스의 식별자, 상태 등의 메타데이터를 저장해놓아야 한다.
이러한 정보를 담고 있는 데이터 구조가 **PCB**다.

PCB는 프로세스 생성과 동시에 커널 영역에 생성되며, 프로세스가 종료되면 PCB 또한 제거된다.

## PCB 구조
운영체제에 따라 다를 수 있지만 일반적으로 PCB에는 다음과 같은 정보가 포함된다.
- **프로세스 식별자**(PID): 각 프로세스의 고유 식별자
- **프로세스 상태**: 생성, 준비, 실행, 대기 등 상태
- **프로그램 카운터**(PC): 이 프로세스가 다음에 실행할 명령어의 주소
- **CPU 레지스터**: CPU 레지스터 관련 정보
- **CPU 스케줄링 정보**: 우선순위, 최종 실행 시각, CPU 점유시간 등
- **입출력 상태 정보**: 프로세스에 할당된 입출력장치 목록, 사용 파일 목록 등
- **메모리 관리 정보**
- ...

## 프로세스 테이블
운영체제는 빠르게 각 PCB에 접근하기 위해 프로세스 테이블을 통해 각 PCB를 관리한다.
프로세스 테이블에는 PCB의 포인터가 저장된다.
프로세스 테이블은 연결 리스트로 관리되어 삽입/삭제가 용이하다.

![process_table](https://github.com/user-attachments/assets/46a56c4d-8d82-4c4f-a360-ae61ab1be671)

# Context Switching(문맥 교환)
프로세스 A가 프로세스 B에게 CPU를 넘겨줄 때 프로세스 A는 다시 실행될 때를 위해 기억해야 할 정보들이 있다. 이를 context라고 하며 PCB에 저장되어 있다.

context에는 프로그램 카운터(PC), CPU 레지스터들의 값, 메모리 정보 등의 정보를 백업한다.

Context Switching은 동작 중인 프로세스의 context를 PCB에 보관하고, 바꿀 프로세스의 context를 PCB로부터 복구하여 프로세스를 실행하는 것을 말한다.
이 과정을 여러 프로세스가 빠르게 번갈아 가면서 수행하기 때문에 프로세스가 동시에 실행되는 것처럼 보이게 된다.

![context_switch](https://github.com/user-attachments/assets/1c9bc38f-4fd2-42b4-8e4e-e612a316b400)

---
**Reference**<br>
- https://gyoogle.dev/blog/computer-science/operating-system/PCB%20&%20Context%20Switching.html
- https://yanghs6.github.io/posts/1003_process_state/
