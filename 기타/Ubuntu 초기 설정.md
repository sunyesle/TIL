# Ubuntu 초기 설정
예시에서 사용한 버전은 `Ubuntu 24.04.3 LTS` 이다.

## 패키지 업데이트
```bash
# 설치 되어있는 패키지에 대해서 최신 버전이 있는지 확인한다.
sudo apt update

# update를 통해서 확인한 패키지들의 버전을 업그레이드한다.
sudo apt upgrade -y
```

## 시스템 시간 설정
```bash
# 현재 날짜 및 시간을 확인한다.
date

# 현재 서버의 시간대를 확인한다.
timedatectl

# 시간대 목록을 확인한다.
timedatectl list-timezones | grep Seoul

# 시간대를 변경한다.
sudo timedatectl set-timezone Asia/Seoul
```

## 시스템 언어 설정
```bash
# 현재 Locale을 확인한다.
locale

# Locale 목록을 확인한다.
locale -a

# 생성 가능한 Locale을 확인한다.
cat /usr/share/i18n/SUPPORTED | grep ko_KR

# Locale을 생성한다.
sudo locale-gen ko_KR.UTF-8

# Locale을 변경한다.
sudo update-locale LANG=ko_KR.UTF-8
```
변경 사항은 세션 재접속 후 확인할 수 있다.

## 방화벽 설정
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

새 규칙이 REJECT 규칙보다 앞에 올 수 있도록 5번 위치에 규칙을 삽입한다.
- **iptables 명령어 구조**: `iptables -[옵션] [체인명] -m state --state [연결상태] -p [프로토콜] --dport [목적지포트] -j [정책]`
```bash
# INPUT 체인의 5번째 위치에 규칙을 삽입한다. 프로토콜 tcp, 포트 80, NEW 상태의 패킷을 허용한다.
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 80 -j ACCEPT

# INPUT 체인의 5번째 위치에 규칙을 삽입한다. 프로토콜 tcp, 포트 443, NEW 상태의 패킷을 허용한다.
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 443 -j ACCEPT
```

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

재부팅 시 iptables 규칙이 초기화된다. `netfilter-persistent`를 이용하여 변경된 규칙을 저장하고 부팅 시 읽어오게 한다. 
```bash
# 재부팅해도 유지되도록 저장
sudo netfilter-persistent save
```

## Swap 메모리 설정
Oracle Cloud 무료 인스턴스의 가용 메모리는 1GB이다.
메모리 부족으로 인한 문제를 방지하기 위해 스왑 메모리를 설정해 주자.
```bash
# 메모리를 확인한다.
free -m
               total        used        free      shared  buff/cache   available
Mem:             956         397         196           5         539         559
Swap:              0           0           0

# 스왑 영역을 확인한다. (스왑 영역이 없으므로 아무것도 출력되지 않는다.)
swapon -s
```
다음과 같이 스왑 메모리를 설정할 수 있다.
```bash
# 더미 파일을 생성한다.
sudo fallocate -l 4GB /swapfile

# 권한을 조정한다.
sudo chmod 600 /swapfile

# 스왑 파일을 스왑 영역으로 지정한다.
sudo mkswap /swapfile

# 스왑 파일을 활성화 시킨다.
sudo swapon /swapfile
```

설정 후 다시 메모리와 스왑 영역을 확인해 보자.
```bash
free -m
               total        used        free      shared  buff/cache   available
Mem:             956         379         374           4         355         577
Swap:           3814           0        3814

swapon -s
Filename                                Type            Size            Used            Priority
/swapfile                               file            3906244         0               -2
```

재부팅 시 스왑 메모리 설정이 초기화된다. `/etc/fstab` 파일에 스왑 항목을 등록하면, 부팅할 때 자동으로 스왑이 활성화된다.

> `/etc/fstab`: 리눅스 부팅 시 마운트 정보를 저장하고 있는 파일

```bash
# nano 에디터로 파일을 수정한다.
sudo nano /etc/fstab

# 파일의 최하단에 다음 내용을 추가한다. 이후 Ctrl+O(저장), Ctrl+X(나가기)
/swapfile       none    swap    sw      0       0

# 파일 내용을 확인한다.
cat /etc/fstab

LABEL=cloudimg-rootfs   /        ext4   discard,commit=30,errors=remount-ro     0 1
LABEL=BOOT      /boot   ext4    defaults        0 2
LABEL=UEFI      /boot/efi       vfat    umask=0077      0 1
/swapfile       none    swap    sw      0       0
```
---
**Reference**
- Locale 설정: https://www.44bits.io/ko/post/setup_linux_locale_on_ubuntu_and_debian_container
