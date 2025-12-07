# Oracle Cloud 웹 서버 구축 (3) - 웹 서버 구성 (Nginx + Spring Boot)

## Ubuntu 초기 설정
패키지 업데이트, 시스템 시간 설정, 시스템 언어 설정, Swap 메모리 설정을 진행한다.
> **[Ubuntu 초기 설정]** 글 참고

## 방화벽 설정
HTTP(80번 포트), HTTPS(443번 포트) 접근을 허용하기 위해 규칙을 추가한다.

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
```bash
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 443 -j ACCEPT
```

## Nginx 설치 및 설정
```bash
# Nginx를 설치한다.
sudo apt install nginx -y

# Nginx 버전을 확인한다.
nginx -v

# 사이트 설정 파일을 생성한다.
sudo nano /etc/nginx/sites-available/app.conf

# ---------------------------
# app.conf 내용 예시
# ---------------------------
server {
    listen 80;
    server_name <웹 서버 public IP>;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# 사이트 설정을 활성화한다. (available → enabled 심볼릭 링크 생성)
sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/

# 설정을 테스트한다.
sudo nginx -t

# Nginx를 재시작한다.
sudo systemctl restart nginx

# Nginx 상태를 확인한다.
sudo service nginx status
```

## Spring Boot 개발 및 빌드
배포용 JAR 파일을 생성해 보자.

Spring Web 의존성을 추가한 후, 간단한 컨트롤러를 작성하여 애플리케이션이 정상적으로 동작하는지 확인한다.
이후 프로젝트를 빌드하여 JAR 파일을 생성한다.
```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping
    public String hello() {
        return "Hello World!";
    }
}
```

## 배포 서버 환경 설정 및 스크립트 작성
다시 웹 서버로 돌아와서 애플리케이션 배포를 위한 설정을 진행한다.

```bash
# 폴더를 만들고 해당 폴더로 이동한다.
mkdir ~/app
cd ~/app

# JAR 파일이 위치할 디렉토리를 만든다.
mkdir current
```

### 애플리케이션 실행 스크립트 작성
```bash
# start.sh 파일을 생성한다.
nano start.sh
```
> start.sh 내용
```bash
#!/bin/bash

APP_NAME="app"
PROFILE="prod"
JAVA_OPTS="-Xms512m -Xmx1g"

APP_DIR="$HOME/app"
JAR_PATH="$APP_DIR/current/$APP_NAME.jar"
LOG_DIR="$APP_DIR/logs"
PID_FILE="$APP_DIR/$APP_NAME.pid"

# 로그 디렉토리 생성
mkdir -p "$LOG_DIR"

# JAR 존재 여부 확인
if [ ! -f "$JAR_PATH" ]; then
  echo "Error: JAR file $JAR_PATH does not exist."
  exit 1
fi

# PID 파일 체크 (이미 구동 중인지 확인)
if [ -f "$PID_FILE" ]; then
    PID=$(cat "$PID_FILE")
    if ps -p "$PID" > /dev/null; then
        echo "Error: $APP_NAME is already running with PID $PID. Stop it first."
        exit 1
    else
        echo "Warning: Stale PID file found. Deleting $PID_FILE."
        rm -f "$PID_FILE"
    fi
fi

# 애플리케이션 시작
echo "Starting $APP_NAME with profile [$PROFILE]..."

nohup java $JAVA_OPTS -jar "$JAR_PATH" \
  --spring.profiles.active="$PROFILE" \
  > "$LOG_DIR/$APP_NAME.out" 2>&1 &
PID=$!

echo $PID > "$PID_FILE"
echo "$APP_NAME started with PID $PID"
```

### 애플리케이션 종료 스크립트 작성

```bash
# start.sh 파일을 생성한다.
nano stop.sh
```
> stop.sh 내용
```bash
#!/bin/bash

APP_NAME="app"
APP_DIR="$HOME/app"
PID_FILE="$APP_DIR/$APP_NAME.pid"
STOP_TIMEOUT=10

if [ ! -f "$PID_FILE" ]; then
  echo "PID file not found. Skipping stop."
  exit 0
fi

PID=$(cat "$PID_FILE")

if ! ps -p "$PID" > /dev/null; then
    echo "$PROC_NAME was not running. Deleting file."
    rm -f "$PID_FILE"
    exit 0
fi

# 종료 시도
echo "Stopping $APP_NAME (PID: $PID)..."
kill "$PID"

# 지정된 시간동안 프로세스 종료를 기다림
COUNT=0
while [ $COUNT -lt $STOP_TIMEOUT ]; do
    if ! ps -p "$PID" > /dev/null; then
        echo "$APP_NAME stopped gracefully."
        rm -f "$PID_FILE"
        exit 0
    fi
    sleep 1
    COUNT=$((COUNT + 1))
done

# 강제 종료 시도
if ps -p "$PID" > /dev/null; then
    echo "Force killing $APP_NAME (PID: $PID)..."
    kill -9 "$PID"
    sleep 2 # 강제 종료 후 잠시 대기

    if ! ps -p "$PID" > /dev/null; then
        echo "$APP_NAME stopped forcefully."
        rm -f "$PID_FILE"
        exit 0
    else
        echo "Error: Failed to stop $APP_NAME even after SIGKILL."
        exit 1
    fi
fi
```

### 스크립트 권한 설정
`start.sh`, `stop.sh` 파일에 700 권한을 부여한다. (700: 소유자에게 읽기, 쓰기, 실행 권한)
```bash
# start.sh stop.sh 파일의 권한을 설정한다.
chmod 700 start.sh stop.sh
```

## JAR 파일 전송
빌드된 JAR 파일을 서버의 배포 경로 `/home/ubuntu/app/current`로 옮긴다.
파일명은 start.sh에서 설정한 `app.jar`로 변경한다.

SFTP 클라이언트를 사용해도 되고, 로컬에서 scp 명령어를 사용하여 옮길 수도 있다.
```bash
scp D:/dev/app.jar ubuntu@web:/home/ubuntu/app/current/app.jar
```

## 애플리케이션 실행 및 확인
웹 서버에서 `start.sh` 스크립트로 애플리케이션을 실행하고, 브라우저에서 `http://<웹 서버 public IP>/hello`로 접속하여 `Hello World!` 텍스트가 보여지는지 확인한다.
```bash
# 애플리케이션 실행
/home/ubuntu/app/start.sh

# 로그 확인
tail -f /home/ubuntu/app/logs/app.out

# 애플리케이션 종료
/home/ubuntu/app/stop.sh
```
