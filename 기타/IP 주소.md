# IP 주소

## IPv4 주소 체계
<img alt="IPv4" width="600" src="https://github.com/user-attachments/assets/a6d08c87-e4ac-436e-8316-cd7d7ea481e8"/>

IP 주소는 **32비트**로 구성되어 있다.
**8비트**씩 묶어 10진수로 표기하며(0~255) 각각의 수를 **옥텟**(**Octet**)이라고 한다. 옥텟은 점으로 구분한다.

IP 주소는 네트워크 ID와 호스트 ID로 구분된다.
- **Network ID** : 네트워크를 식별하는 부분이다. 같은 네트워크에 속한 모든 장치는 동일한 네트워크 ID를 갖는다.
- **Host ID** : 하나의 네트워크 내에 존재하는 호스트를 구분하기 위한 부분이다.

## IP 주소 클래스
필요한 호스트 IP 개수에 따라 네트워크 크기를 조절할 수 있도록 **클래스(class)** 개념을 도입했다.<br>
클래스에는 A, B, C, D, E 다섯 가지 종류가 있다.

<img alt="IP class" src="https://github.com/user-attachments/assets/bace9c05-596f-4ef7-85b6-084902537bad"/>
<br>
<br>

| 클래스   | 상위 옥텟    | 주소 영역 구분                      | 주소 범위                       | 호스트 개수     |
|-------|----------|-------------------------------|-----------------------------|------------|
| **A** | 0xxxxxxx | `네트워크`.`호스트`.`호스트`.`호스트` | 0.0.0.0 ~ 127.255.255.255   | 약 1,600만 개 |
| **B** | 10xxxxxx | `네트워크`.`네트워크`.`호스트`.`호스트` | 128.0.0.0 ~ 191.255.255.255 | 약 6만 5천 개  |
| **C** | 110xxxxx | `네트워크`.`네트워크`.`네트워크`.`호스트` | 192.0.0.0 ~ 223.255.255.255 | 약 250개     |

---
**Reference**<br>
- https://brunch.co.kr/@swimjiy/44
- https://velog.io/@sangmin1998/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%84%9C%EB%B8%8C%EB%84%B7Subnet-%EA%B0%9C%EB%85%90
- https://velog.io/@octo__/IPv4%EC%99%80-Subnet-Mask
- https://velog.io/@satoshi25/IP-%EC%A3%BC%EC%86%8C
