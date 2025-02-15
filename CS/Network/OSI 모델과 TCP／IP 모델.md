# OSI 모델과 TCP/IP 모델

![Image](https://github.com/user-attachments/assets/5fa2e69a-47ca-4051-bb57-f342c80e3b08)

## OSI 7계층 모델
국제표준화기구(ISO)에서 개발한 네트워크 모델이다.<br>
네트워크 통신 기능을 7개의 계층으로 나누고, 각 계층의 역할과 표준을 정하였다.

| 계층  | 이름                        | 설명                               |
|------|-----------------------------|------------------------------------|
| 7계층 | 응용 계층 (Application Layer) | 최종 사용자의 응용 프로그램과 직접적으로 관련된다. |
| 6계층 | 표현 계층 (Presentation Layer) | 데이터 형식을 변환한다. |
| 5계층 | 세션 계층 (Session Layer) | 통신 세션을 관리한다. |
| 4계층 | 전송 계층 (Transport Layer) | 데이터 전송의 신뢰성을 보장한다. |
| 3계층 | 네트워크 계층 (Network Layer) | 데이터 패킷의 라우팅을 담당한다. |
| 2계층 | 데이터 링크 계층 (Data Link Layer) | 두 장치 간의 데이터 전송을 처리한다. |
| 1계층 | 물리 계층 (Physical Layer) | 하드웨어 전송 기술을 다룬다. |

## TCP/IP 5계층 모델
미국 국방성(DoD)에서 정의한 네트워크 통신 표준 모델이다.

이미지 가운데의 TCP/IP 모델은 최초로 정의된 **TCP/IP 모델**이다.<br>
OSI 7계층을 간소화해서 4계층으로 표현한다.

이미지 오른쪽의 TCP/IP 모델은 **TCP/IP Updated 모델**이다.<br>
Link 계층을 두 레이어로 세분화하고, Internet 계층의 명칭을 Network로 변경하였다.

## TCP/IP 5계층
| Layer Number | Layer Name  | Addressing  | Protocol Data Unit | Protocol                  |
|--------------|-------------|-------------|--------------------|---------------------------|
| L5           | Application | -           | Message            | HTTP, SSH, FTP, SMTP, POP |
| L4           | Transport   | Port Number | Segment, Datagram  | TCP, UDP                  |
| L3           | Network     | IP Address  | Packet             | IP                        |
| L2           | Data-Link   | MAC Address | Frame              | IEEE 802, Ethernet, Wi-Fi |
| L1           | Physical    | -           | Bit, Signal        | -                         |

### L5 | Application
- 프로그램 구현체와 사용자 인터페이스를 의미한다.
- OS에서 제공하는 L4 API를 활용해 통신 프로그램이 구현된다.
- HTTP, SSH, FTP, SMTP, POP 등 다양한 프로토콜이 활용된다.

### L4 | Transport
> process-to-process delivery
- 포트 번호를 사용하여 최종 도착지인 프로세스까지 데이터를 전달한다.
- OS 커널에 구현되어 있다.
- 패킷 전송 프로토콜로는 TCP와 UDP가 있다.

#### TCP, UDP
|          | TCP                                           | UDP                                        |
|----------|-----------------------------------------------|--------------------------------------------|
| 의미       | 전송 제어 프로토콜<br>(Transmission Control Protocol) | 사용자 데이터그램 프로토콜<br>(User Datagram Protocol) |
| 연결 방식    | 연결형 서비스                                       | 비연결형 서비스                                   |
| 패킷 교환 방식 | 가상 회선 방식                                      | 데이터그램 방식                                   |
| 데이터 단위   | 세그먼트                                          | 데이터그램                                      |
| 전송 순서    | 전송 순서 보장                                      | 전송 순서가 바뀔 수 있음                             |
| 수신 여부 확인 | 수신 여부 확인                                      | 수신 여부 확인하지 않음                              |
| 장점       | 신뢰성 보장                                        | 빠르다                                        |
| 활용       | 대부분의 애플리케이션                                   | 게임, 스트리밍 등 실시간 속도가 중요한 애플리케이션              |

#### 포트 번호(Port Number)
- 컴퓨터는 2byte 길이의 포트 번호를 가진다.
- 잘 알려진 포트(well-known port)인 0번~1023번 포트는 OS와 주요 프로토콜에서 사용하기 때문에, 사용자 애플리케이션은 그 외의 포트 번호를 사용한다.

### L3 | Network
> host-to-host delivery
- 라우팅(Routing)과 포워딩(Forwarding)을 수행해서 목적지 IP 주소까지 패킷을 전달한다.
- OS 커널에 구현되어 있다.
- URL이 주어지면 DNS(Domain Name Resolution)를 통해 IP 주소를 찾고, 실제 패킷은 IP 주소를 향해 전송된다.
- 패킷이 호스트에 도작하면 IP 주소의 광역대에 따라 라우팅 테이블(Routing Table)에 지정된 경로로 패킷을 포워딩한다.

#### IP Address(Internet Protocol Address)
- Host의 **논리적 주소**로, 전 세계의 네트워크상에서 유일하다.
- IPv4는 4byte, IPv6는 8byte 주소를 갖는다.

### L2 | Data-Link
> 1-hop delivery
- 라우팅과 포워딩을 수행해서 목적지 MAC 주소까지 프레임을 전달한다.
- 인접 노드들 간의 신뢰할 수 있는 전달이다.
- Ethernet Card에 구현되어 있다.

#### 홉(Hop)
- 홉은 컴퓨터 네트워크에서 출발지와 목적지 사이에 위치한 경로의 한 부분이다.
- 1 hop은 네트워크에서 한 번의 이동을 의미한다. 예를 들어 한 라우터에서 다음 라우터로 데이터를 전송할 때 그 경로가 1 hop이다.

#### MAC Address(Media Access Control Address)
- Ethernet Card의 **물리적 주소**로 로컬 네트워크 안에서만 유일하다.
- 48 bit 주소를 갖는다.
- Gateway(라우터)는 Ethernet Card를 2개 가지고 있어서 LAN과 WAN을 연결한다.

#### ARP(Address Resolution Protocol)
- IP 주소를 MAC 주소와 매칭시키기 위한 프로토콜이다.
- 호스트는 ARP를 사용하여 IP 주소에 대응하는 MAC 주소를 조회하고, 조회한 정보를 ARP Table에 저장하여 이후 통신이 이루어질 때 참고한다.

### L1 | Physical
- Encoding: 0과 1의 나열을 아날로그 신호로 변환해서 전송한다.
- Decoding: 아날로그 신호를 받으면 0과 1로 해석한다.
- 물리적, 기계적, 전기적 기능으로 하드웨어에 구현되어 있다.

---
**Reference**<br>
- https://velog.io/@jwkim/cs-nw-osi-tcp-ip
- https://velog.io/@kimmainsain/%EB%AA%A8%EB%91%90%EC%9D%98-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-2
- https://itchallenger.tistory.com/215
