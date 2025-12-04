# Oracle Cloud 웹 서버 구축 (2) - SSH 원격 접속

> 글에서는 사용되는 IP 주소 값은 예시이며, 실제 환경에 맞게 치환해서 사용해야 한다.
> - 웹 서버 퍼블릭 IP: `203.0.113.10`
> - 웹 서버 프라이빗 IP: `10.0.0.1`
> - DB 서버 프라이빗 IP: `10.0.0.2`

생성한 서버에 원격 접속하기 위한 SSH 연결 설정을 진행할 것이다.

인스턴스 생성 시에 다운받은 SSH 프라이빗 키들을 `C:\Users\<사용자명>\.ssh` 폴더로 옮긴다.

웹 서버 키는 `ssh-key-web.key`, DB 서버 키는 `ssh-key-db.key`로 이름을 변경하였다.

## 파일 권한 설정
프라이빗 키의 권한이 너무 오픈되어 있을 경우 SSH 접속 시 `Permission denied` 오류가 발생한다.
```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions for 'C:\\Users\\Username\\.ssh\\ssh-key-web.key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "C:\\Users\\Username\\.ssh\\ssh-key-web.key": bad permissions
ubuntu@203.0.113.10: Permission denied (publickey).
```
다음 방법대로 `ssh-key-web.key`와 `ssh-key-db.key`의 권한 설정을 진행한다.

### Linux
Linux에서는 `chmod` 명령어를 통해 파일 권한을 `600`으로 수정하면 된다.
### Windows
Windows에서는 다음과 같이 파일 권한을 설정할 수 있다.
1. 파일 선택 > 마우스 오른쪽 클릭 > [속성] 클릭
2. [보안] 탭 > [고급] 클릭
3. [상속 사용 안함] 클릭 > "이 개체에서 상속된 사용 권한을 모두 제거합니다." 선택
4. [추가] 클릭 > "보안 주체 선택" 클릭 > 선택할 개체 이름에 사용자명 입력 후 [이름 확인] 클릭 > [확인] 클릭
5. "읽기 및 실행", "읽기" 권한 체크 > [확인] 클릭
6. 소유자에 대한 단일 권한 확인 후, [적용] > [확인] 클릭
7. 최종적으로 소유자에게만 권한이 부여된 것을 확인 후, [확인] 클릭

<img width="421" height="580" alt="권한 설정 이미지" src="https://github.com/user-attachments/assets/ee4849f2-bed2-41e8-b95c-3c20d9876b2c" />

## SSH Config 작성
SSH 접속 명령어를 간소화 할 수 있도록 설정 파일을 작성할 것이다.

`C:\Users\<사용자명>\.ssh` 폴더에 `config` 파일을 생성한다.

텍스트 에디터로 `config` 파일을 열어서 다음 내용을 입력한다.
```
Host web
  HostName 203.0.113.10
  User ubuntu
  IdentityFile ~/.ssh/ssh-key-web.key
  ForwardAgent yes

Host db
  HostName 10.0.0.2
  User ubuntu
  IdentityFile ~/.ssh/ssh-key-db.key
  ProxyJump web
```
- **HostName**: IP 또는 도메인
- **User**: 서버에 접속할 사용자명
- **IdentityFile**: 키 파일 경로
- **ForwardAgent**: SSH Agent forwarding 활성화 여부
- **ProxyJump**: 중간 서버를 통해 최종 서버에 연결

설정을 완료하면 `ssh web`, `ssh db` 명령어로 각 서버에 원격 접속을 할 수 있다.
