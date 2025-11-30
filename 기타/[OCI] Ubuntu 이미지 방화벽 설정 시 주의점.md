# [OCI] Ubuntu 이미지 방화벽 설정 시 주의점
오라클 클라우드 Ubuntu 이미지의 방화벽 규칙을 편집할 때 UFW를 사용하여 규칙을 편집하면 인스턴스가 부팅되지 않을 수 있다.
따라서, **UFW는 기본적으로 비활성화 되어있으며, 방화벽 설정 시 iptables를 사용해야 한다.**

아래는 INPUT 체인에 HTTP(80)와 HTTPS(443) 포트를 허용하는 규칙을 추가하는 예시이다.

## 방법 1 - iptables 명령어를 사용하여 규칙 수정
```bash
# 적용된 규칙 목록을 확인한다.
sudo iptables -L --line-numbers

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
2    ACCEPT     icmp --  anywhere             anywhere
3    ACCEPT     all  --  anywhere             anywhere
4    ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
5    REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited
...
```

새 규칙은 반드시 REJECT 규칙보다 앞에 삽입해야 한다.

> **iptables 명령어 구조**: `iptables -[옵션] [체인명] -m state --state [연결상태] -p [프로토콜] --dport [목적지포트] -j [정책]`

```bash
# INPUT 체인의 5번째 위치에 규칙을 삽입한다. 프로토콜 tcp, 포트 80, NEW 상태의 패킷을 허용한다.
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 80 -j ACCEPT

# INPUT 체인의 5번째 위치에 규칙을 삽입한다. 프로토콜 tcp, 포트 443, NEW 상태의 패킷을 허용한다.
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 443 -j ACCEPT
```

재부팅 시 iptables 규칙이 초기화된다.
netfilter-persistent를 사용하여 변경된 규칙을 저장하고 부팅 시 읽어오게 한다.
```bash
# 재부팅해도 유지되도록 저장
sudo netfilter-persistent save
```

## 방법 2 - /etc/iptables/rules.v4 파일 수정하기
```bash
# 에디터를 사용하여 파일을 수정한다.
sudo nano /etc/iptables/rules.v4
```

다음과 같이 내용을 추가한다.
```bash
...
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT # 추가
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT # 추가
-A INPUT -j REJECT --reject-with icmp-host-prohibited
...
```

이렇게 변경한 방화벽 규칙은 재부팅 후 적용된다. 규칙을 즉시 적용하려면 다음 명령어를 실행한다.
```bash
sudo su -
iptables-restore < /etc/iptables/rules.v4
```

## 적용 후 확인
규칙 추가 후 적용된 내용을 확인해보자.

```bash
sudo iptables -L --line-numbers

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
2    ACCEPT     icmp --  anywhere             anywhere
3    ACCEPT     all  --  anywhere             anywhere
4    ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
5    ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:https
6    ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:http
7    REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited
...
```

---
**Reference**
- https://blogs.oracle.com/developers/enabling-network-traffic-to-ubuntu-images-in-oracle-cloud-infrastructure
- https://docs.oracle.com/en-us/iaas/Content/Compute/References/bestpracticescompute.htm#Essentia
