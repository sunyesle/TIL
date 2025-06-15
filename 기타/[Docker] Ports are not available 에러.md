# [Docker] Ports are not available 에러

도커 컨테이너 실행 시 포트 충돌 오류 해결 방법
```
Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:8086 -> 0.0.0.0:0: listen tcp 0.0.0.0:8086: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
```

## 1. 포트를 점유중인 프로세스 종료
CMD에서 다음 명령어를 입력한다.

**포트를 점유중인 프로세스의 PID를 확인**
```
netstat -ano | findstr :[포트번호]
```

**프로세스 종료**
```
taskkill /f /pid [PID]
```

### 예시
```
C:\WINDOWS\system32>netstat -ano | findstr :9000
  TCP    0.0.0.0:9000           0.0.0.0:0              LISTENING       3488
  TCP    127.0.0.1:2027         127.0.0.1:9000         ESTABLISHED     6540
  TCP    127.0.0.1:9000         127.0.0.1:2027         ESTABLISHED     3488
  TCP    [::]:9000              [::]:0                 LISTENING       3488

C:\WINDOWS\system32>taskkill /f /pid 3488
성공: 프로세스(PID 3488)가 종료되었습니다.

C:\WINDOWS\system32>netstat -ano | findstr :9000
```

## 2. WinNAT 서비스 재시작
포트를 점유한 프로세스도 찾을 수 없는 데도 포트를 사용할 수 없다는 오류가 발생하고 있다면,
시스템이 해당 포트를 동적으로 예약했을 가능성이 높다.

CMD를 관리자 권한으로 실행해서 다음 명령어를 입력한다.

**WinNAT 서비스가 점유 중인 포트 확인**
```
netsh interface ipv4 show excludedportrange protocol=tcp
```

**WinNAT 서비스를 재시작하여 점유하고 있는 포트를 해제**
```
net stop winnat
net start winnat
```

### 예시
```
C:\WINDOWS\system32>netsh interface ipv4 show excludedportrange protocol=tcp
프로토콜 tcp 포트 제외 범위

시작 포트    끝 포트
----------    --------
      7654        7753
      7754        7853
      7939        8038
      8039        8138
     10226       10325
     10326       10425
     10426       10525
     10526       10625
     50000       50059     *

* - 관리 포트 제외입니다.

C:\WINDOWS\system32>net stop winnat
Windows NAT Driver 서비스를 잘 멈추었습니다.

C:\WINDOWS\system32>net start winnat
Windows NAT Driver 서비스가 잘 시작되었습니다.

C:\WINDOWS\system32>netsh interface ipv4 show excludedportrange protocol=tcp
프로토콜 tcp 포트 제외 범위

시작 포트    끝 포트
----------    --------
     50000       50059     *

* - 관리 포트 제외입니다.
```

---
**Reference**<br>
- https://velog.io/@twonezero_98/Docker-%EC%8B%A4%ED%96%89-%EC%8B%9C-port-%EC%82%AC%EC%9A%A9-%EB%B6%88%EA%B0%80-%EC%97%90%EB%9F%AC


