# 네트워크란
## 컴퓨터 네트워크
**컴퓨터 네트워크**란 2개 이상의 컴퓨터가 데이터를 주고받을 수 있도록 연결된 구조를 말한다.

## 패킷
**패킷**은 컴퓨터 네트워크에서 데이터를 전송하는 **기본 단위**이다.

데이터 통신 중에 정보를 주고받을 때, 큰 데이터를 **작은 조각**으로 나누어 전송하게 되는데, 이 작은 조각이 패킷이다.
목적지에서는 패킷을 재조립해서 정보를 얻는다.

# 네트워크의 종류
네트워크는 범위에 따라 크게 LAN, WAN으로 나눌 수 있다.

## LAN(Local Area Network)
**LAN**(근거리 통신망)은 한정된 지역(집, 사무실, 학교) 내에서 사용되는 네트워크를 말한다.

### 가정에서의 랜 구성
<img alt="1-10 가정에서의 랜 구성 예" width="600" src="https://github.com/user-attachments/assets/6ae124ec-0e89-44e7-9a75-5d50de3dd37a">

<img alt="1-11 가정에서의 네트워크 구성 예" width="600" src="https://github.com/user-attachments/assets/a19b3449-e144-4191-a38a-e7a6d01532fb">

인터넷 공유기를 중심으로 내부 인터넷망(사설망)을 구성하고, 다양한 기기를 연결할 수 있다.

### 회사에서의 랜 구성
<img alt="1-12 회사에서의 랜 구성 예" width="600" src="https://github.com/user-attachments/assets/ccc969f2-ac37-4aef-baae-880f9fe87d46">

<img alt="1-13 회사에서의 네트워크 구성 예" width="600" src="https://github.com/user-attachments/assets/8b35d312-3077-458f-9ec0-f432e88c7e24">

가정에서의 랜 구성과 눈에 띄게 다른 점은 *DMZ*라는 네트워크 영역이 있다는 것이다.

> **DMZ**(**DeMilitarized Zone**)<br>
외부 네트워크와 내부 네트워크 사이에서 외부 네트워크 서비스를 제공하면서 내부 네트워크를 보호하는 서브넷. 외부에 오픈된 서버 영역이다.

회사에서는 서버를 운영하기 위해 서버를 사내에 설치하거나, 데이터 센터에 두거나, 클라우드에 둘 수 있는데,
위 그림은 서버를 사내에 두고 있는 경우이다.

> **온프레미스**(**on-premise**)<br>
사내 또는 데이터 센터 내에 서버를 두고 운영하는 것. 원격 환경에서 서버를 운영하는 클라우드와 대비되는 개념이다.

## WAN(Wide Area Network)
**WAN**(광역 통신망)은 넓은 지역에 걸쳐 여러 LAN을 연결하는 네트워크를 말한다.

대표적인 WAN의 예로는 인터넷, 기업 전용망(VPN) 등이 있다.

## 인터넷
**인터넷**은 TCP/IP 프로토콜을 기반으로 전 세계의 크고 작은 네트워크를 연결하는 세계 최대 규모의 컴퓨터 네트워크이다.

---
**Reference**
- https://ujkim-game.tistory.com/40
- https://velog.io/@kimmainsain/모두의-네트워크-정리-1
- https://itchallenger.tistory.com/212
- https://velog.io/@april_5/네트워크의-구성
- https://engineerinsight.tistory.com/302
