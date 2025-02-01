# OSI 모델과 TCP/IP 모델

![Image](https://github.com/user-attachments/assets/5fa2e69a-47ca-4051-bb57-f342c80e3b08)

## OSI 7계층 모델
**OSI 7계층 모델**은 국제표준화기구(ISO)에서 개발한 네트워크 모델이다.<br>
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

---
**Reference**<br>
- https://velog.io/@jwkim/cs-nw-osi-tcp-ip
- https://velog.io/@kimmainsain/%EB%AA%A8%EB%91%90%EC%9D%98-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-2
- https://itchallenger.tistory.com/215
