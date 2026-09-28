# STREAMS
UNIX 운영체제의 **STREAMS**는 사용자 프로세스와 장치 드라이버 사이의 데이터 처리 경로를 스트림으로 구성하는 I/O 프레임워크이다.

## 주요 구성 요소
### 스트림 헤드(Stream Head)
사용자 프로세스와 스트림을 연결하는 경계 부분이다. 프로세스가 `read()`, `write()` 등의 시스템콜을 사용하면 이를 스트림 모듈로 전달한다.

### 모듈(Module)
스트림 중간에 삽입되는 데이터 처리 모듈이다. 프로토콜 처리, 데이터 변환, 문자처리, 압축/해제 등 특정 기능을 수행한다.

### 드라이버(Driver)
스트림의 가장 아래에 위치하며 실제 장치와 통신하는 부분이다.

<img width="549" height="572" alt="13_14_STREAMS" src="https://github.com/user-attachments/assets/a4a88602-6b15-481c-b6d1-9cdc24d8737a" />

## 데이터 흐름
스트림에서는 데이터를 **메시지** 단위로 전달하며, 각 모듈의 **읽기 큐**(Read Queue), **쓰기 큐**(Write Queue)​를 통해 메시지가 전달된다.

- **하향 스트림**(Downstream, Write): 스트림 헤드에서 드라이버 방향으로 데이터 메시지가 흐른다. (출력)
- **상향 스트림**(Upstream, Read): 드라이버에서 스트림 헤드 방향으로 데이터 메시지가 흐른다. (입력)

## 장점 및 활용
- **모듈성 및 유연성**: 통신 프로토콜 등을 독립적인 모듈로 구현하고, 필요에 따라 동적으로 I/O 경로에 추가/제거할 수 있어 유지보수와 재사용성이 높다.

TCP/IP 프로토콜 스택 구현, 터미널 I/O 처리 등 복잡하고 계층적인 I/O 처리에 사용된다.

<img width="1024" height="881" alt="a42cbea2-6219-447e-a123-ee14e8a2c850" src="https://github.com/user-attachments/assets/d42df04c-e273-4a08-9ada-cfb00cfe9d2b" />

---
**Reference**
- https://wikidocs.net/312511
- https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/13_IOSystems.html
