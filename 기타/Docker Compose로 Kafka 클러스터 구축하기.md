# Docker Compose로 Kafka 클러스터 구축하기

## docker-compose.yml
```yml
networks:
  kafka_network:

volumes:
  Kafka00:
    driver: local
  Kafka01:
    driver: local
  Kafka02:
    driver: local
    
services:
# Kafka00
  Kafka00Service:
    image: bitnami/kafka:3.7.0
    restart: unless-stopped
    container_name: Kafka00Container
    ports:
      - '10000:9094'
    environment:
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # KRaft settings
      - KAFKA_CFG_BROKER_ID=0
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      # Listeners
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka00Service:9092,EXTERNAL://127.0.0.1:10000
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      # Clustering
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
    networks:
      - kafka_network
    volumes:
      - "Kafka00:/bitnami/kafka"
# Kafka01
  Kafka01Service:
    image: bitnami/kafka:3.7.0
    restart: always
    container_name: Kafka01Container
    ports:
      - '10001:9094'
    environment:
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # KRaft settings
      - KAFKA_CFG_BROKER_ID=1
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      # Listeners
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka01Service:9092,EXTERNAL://127.0.0.1:10001
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      # Clustering
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
    networks:
      - kafka_network
    volumes:
      - "Kafka01:/bitnami/kafka"
# Kafka02
  Kafka02Service:
    image: bitnami/kafka:3.7.0
    restart: always
    container_name: Kafka02Container
    ports:
      - '10002:9094'
    environment:
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # KRaft settings
      - KAFKA_CFG_BROKER_ID=2
      - KAFKA_CFG_NODE_ID=2
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      # Listeners
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka02Service:9092,EXTERNAL://127.0.0.1:10002
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      # Clustering
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
    networks:
      - kafka_network
    volumes:
      - "Kafka02:/bitnami/kafka"
      
  KafkaWebUiService:
    image: provectuslabs/kafka-ui:latest
    restart: always
    container_name: KafkaWebUiContainer
    ports:
      - '8080:8080'
    environment:
      - KAFKA_CLUSTERS_0_NAME=Local-Kraft-Cluster
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=Kafka00Service:9092,Kafka01Service:9092,Kafka02Service:9092
      - DYNAMIC_CONFIG_ENABLED=true
      - KAFKA_CLUSTERS_0_AUDIT_TOPICAUDITENABLED=true
      - KAFKA_CLUSTERS_0_AUDIT_CONSOLEAUDITENABLED=true
      #- KAFKA_CLUSTERS_0_METRICS_PORT=9999
    depends_on:
      - Kafka00Service
      - Kafka01Service
      - Kafka02Service
    networks:
      - kafka_network
```

## Kafka Environment 설정
- `KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE`
  - 존재하지 않는 토픽에 대한 메시지가 수신될 때 자동으로 토픽을 생성하도록 설정한다.
- `KAFKA_CFG_BROKER_ID`
  - 브로커의 고유 식별자
- `KAFKA_CFG_NODE_ID`
  - 브로커의 고유 식별자
- `KAFKA_KRAFT_CLUSTER_ID`
  - 클러스터의 고유 식별자
- `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS`
  - 클러스터의 controller quorum에 투표할 수 있는 노드
  - `id@host:port` 형식을 사용
- `KAFKA_CFG_PROCESS_ROLES`
  - 브로커가 수행할 역할을 정의한다.
  - `controller,broker`, `controller`, `broker`
- `ALLOW_PLAINTEXT_LISTENER`
  - Plaintext 프로토콜을 사용하는 리스너 허용 여부
- `KAFKA_CFG_LISTENERS`
  - Kafka가 클라이언트로부터 수신하는 네트워크 인터페이스와 포트
- `KAFKA_CFG_ADVERTISED_LISTENERS`
  - Kafka가 클라이언트에게 노출할 네트워크 인터페이스와 포트
- `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP`
  - 각 리스너의 보안 프로토콜 매핑
- `KAFKA_CFG_CONTROLLER_LISTENER_NAMES`
  - 컨트롤러 역할을 하는 리스너 지정
- `KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR`
  - Offset Topic의 복제본 수
- `KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR`
  - 트랜잭션 상태 로그의 복제본 수
- `KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR`
  - 트랜잭션 로그의 최소 ISR(In-Sync Replica)

---
**Reference**<br>
- https://github.com/ArminShoeibi/KafkaDockerCompose/blob/main/docker-compose-cluster.yml
- https://breezymind.com/silicon-mac-kafka-cluster-docker-compose
- https://kylo8.tistory.com/entry/Kafka-Docker-Compose%EB%A5%BC-%ED%86%B5%ED%95%B4-Kafka-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-Kraft-%EB%AA%A8%EB%93%9C
- https://curiousjinan.tistory.com/entry/mac-kafka-kraft-docker
