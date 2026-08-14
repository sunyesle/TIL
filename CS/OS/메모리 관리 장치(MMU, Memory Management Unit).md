# MMU와 주소 변환
프로그램이 실행될 때 CPU는 논리 주소를 사용하여 메모리에 접근한다.
그러나 실제 데이터는 물리 메모리에 저장되어 있으므로, 논리 주소를 물리 주소로 변환해야 한다.

## 메모리 관리 장치(MMU, Memory Management Unit)
MMU는 CPU가 생성한 논리 주소를 물리 주소로 변환하는 하드웨어 장치이다.

런타임 바인딩에서는 프로그램이 실행되는 동안 주소 변환이 이루어져야 하므로, CPU가 메모리에 접근할 때마다 MMU가 주소 변환을 수행한다.

MMU는 메모리 관리 방식에 따라 다양한 형태를 갖는다.

## 재배치 레지스터(Relocation Register)
<img width="586" height="419" alt="8_04_DynamicRelocation" src="https://github.com/user-attachments/assets/7f82bb9c-c90f-4987-a537-5460793af796" />

가장 단순한 형태의 주소 변환에서는 재배치 레지스터를 사용할 수 있다.

재배치 레지스터에는 프로세스의 시작 주소를 저장한다.
CPU가 논리 주소를 사용하여 메모리에 접근할 때마다 논리 주소에 재배치 레지스터의 값을 더해서 최종 물리 주소를 구한다.

## 페이지 테이블(Page Table)
<img width="797" height="483" alt="8_10_PagingHardware" src="https://github.com/user-attachments/assets/d851510f-4250-4c16-9b99-37948871cf43" />

- **p**: 페이지 번호(page number)
- **f**: 프레임 번호(frame number)
- **d**: 페이지 오프셋(page offset)

> **물리 주소 = (프레임 번호 × 페이지 크기) + 페이지 오프셋**

1. 논리 주소에서 페이지 번호와 페이지 오프셋을 추출한다. 논리 주소를 페이지 크기로 나누었을 때 몫은 페이지 번호, 나머지는 페이지 오프셋이 된다.
2. 페이지 테이블에서 페이지 번호를 이용해 해당 페이지가 할당된 프레임 번호를 찾는다.
3. 프레임 번호에 페이지 크기를 곱해서 해당 프레임의 시작 물리 주소를 구한다.
4. 프레임의 시작 물리 주소와 페이지 오프셋을 더해서 최종 물리 주소를 구한다.

대부분의 컴퓨터는 페이지 테이블을 주기억장치에 저장하고 페이지 테이블 기준 레지스터(PTBR, Page-Table Base Register)가 페이지 테이블의 시작 주소를 가리키도록 한다. 프로세스가 변경되면 PTBR 값을 새로운 페이지 테이블의 시작 주소로 변경하여 다른 페이지 테이블을 사용할 수 있다.

하지만 페이지 테이블이 주기억장치에 저장되어 있기 때문에 주소 변환 과정에서 추가적인 메모리 접근이 발생하는 문제가 있다. 물리 주소를 얻기 위해 먼저 메모리에 저장된 페이지 테이블에 접근하여 프레임 번호를 확인한 후, 변환된 물리 주소를 이용해 실제 데이터에 접근해야 하므로 총 2번 메모리에 접근하게 된다.

## TLB(Translation Lookaside Buffer)
페이지 테이블이 주기억장치에 저장되어 있어 추가적인 메모리 접근이 발생하는 문제를 해결하기 위해 TLB라는 캐시가 사용된다.

TLB는 메모리 주소 변환을 위한 캐시 메모리로, 페이지 테이블의 일부 엔트리를 저장한다. key-value 쌍으로 데이터를 관리하는 연관 메모리(associative memory)이며, key에는 페이지 번호, value에는 프레임 번호가 대응한다.

MMU는 페이지 테이블을 조회하기 전에 TLB를 먼저 확인한다. 원하는 페이지가 TLB에 있는 경우 곧바로 프레임 번호를 얻을 수 있다. 그렇지 않은 경우 주기억장치에 있는 페이지 테이블을 조회하여 프레임 번호를 얻는다. 이후 해당 페이지 테이블 엔트리를 TLB에 저장하여 다음 접근에서 사용할 수 있다.

<img width="704" height="540" alt="8_14_PagingHardware" src="https://github.com/user-attachments/assets/cffa34ad-d3c4-4eab-8aba-380352d36ff4" />

---
**Reference**<br>
- https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/8_MainMemory.html
- https://rebro.kr/178
- https://github.com/kim-svadoz/TIL/blob/master/ComputerScience/OS/OS.md
- https://blog.naver.com/tlsrka649/223121968708
