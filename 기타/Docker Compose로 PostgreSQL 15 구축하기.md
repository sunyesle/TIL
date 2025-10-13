# Docker Compose로 PostgreSQL 15 구축하기
## docker-compose.yml
```yml
services:
  postgres:
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - TZ=Asia/Seoul
    ports:
      - "5432:5432"
    volumes:
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./data/postgres:/var/lib/postgresql/data
```
