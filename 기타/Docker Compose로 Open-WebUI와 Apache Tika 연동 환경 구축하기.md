# Docker Compose로 Open-WebUI와 Apache Tika 연동 환경 구축하기
## docker-compose.yml
```yml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    volumes:
      - open-webui-data:/app/backend/data
    environment:
      # 문서 처리
      - CONTENT_EXTRACTION_ENGINE=tika
      - TIKA_SERVER_URL=http://tika:9998
      # LLM 연동
      - OPENAI_API_BASE_URL=http://host.docker.internal:8000/v1
      - OPENAI_API_KEY=${API_KEY}
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      - tika

  tika:
    image: apache/tika:latest-full
    container_name: tika
    restart: unless-stopped
    ports:
      - "9998:9998"

volumes:
  open-webui-data:
```

---
**Reference**
- https://docs.openwebui.com/features/chat-conversations/rag/document-extraction/apachetika/

