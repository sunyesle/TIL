# MMU(Memory Management Unit)
프로세스는 가상 주소를 사용하고, CPU는 실행 중 가상주소를 참조하게 된다.그러나 실제 메모리에 접근하려면 가상 메모리 주소를 물리 메모리 주소로 변환해야 한다.

이 역할을 담당하는 것이 **메모리 관리 장치**(MMU)이다.

MMU는 CPU와 메모리 사이에 위치하여 가상 메모리 주소와 물리 메모리 주소 간의 변환을 담당하는 컴퓨터 하드웨어 부품이다.

## MMU 동작 과정
![MMU](https://github.com/user-attachments/assets/eb71c392-fa18-4235-9eef-9ca0d03380f0)

(**p**: 페이지 번호, **f**: 프레임 번호, **d**: offset)

- virtual address를 page size로 나누면 몫은 page number, 나머지는 offset이 된다.
- page table에서 page number를 통해서 frame number를 찾아낸다.
- frame number에 frame size(=page size)를 곱하면 해당 프레임의 시작 주소를 얻을 수 있으며, 여기에 offset의 값을 더해주면 물리 주소를 구할 수 있다.

대부분의 컴퓨터는 페이지 테이블을 메인 메모리에 저장하고 **페이지 테이블 기준 레지스터**(**PTBR, Page-Table Base Register**)가 페이지 테이블을 가리키도록 한다.
다른 페이지 테이블을 사용하려면 이 레지스터만 변화시키면 되기 때문에 Context Switching 시간을 줄일 수 있다.

PTBR의 문제점은 메모리의 접근 시간이다.
물리 주소를 얻기 위해 메모리상의 페이지 테이블에 먼저 접근하고, 다시 실제 데이터에 접근해야 하므로 총 2번 메모리에 접근하게 된다.

## TLB(Translation Lookaside Buffer)

PTBR의 메모리 접근 속도 문제를 해결하기 위해 **TLB**(Translation Look-aside Buffer)이라 불리는 캐시가 사용된다.

TLB는 메모리 주소 변환을 위한 별도의 캐시 메모리로, 페이지 테이블에서 빈번히 참조되는 일부 엔트리를 캐싱하고 있다.
key-value 쌍으로 데이터를 관리하는 연관 메모리(associative memory)이며, key에는 page number, value에는 frame number가 대응한다.

CPU는 페이지 테이블보다 TLB를 우선적으로 참조하여, 만약 원하는 페이지가 TLB에 있는 경우 곧바로 frame number를 얻을 수 있고, 그렇지 않은 경우 메인 메모리에 있는 페이지 테이블로부터 frame number를 얻을 수 있다.

![TLB](https://github.com/user-attachments/assets/cffa34ad-d3c4-4eab-8aba-380352d36ff4)

---
**Reference**<br>
- https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/8_MainMemory.html
- https://rebro.kr/178
- https://github.com/kim-svadoz/TIL/blob/master/ComputerScience/OS/OS.md
- https://blog.naver.com/tlsrka649/223121968708
