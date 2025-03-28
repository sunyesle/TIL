# TCP/IP
TCP/IP는 패킷 통신 방식의 인터넷 프로토콜인 IP와 전송 제어 프로토콜인 TCP로 이루어져있다.

**IP**는 IP 주소를 기반으로 라우팅을 수행하며, 각 패킷을 목적지 호스트까지 전송하는 역할을 한다.

**TCP**는 IP위에서 동작하며, 모든 패킷을 안정적으로, 순서대로, 에러 없이 전송할 수 있게 해준다.
또한, TCP는 IP 주소뿐만 아니라 Port 번호를 사용하여 연결을 관리한다.

> **IP**는 **각 패킷**이 **올바른 목적지에 도달하는 것**에 중점을 두고, **TCP**는 **모든 패킷**의 **신뢰성 있는 전송**에 중점을 둔다.

## 3-Way Handshake
TCP로 이루어지는 모든 통신은 반드시 **3-Way Handshake**를 통해 시작한다.

![tcp_three_way_handshake](https://github.com/user-attachments/assets/763e229e-4171-4102-ba80-f305f0c17c99)

- `SYN`: 송신자가 수신자에게 연결을 요청하는 SYN 패킷을 보낸다. 이때 Port가 열려있어야 한다.
- `SYN/ACK`: 수신자가 연결 요청을 수락하고, 응답으로 SYN/ACK 패킷을 보낸다.
- `ACK`: 송신자가 연결 요청이 수락되었음을 확인하고, ACK 패킷을 보내 최종적으로 연결이 성립된다.

## 레이어별 동작 과정
### 데이터 전송 시
![데이터 전송](https://github.com/user-attachments/assets/fd412279-d214-457d-9e42-d9000c1c061e)
#### Application
애플리케이션이 전송할 데이터를 생성하고, write 시스템 콜을 호출해서 데이터를 보낸다. 시스템 콜을 호출하면 커널 영역으로 전환된다.
#### File
Linux나 Unix를 포함한 POSIX 계열 운영체제는 소켓을 file descriptor로 애플리케이션에 노출한다. POSIX 계열의 운영체제에서 소켓은 파일의 한 종류다.
#### Socket
커널 소켓은 두 개의 버퍼를 가지고 있다. 송신용으로 준비한 send socket buffer, 수신용으로 준비한 receive socket buffer이다.
write 시스템 콜을 호출하면 유저 영역의 데이터가 커널 메모리로 복사되고, send socket buffer의 뒷부분에 추가된다.
#### TCP
소켓과 연결된 TCB(TCP Control Block) 구조체가 있다. TCB에는 connection state, receive window, sequence 번호 등 TCP 연결 처리에 필요한 정보가 있다.

현재 TCP 상태가 데이터 전송을 허용하면 새로운 TCP segment, 즉 패킷을 생성한다.
TCP 상태에 따라 패킷을 한 개 이상 전송할 수 있다.

흐름제어 같은 이유로 데이터 전송이 불가능하면 시스템 콜은 여기서 끝나고, 유저 모드로 돌아간다.

TCP segment에는 TCP 헤더와 페이로드(payload)가 담겨있다.

페이로드의 최대 크기(MMS, Max Segment Size)는 최대 전송 단위(MTU, Maximum Transmission Unit)에 의해 결정된다.
```
MMS = MTU - IP 헤더 길이 - TCP 헤더 길이
```

#### IP
TCP segment에 IP 헤더를 추가하고, IP 라우팅을 한다.
IP 라우팅이란 목적지 IP 주소(destination IP)로 가기 위한 다음 장비의 IP 주소(next hop IP)를 찾는 과정을 말한다.

#### Ethernet
ARP(Address Resolution Protocol)를 사용해서 next hop IP의 MAC 주소를 찾는다. 그리고 Ethernet 헤더를 패킷에 추가한다.

IP routing을 하면 그 결과물로 next hop IP와 해당 IP로 패킷 전송할 때 사용하는 인터페이스(transmit interface, 혹은 NIC)를 알게 된다. 따라서 transmit NIC의 드라이버를 호출한다.

#### Driver
드라이버는 NIC 제조사가 정의한 드라이버-NIC 통신 규약에 따라 패킷 전송을 요청한다.

#### NIC
패킷 전송 요청을 받고, 메인 메모리에 있는 패킷을 자신의 메모리로 복사하고, 네트워크 선으로 전송한다.

NIC가 패킷을 전송할 때 NIC는 호스트 CPU에 인터럽트를 발생시킨다. 운영체제가 핸들러를 호출하고, 핸들러는 전송된 패킷을 운영체제에 반환한다.


### 데이터 수신 시
![데이터 수신](https://github.com/user-attachments/assets/3e52af34-ec40-4275-b58b-ac1e9775a1bf)
#### NIC
NIC가 패킷을 자신의 메모리에 기록한다.
패킷이 올바른지 검사하고, 호스트의 메모리 버퍼로 전송한다. 이후, 패킷 전송이 완료되면 운영체제에 인터럽트를 발생시킨다.

#### Driver
드라이버가 상위 레이어로 패킷을 전달하려면 운영체제가 이해할 수 있도록, 받은 패킷을 운영체제가 사용하는 패킷 구조체로 포장해야 한다.
드라이버는 포장한 패킷을 상위 레이어로 전달한다.

#### Ethernet
패킷이 올바른지 검사한다.

Ethernet 헤더의 ethertype 값을 보고 상위 프로토콜(네트워크 프로토콜)을 찾는다. IPv4 ethertype 값은 0x0800이다.

Ethernet 헤더를 제거하고 IP 레이어로 패킷을 전달한다.

#### IP
패킷이 올바른지 검사한다.

IP 라우팅을 해서 패킷을 로컬 장비가 처리해야 하는지, 아니면 다른 장비로 전달해야 하는지 판단한다.
로컬 장비가 처리해야 하는 패킷이라면 IP 헤더의 protocol 값을 보고 상위 프로토콜(트랜스포트 프로토콜)을 찾는다. TCP protocol 값은 6이다.

IP 헤더를 제거하고 TCP 레이어로 패킷을 전달한다.

#### TCP
패킷이 올바른지 검사한다.

패킷이 속하는 연결, 즉 TCB를 찾는다. 이때 `소스 IP, 소스 port, 타깃 IP, 타깃 port`를 식별자로 사용한다.
연결을 찾으면 프로토콜을 수행해서 받은 패킷을 처리한다. 새로운 데이터를 받았다면 receive socket buffer에 추가한다. TCP 상태에 따라 새로운 패킷(ex. ACK 패킷)을 전송할 수 있다.

여기까지 해서 TCP/IP 수신 패킷 처리 과정이 끝나게 된다.

이후 애플리케이션이 read 시스템 콜을 호출하면 커널 영역으로 전환되고, socket buffer에 있는 데이터를 유저 공간의 메모리로 복사해 간다. 복사한 데이터는 socket buffer에서 제거한다.
그리고 TCP를 호출한다. TCP는 socket buffer에 새로운 공간이 생겼기 때문에 recieve window를 증가시킨다. 프로토콜 상태에 따라 패킷을 전송하고 패킷 전송이 없으면 시스템 콜이 종료된다.

## 흐름 제어(Flow Control)
송신 측과 수신 측의 데이터 처리 속도 차이를 해결하기 위한 기법이다.

## 혼잡 제어(Congestion Control)
송신 측의 데이터 전달과 네트워크의 데이터 처리 속도 차이를 해결하기 위한 기법이다.

---
**Reference**<br>
- https://aws-hyoh.tistory.com/57
- https://better-together.tistory.com/110
- https://d2.naver.com/helloworld/47667
- http://www.ktword.co.kr/test/view/view.php?no=1899
