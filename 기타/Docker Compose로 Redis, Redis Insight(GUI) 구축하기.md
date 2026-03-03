# Docker Compose로 Redis, Redis Insight(GUI) 구축하기
## docker-compose.yml
```yml
services:
  redis:
    container_name: redis
    image: redis:7.4.2-alpine
    ports:
      - 6379:6379
    command: [ "redis-server" ]
    volumes:
      - ./data/redis:/data
  redisinsight:
    container_name: redisinsight
    image: redis/redisinsight:latest
    ports:
      - 5540:5540
    depends_on:
      - redis
    volumes:
      - ./data/redis_insight:/db
```