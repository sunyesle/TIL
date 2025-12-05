# SSH Forwarding
일반적으로 어떤 서비스를 제공할 때 보안을 위하여 배포 DB는 사설망에서 돌아가야 한다.

OCI 인스턴스를 만든다고 했을 때, VCN(VPC)에는 퍼블릭 서브넷과 프라이빗 서브넷이 존재하고, 퍼블릭 서브넷은 internet gateway와 연결되어 있다.

여기서 퍼블릭 서브넷에 연결되어 있는 웹 서버는 외부망에서 22번 포트(SSH)로 접근할 수 있다.

프라이빗 서브넷에 연결되어 있는 DB 서버는 22번 포트는 열려있지만, 라우팅 테이블이 internet gateway에 연결되어 있지 않기 때문에,
외부에서 접근이 불가능하고, 사설망에 존재하는 서버(동일한 VCN 내에 존재하는 서버)에서만 접근할 수 있다.

즉, DB 서버에 접속하려면 다음과 같이 접속해야 한다.
```
로컬 PC --(public IP로 SSH 접속)--> 웹 서버 --(private IP로 SSH 접속)--> DB 서버
```

그렇다고 외부에서 접속 가능한 웹 서버에 DB 서버 키를 복사해 둔다면, 보안을 위해 DB 서버의 망 분리를 해놓은 이유가 없어진다.
웹 서버가 뚫리면 자동으로 DB서버도 뚫리기 때문이다.

이럴 때 **SSH Forwarding**을 사용하면, 웹 서버에 개인키를 복사하지 않고도 로컬 PC에 있는 SSH 개인키를 사용해 DB 서버에 인증할 수 있다.
웹 서버는 단순히 인증 요청을 전달만 하고, 실제 개인키는 항상 로컬 PC에만 존재하므로 보안성을 유지한 채로 프라이빗 서브넷의 DB 서버에 접근할 수 있다.

## SSH Forwarding 작동 방식
1. 로컬 PC에서 SSH 에이전트 활성화
   - 로컬에서 SSH 키를 `ssh-agent`에 등록하고, `ForwardAgent` 옵션을 통해 인증 정보 전달을 허용한다.
2. 중간 서버(Bastion)로 인증 정보 전달
   - `ForwardAgent`가 활성화된 경우, 중간 서버로 인증 정보를 전달한다.
3. 최종 서버로 연결
   - 중간 서버에서 전달받은 인증 정보를 사용해 최종 서버에 연결한다.

## ssh-agent 기본 명령어
```bash
# ssh-agent 활성화
eval $(ssh-agent)

# ssh-agent에 키 등록
ssh-add ~/.ssh/id_rsa

# 등록된 키 확인
ssh-add -l

# 등록된 키 전체 삭제
ssh-add -D
```

## 예시
`로컬 PC → 웹 서버 → DB 서버`로 접속하는 예시이다.

### 1. 명령어만을 사용한 방식
```bash
eval $(ssh-agent)
ssh-add ~/.ssh/ssh-key-web.key
ssh-add ~/.ssh/ssh-key-db.key

ssh -A -J ubuntu@<웹 서버 public IP> ubuntu@<DB 서버 private IP>
```
- `-A`: SSH Agent Forwarding 활성화
- `-J ubuntu@<웹 서버 public IP>`: ProxyJump 옵션. 웹 서버를 중간 서버로 사용

### 2. SSH Config 파일을 활용한 방식
`~/.ssh/config` 파일을 사용하면 매번 긴 명령어를 입력할 필요 없이 간편하게 서버에 접속할 수 있다.

```
Host web
  HostName <웹 서버 public IP>
  User ubuntu
  IdentityFile ~/.ssh/ssh-key-web.key
  ForwardAgent yes

Host db
  HostName <DB 서버 private IP>
  User ubuntu
  IdentityFile ~/.ssh/ssh-key-db.key
  ProxyJump web
```
위와 같이 설정 파일을 작성하면 다음 명령어만으로 DB 서버에 접속할 수 있다.
```
ssh db
```

---
**Reference**
- https://rupijun.tistory.com/entry/SSH-Forwarding과-Config-설정을-통한-효율적-사용
- https://isaac56.github.io/linux/2021/05/04/SSH-Agent_Forwarding/
