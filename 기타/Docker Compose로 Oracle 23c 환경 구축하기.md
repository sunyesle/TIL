# Docker Compose로 Oracle 23c 환경 구축하기

## docker-compose.yml
```yml
version: '3.8'

services:
  oracle23c:
    image: gvenzl/oracle-free:latest
    container_name: oracle23c
    ports:
      - "1521:1521"
    environment:
      ORACLE_PASSWORD: sys_password
      APP_USER: my_user
      APP_USER_PASSWORD: user_password
    volumes:
      - ./oracle/data:/opt/oracle/oradata
```

---
**Reference**<br>
- https://hub.docker.com/r/gvenzl/oracle-free
