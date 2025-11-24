# Ubuntu 초기 설정
예시에서 사용한 버전은 `Ubuntu 24.04.3 LTS` 이다.

## 패키지 업데이트
```bash
sudo apt update
sudo apt upgrade -y
```
- `apt update`: 설치 되어있는 패키지에 대해서 최신 버전이 있는지 확인한다.
- `apt upgrade`: `update`를 통해서 확인한 패키지들의 버전을 업그레이드한다.

## 시간대 설정
```bash
ubuntu@instance-web:~$ timedatectl
               Local time: Mon 2025-11-24 11:20:49 UTC
           Universal time: Mon 2025-11-24 11:20:49 UTC
                 RTC time: Mon 2025-11-24 11:20:49
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
ubuntu@instance-web:~$ timedatectl list-timezones | grep Seoul
Asia/Seoul
ubuntu@instance-web:~$ sudo timedatectl set-timezone Asia/Seoul
ubuntu@instance-web:~$ timedatectl
               Local time: Mon 2025-11-24 20:24:26 KST
           Universal time: Mon 2025-11-24 11:24:26 UTC
                 RTC time: Mon 2025-11-24 11:24:26
                Time zone: Asia/Seoul (KST, +0900)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
- timedatectl: 현재 서버의 시간대를 확인한다.
- timedatectl list-timezones: 시간대 목록을 확인한다.
- timedatectl set-timezone [time zone]: 시간대를 변경한다.

## Locale 설정
```bash
ubuntu@instance-web:~$ locale
LANG=C.UTF-8
LANGUAGE=
LC_CTYPE="C.UTF-8"
LC_NUMERIC="C.UTF-8"
LC_TIME="C.UTF-8"
LC_COLLATE="C.UTF-8"
LC_MONETARY="C.UTF-8"
LC_MESSAGES="C.UTF-8"
LC_PAPER="C.UTF-8"
LC_NAME="C.UTF-8"
LC_ADDRESS="C.UTF-8"
LC_TELEPHONE="C.UTF-8"
LC_MEASUREMENT="C.UTF-8"
LC_IDENTIFICATION="C.UTF-8"
LC_ALL=
ubuntu@instance-web:~$ locale -a
C
C.utf8
POSIX
en_US.utf8
ubuntu@instance-web:~$ sudo locale-gen ko_KR.UTF-8
Generating locales (this might take a while)...
  ko_KR.UTF-8... done
Generation complete.
ubuntu@instance-web:~$ locale -a
C
C.utf8
POSIX
en_US.utf8
ko_KR.utf8
ubuntu@instance-web:~$ sudo update-locale LANG=ko_KR.UTF-8
```
- `locale`: 현재 Locale 확인한다.
- `locale -a`: Locale 목록을 확인한다.
- `locale-gen ko_KR.UTF-8`: Locale을 생성한다.
- `update-locale LANG=ko_KR.UTF-8`: Locale을 변경한다.

변경 사항은 세션 재접속 후 확인할 수 있다.
```bash
ubuntu@instance-web:~$ locale
LANG=ko_KR.UTF-8
LANGUAGE=
LC_CTYPE="ko_KR.UTF-8"
LC_NUMERIC="ko_KR.UTF-8"
LC_TIME="ko_KR.UTF-8"
LC_COLLATE="ko_KR.UTF-8"
LC_MONETARY="ko_KR.UTF-8"
LC_MESSAGES="ko_KR.UTF-8"
LC_PAPER="ko_KR.UTF-8"
LC_NAME="ko_KR.UTF-8"
LC_ADDRESS="ko_KR.UTF-8"
LC_TELEPHONE="ko_KR.UTF-8"
LC_MEASUREMENT="ko_KR.UTF-8"
LC_IDENTIFICATION="ko_KR.UTF-8"
LC_ALL=
```
