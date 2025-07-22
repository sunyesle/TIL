# [Docker] error getting credentials 에러

Docker Desktop 재설치 이후 Docker Compose 실행 시 다음과 같은 에러가 발생했다.
```log
error getting credentials - err: exec: "docker-credential-desktop": executable file not found in %PATH%, out: ``
`docker-compose` process finished with exit code 1      
```

## 해결 방법
### 1. 로그아웃 후 재로그인

### 2. ~/.docker/config.json 파일 수정
`C:\Users\{사용자명}\.docker\config.json` 파일의 `credsStore`를 `credStore`로 수정한다.
```json
{
	"auths": {},
	"credStore": "desktop",
	"currentContext": "desktop-linux"
}
```

---
**Reference**<br>
- https://forums.docker.com/t/docker-credential-desktop-exe-executable-file-not-found-in-path-using-wsl2/100225
