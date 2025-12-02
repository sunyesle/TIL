# SSH 원격 접속

## SSH란
**SSH**(**Secure Shell**)는 원격 서버에 접속하기 위해 사용되는 보안 프로토콜이다.

기존 원격 접속에는 Telnet 프로토콜을 사용했는데, 데이터를 평문으로 전송하기 때문에 패킷 캡처 등에 취약했다.
SSH는 Telnet의 보안 문제점을 개선한 프로토콜로, 암호화된 안전한 통신을 제공한다.

기본 포트는 22번이다.

## SSH 명령어
```bash
# 사용자명 + IP로 접속
ssh <Username>@<IP>

# 사용자명 + IP + 포트번호로 접속
ssh <Username>@<IP> -p <Port>

# 사용자명 + IP + 포트번호 + 키파일로 접속
ssh -i "<Path to Private Key>" <Username>@<IP> -p <Port>
```

## SSH Config
SSH Config 파일에 Host를 설정하면 접속할 때마다 명령어를 일일히 작성할 필요없이 `ssh <Host Alias>`로 접속할 수 있다.

SSH Config 파일은 `~/.ssh/config`에 위치한다.
윈도우 기준으로 `C:\Users\<사용자명>\.ssh` 경로에 확장명 없이 `config` 파일을 생성하면 된다.

### 작성 방법
```
Host <Alias>
    HostName <Server Address>
    User <Username>
    IdentityFile <Path to Private Key>
    Port <Port Number>
```

> 예시
```
Host my-server
    HostName 192.168.1.100
    User ubuntu
    IdentityFile ~/.ssh/id_rsa
    Port 22
```
위와 같이 설정을 작성하면 `ssh my-server` 명령어만으로 `192.168.1.100` 서버 `ubuntu` 사용자로 연결할 수 있다.

## SSH Config 주요 설정 값
### 1. Host
설정 블록의 이름으로, ssh 명령어에서 사용할 별칭
<br>(예시: `Host my-server`)

### 2. HostName
실제 서버의 도메인 이름 또는 IP 주소
<br>(예시: `HostName example.com`, `HostName 192.168.1.100`)

### 3. User
서버에 접속할 사용자 이름
<br>(예시: `User ubuntu`)

### 4. IdentityFile
인증에 사용할 개인 키 파일 경로
<br>(예시: `IdentityFile ~/.ssh/my-key.key`)

### 5. Port
SSH 연결에 사용할 포트 번호 (기본값: 22)
<br>(예시: `Port 2022`)

### 6. ForwardAgent
SSH 에이전트 포워딩 활성화 여부 (`yes`/`no`)
<br>(예시: `ForwardAgent yes`)

### 7. ProxyJump
중간 서버를 통해 최종 서버에 연결
```
Host a-server
    HostName 192.168.1.200
    User ubuntu

Host b-server
    HostName 192.168.1.250
    User ubuntu
    ProxyJump a-server # a-server로 접속 후 b-server로 점프
```

### 8. ServerAliveInterval
서버와의 연결이 유휴 상태일 때, KeepAlive 패킷을 보낼 간격 (초 단위)
<br>(예시: `ServerAliveInterval 60`)

### 9. ServerAliveCountMax
KeepAlive 패킷 전송 실패시 연결을 종료하기 전까지의 시도 횟수
<br>(예시: `ServerAliveCountMax 3`)

## 설정 예시
### 웹 서버를 경유한 DB 서버 접속 설정
```
Host web
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/ssh-key-web.key
    ForwardAgent yes

Host db
  HostName 10.0.0.10
  User ubuntu
  IdentityFile ~/.ssh/ssh-key-db.key
  ProxyJump web
```

### 특정 도메인에 공통 설정
```
Host *.example.com
    User shared-user
    IdentityFile ~/.ssh/shared_key
    Port 22
```

---
**Reference**
- https://rupijun.tistory.com/entry/SSH-Config-파일sshconfig의-사용법과-설정값
- https://priming.tistory.com/72
