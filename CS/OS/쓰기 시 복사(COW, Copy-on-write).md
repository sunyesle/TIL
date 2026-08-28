# 쓰기 시 복사(COW, Copy-on-write)
COW는 메모리 복사를 최적화하는 기법으로, 메모리를 복사해야 하는 상황에서 처음부터 실제 메모리를 복사하지 않고 동일한 물리 메모리 페이지를 공유하다가, 수정이 발생하는 경우에만 해당 페이지를 복사한다.

프로세스가 메모리를 복사할 때 처음부터 모든 페이지를 복사하면 실제로 사용되지 않는 페이지까지 복사하게 되어 불필요한 메모리와 시간이 소모될 수 있다. COW는 이러한 복사를 실제 수정이 발생할 때까지 지연시켜 메모리 사용량과 복사에 필요한 비용을 줄인다.

## 동작 과정
대표적으로 `fork()`를 사용하여 프로세스를 생성할 때 COW가 사용된다.

1. 부모 프로세스가 fork()를 호출하면 운영체제는 자식 프로세스를 생성한다.
2. 자식 프로세스는 부모의 페이지 테이블을 복사하여, 동일한 물리 메모리 영역을 공유한다.
3. 공유 중인 메모리 영역은 보호 비트를 읽기 전용으로 설정하여, 부모와 자식 프로세스가 직접 수정하지 못하도록 한다.
4. 부모 또는 자식 프로세스가 공유된 메모리 영역을 수정하려 하면 페이지 부재가 발생한다.
5. 운영체제는 해당 페이지의 내용을 새로운 프레임에 복사한다.
6. 쓰기를 시도한 프로세스의 페이지 테이블이 새 프레임을 가리키도록 업데이트하고 보호 비트를 읽기/쓰기로 변경한다.
7. 쓰기를 시도한 프로세스의 중단된 명령어를 재실행한다.

## 예시
<img width="728" height="305" alt="9_07_Page_C_Unmodified" src="https://github.com/user-attachments/assets/6fb0a2e5-f083-419d-9b29-17496a0935d0" /><br>
**프로세스1이 페이지C를 수정하기 전**

<br>

<img width="727" height="346" alt="9_08_Page_C_Modified" src="https://github.com/user-attachments/assets/4c7bb15b-6093-4cb8-8c05-09dceab4df9c" /><br>
**프로세스1이 페이지C를 수정한 후**

---
**Reference**
- https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/9_VirtualMemory.html
- https://miintto.github.io/docs/os-cow
