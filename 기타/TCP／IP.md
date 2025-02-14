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

---
**Reference**<br>
- https://aws-hyoh.tistory.com/57
- https://better-together.tistory.com/110
