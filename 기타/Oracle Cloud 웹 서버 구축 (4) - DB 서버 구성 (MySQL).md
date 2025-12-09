# Oracle Cloud 웹 서버 구축 (4) - DB 서버 구성 (MySQL)

## Ubuntu 초기 설정
패키지 업데이트, 시스템 시간 설정, 시스템 언어 설정, Swap 메모리 설정을 진행한다.
> **[Ubuntu 초기 설정]** 글 참고

## 방화벽 설정
MySQL(3306번 포트) 접근을 허용하기 위해 규칙을 추가한다.

OCI의 Ubuntu 이미지 방화벽 설정 시에는 ufw를 사용해서는 안되며, iptables를 사용해야 한다.
```bash
# 적용된 규칙 목록을 확인한다.
sudo iptables -L INPUT --line-numbers

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
2    ACCEPT     icmp --  anywhere             anywhere
3    ACCEPT     all  --  anywhere             anywhere
4    ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
5    REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited
```

REJECT 규칙(5번) 앞에 방화벽 규칙을 추가한다.
```
sudo iptables -I INPUT 5 -m state --state NEW -p tcp -s <웹 서버 private IP> --dport 3306 -j ACCEPT
```

## 오프라인 환경에서 MySQL 패키지 설치
DB 서버는 인터넷에 연결되어있지 않기 때문에, 웹 서버에서 패키지를 다운로드 받고 DB 서버로 옮겨서 설치를 진행해준다.

우선 DB 서버에 패키지를 받을 폴더를 만든다.
```bash
# pkg 폴더를 만든다.
mkdir ~/pkg
```

웹 서버로 이동해서 패키지를 다운로드 받는다.
```bash
# --download-only 옵션으로 설치는 하지 않고 다운로드만 받는다.
# /var/cache/apt/archives 폴더에 .deb 파일이 다운로드된다. 
sudo apt reinstall --download-only -y mysql-server

# pkg 폴더를 만든다.
mkdir ~/pkg/

# 다운로드 받은 .deb 파일을 ~/pkg 폴더에 복사한다.
sudo cp /var/cache/apt/archives/*.deb ~/pkg/

# /var/cache/apt/archives 폴더의 .deb 파일들을 삭제한다.
sudo apt clean
```

로컬에서 `scp` 명령어로 웹서버 `~/pkg` 폴더의 .deb 파일을 DB 서버의 `~/pkg` 폴더로 전송한다.
```bash
sudo cp /var/cache/apt/archives/*.deb ~/pkg/
```

DB 서버에서 패키지를 다운로드한다.
```bash
# pkg 폴더에 있는 모든 .deb 패키지를 설치한다.
sudo apt install ~/pkg/*.deb

# 정상적으로 설치되었는지 확인한다.
mysql --version

# ~/pkg 폴더의 .deb 파일들을 삭제한다.
rm ~/pkg/*.deb
```

## MySQL 설정
### 설정 파일 수정
```bash
# 설정 파일을 수정한다.
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

`mysqld.cnf` 파일의 변경 사항은 다음 두 가지이다.
- **외부 접속 설정**: `bind-address = 0.0.0.0` 으로 수정한다.
- **타임존 설정**(**UTC**): `default-time-zone = '+00:00'` 을 추가한다.

```bash
# 설정 변경사항 적용을 위해 mysql을 재시작한다.
sudo systemctl restart mysql
```

### 데이터베이스 생성, 계정 생성, 권한 설정
```sql
# MySQL에 root 계정으로 접속한다.
sudo mysql -u root

# 애플리케이션에서 사용할 mydb 데이터베이스를 생성한다.
CREATE DATABASE mydb;

# 웹 서버에서 접속 가능한 testuser 계정을 생성한다.
CREATE USER 'testuser'@'<웹 서버 private IP>' IDENTIFIED BY 'password1111';

# testuser 계정에게 mydb 데이터베이스에 대한 모든 권한을 부여한다.
GRANT ALL PRIVILEGES ON mydb.* TO 'testuser'@'<웹 서버 private IP>';

# 권한 설정을 반영한다.
FLUSH PRIVILEGES;
```

설정한 내용이 반영되었는지 확인한다.
```sql
# 타임존 확인
SELECT @@system_time_zone, @@global.time_zone, @@session.time_zone;

# 사용자 확인
SELECT user, host FROM mysql.user;

# 권한 확인
SHOW GRANTS FOR 'testuser'@'<웹 서버 private IP>';
```

## 로컬 환경에서 DB 접속하기
DBeaver, MySql Workbench 등 SSH 터널링을 지원하는 툴을 이용해서 DB 서버에 접속할 수 있다.
