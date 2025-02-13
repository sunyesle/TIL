# HTTPS(HyperText Transfer Protocol Secure)

## HTTPS란
- HTTPS는 **HTTP의 보안이 강화된 버전**이다.
- HTTP를 기반으로 웹 서버와 통신하되 **암호화 통신**을 위한 별도의 협의 과정을 거치게 된다. 이때 **SSL**과 **TLS**라는 프로토콜을 사용하여 데이터를 암호화한다.
- HTTPS의 기본 TCP/IP 포트는 **443**이다.
- HTTPS를 사용하는 웹페이지의 URI는 `https://` 로 시작한다.

### 통신 과정
![Image](https://github.com/user-attachments/assets/2ac476d5-0584-4255-b8e1-59db6715bf5c)

## SSL 인증서
인증 기관(CA)에서 발급하는 인증서이다.
인증서는 클라이언트가 접속한 서버가 신뢰할 수 있는 서버인지 보장하는 역할을 한다.

## Key
HTTPS에서 암호화와 복호화 작업을 위해서는 키가 필요하다.

### 대칭키
암호화/복호화에 동일한 하나의 키를 사용한다.
- **장점**: 암호화와 복호화 속도가 빠르다.
- **단점**: 키 관리의 어려움. 동일한 키를 사용하기 키를 안전하게 관리하고 전달해야 한다.

### 비대칭키
암호화/복호화에 각각 다른 키를 사용한다. 공개키로 암호화한 데이터는 개인키로 복호화할 수 있다.
- **장점**: 키 관리의 용이성. 공개키는 안전하게 공유될 수 있으며, 개인키는 소유자만 관리하면 된다.
- **단점**: 암호화와 복호화 속도가 빠르다.

**공개키**는 공개 가능한 키이다. 개인키와 쌍을 이루어 존재한다.
**개인키**는 사적인 키로 오로지 key 발행자만이 갖는 키이다. 공개키와 쌍을 이루어 존재한다.

> 공개키로 암호화 → 개인키로 복호화 ⇒ **데이터 보안**<br>
> 개인키로 암호화 → 공개키로 복호화 ⇒ **전자 서명**

## HTTP 통신 과정
![Image](https://github.com/user-attachments/assets/71e0824e-a8bc-49e0-b9ef-dead36740402)

위 그림과 함께 클라이언트와 서버가 어떻게 SSL 인증서를 검증하고, 데이터를 암호화/복호화하는지 살펴보자.

- **[1] 서버**: 한 쌍의 서버의 공개키와 개인키를 생성한다. 그리고 서버 내 사이트의 각종 정보와 자신(서버)의 공개키를 인증기관에 전달하여 SSL 인증서 생성을 요청한다.
- **[2] 인증기관**: SSL 인증서를 발급한다. 이 인증서에는 서버의 도메인을 비롯하여 서버임을 증명하는 각종 정보를 담고 있다. 이때 인증기관은 자신만의 한 쌍의 공개키와 개인키를 생성하고, SSL 인증서를 인증기관의 개인키로 암호화하여 서버에게 전달하고, 서버는 이 SSL 인증서를 게시한다.
- **[3] 웹브라우저**: 인증기관의 개인키로 암호화된 SSL 인증서를 서버로부터 전달받았다. SSL 인증서가 진짜인지 검증하려고 한다.
- **[4~5] 웹브라우저**: 인증기관에서 공개하고 있는 인증기관의 공개키를 가져와 이 SSL 인증서를 복호화해 본다. 만약 복호화되어 서버의 정보를 확인할 수 있다면, SSL 인증서는 진짜이다. 인증서를 보유한 서버가 진짜임을 밝힘과 동시에 발급한 대상이 인증기관임을 알 수 있다. 그리고 웹 브라우저는 서버의 공개키를 확보했다.
- **[6] 웹브라우저**: 웹브라우저가 서버를 신뢰하고 있다. 방금 전달받은 서버의 공개키로 실제 데이터 암호화에 사용할 비밀키(대칭키)를 암호화하여 서버에게 전송한다.
- **[7] 서버**: 서버는 웹브라우저가 자신(서버)의 공개키로 암호화하여 전달한 비밀키(대칭키)를 자신의 개인키로 복호화한다. 이로써 웹브라우저와 서버는 동일한 비밀키(대칭키)를 보유하게 된다.

## SSL Handshake(TLS Handshake)
송신자와 수신자가 서로 주고받을 데이터의 안전한 암호화를 위한 **암호화 알고리즘**(Cipher Suite)을 결정하고, 서로의 **대칭키**를 얻기 위한 일련의 협상 과정을 의미한다.

> **Cipher Suite**<br>
> 암호화 협상에 필요한 알고리즘의 집합이다.
>
> ![Cipher Suite](https://github.com/user-attachments/assets/98379dcf-dbe3-469c-a826-4e4f74d1d6b6)

### SSL Handshake 과정
![SSL Handshake](https://github.com/user-attachments/assets/ce21793d-4795-463e-9ff4-0b301ceb26b7)

**파란색**은 TCP layer의 **3-way handshake**로 HTTPS가 TCP기반의 프로토콜이기 때문에 암호화 협상(SSL handshake)에 앞서 연결을 생성하기 위해 실시하는 과정이고, **노란색**이 **SSL handshake**이다.

SSL handshake는 다음과 같은 순서로 진행된다.
- `Client`: 암호화 알고리즘 나열 및 전달
- `Server Hello`: 암호화 알고리즘 선택
- `Certificate`: 인증서 전달
- `Client Key Exchange`: 데이터를 암호화할 대칭키 전달
- `Change Cipher Spec`: 정보 전달 완료
- `Finished`: SSL handshake 종료

각 패킷들에 대해 자세히 알아보자.

### Client Hello
Client가 Server에 연결을 시도하며 전송하는 패킷이다. 자신이 사용 가능한 Cipher Suite 목록, Session ID, SSL Protocol Version, Random byte 등을 전달한다.

<img alt="client hello" width="600" src="https://github.com/user-attachments/assets/1ade4529-1a0b-43f8-ae54-667fc161a96f"/>

### Server Hello
Client가 보내온 Cipher Suite 중 하나를 선택한 다음 Client에게 이를 알린다. 또한 자신의 SSL Protocol Version 등도 같이 보낸다.

<img alt="server hello" width="600" src="https://github.com/user-attachments/assets/8c3f707b-c40b-4d8b-a71a-fc63249353f6"/>

### Certificate
Server가 자신의 SSL 인증서를 Client에게 전달한다.
 
Client는 Server가 보낸 CA의 개인키로 암호화된 SSL 인증서를 CA의 공개키를 사용하여 복호화한다.
복호화에 성공하면 CA에서 서명한 것이 맞는 것이기 때문에 진짜임이 증명된다.

<img alt="certificate" width="800" src="https://github.com/user-attachments/assets/c969c529-a4b3-46a7-8d83-a4134a874883"/>

### Server Key Exchange / Server Hello Done
Server Key Exchange는 Server의 공개키가 SSL 인증서 내부에 없는 경우, Server가 직접 전달함을 의미한다. 공개키가 SSL 인증서 내부에 있을 경우 생략된다.

그리고 Server Hello Done 패킷을 보내어 서버가 행동을 마쳤음을 전달한다.

<img alt="server key exchange" width="600" src="https://github.com/user-attachments/assets/357e784f-8db3-46e8-b1ed-a6f7bd210d4e"/>

### Client Key Exchange
Client는 대칭키를 생성한 후 서버의 공개키를 이용해 암호화하여 서버에게 전송한다.

여기서 전달된 대칭키가 SSL handshake의 목적이자 가장 중요한 수단인 데이터를 실제로 암호화하는 데 사용되는 대칭키(비밀키)이다.

<img alt="client key exchange" width="600" src="https://github.com/user-attachments/assets/41e9c5dd-c6f5-4fbd-ad78-c19653504dce"/>

### Change Cipher Spec / Finished
Client, Server 모두가 서로에게 보내는 패킷으로 교환할 정보를 모두 교환한 뒤 통신할 준비가 되었음을 알리는 패킷이다.

그리고 Finished 패킷을 보내어 SSL handshake를 종료하게 된다.

<img alt="change cipher spec" width="600" src="https://github.com/user-attachments/assets/bb3689af-e9b6-41b0-859c-2aef63a62458"/>

---
**Reference**<br>
- https://aws-hyoh.tistory.com/38
