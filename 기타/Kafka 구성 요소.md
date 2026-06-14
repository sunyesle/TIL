# Kafka 구성 요소

## Broker
카프카를 구성하는 물리적인 서버.

프로듀서와 컨슈머 사이에서 메시지를 저장, 관리한다.

<img width="171" height="51" alt="broker" src="https://github.com/user-attachments/assets/0c9429f9-acd5-4373-8ffe-f17f66d43ccf" />

## Kafka Cluster
여러 대의 Kafka Broker로 구성된 시스템.

확장성과 고가용성을 위하여 Broker들이 클러스터로 구성된다.

<img width="181" height="231" alt="kafka cluster" src="https://github.com/user-attachments/assets/41c6cd6f-4964-4a29-ab87-95de48959afe" />

## Topic
카프카 메시지를 묶는 논리적인 단위.

Topic은 한 개 이상의 Partition으로 나누어지게 된다.

<img width="501" height="231" alt="topic" src="https://github.com/user-attachments/assets/0faaf29e-59c3-4774-9a89-6979dcf6690f" />

## Partition
Topic 내에서 데이터가 분할되어 저장되는 물리적인 단위.

Partition은 1개의 leader replica와 0개 이상의 follower replica로 구성된다.

Partition이 여러 개일 경우 Producer가 보낸 메시지의 순서는 보장될 수 없지만, 각 Partition 안에서의 메시지 순서는 보장된다.

카프카의 복제(Replication)는 Partition 단위로 이루어진다.

<img width="501" height="231" alt="partition" src="https://github.com/user-attachments/assets/e78d5d8f-e0dd-4103-a253-3529d102968c" />

### Leader Partition
프로듀서 또는 컨슈머와 직접 통신하여 읽기, 쓰기를 수행하는 파티션.

### Follower Partition
리더 파티션의 데이터를 복제하여 보관하는 파티션.

리더 파티션이 속해있는 Broker에 장애가 발생한다면, 팔로워 파티션 중 하나가 리더로 선출된다.

<img width="491" height="308" alt="leader follower" src="https://github.com/user-attachments/assets/2c3b0728-f281-4561-9560-9f217409cfa2" />

### Replication
Kafka는 높은 가용성을 보장하기 위해 파티션을 여러 브로커에 복제한다. 이를 통해, 하나의 브로커에 장애가 발생해도 시스템은 계속 작동할 수 있다.

각 Topic 별로 Replication Factor 값을 설정할 수 있다.

<img width="671" height="441" alt="replica_0" src="https://github.com/user-attachments/assets/cfd2f37d-218e-4e17-a439-f2b7a64bb9fa" />

<br>

<img width="672" height="442" alt="replica_1" src="https://github.com/user-attachments/assets/4cb6e1e8-5bee-4efa-ae19-d3f28ee4d073" />

## Segment
메시지를 저장하는 물리적인 단위.

각 메시지들은 세그먼트라는 로그 파일의 형태로 브로커의 로컬 디스크에 저장된다.

세그먼트가 일정 크기에 도달하면 새로운 세그먼트가 생성된다.

<img width="541" height="262" alt="segment" src="https://github.com/user-attachments/assets/69d2c926-c05b-4d8b-9357-385c94cc5d10" />

## Record
Kafka에서 전송되는 메시지 또는 데이터의 최소 단위.

## Producer
메지시를 생산해서 Topic으로 메시지를 보내는 애플리케이션.

## Consumer
Topic의 메시지를 가져와서 소비하는 애플리케이션.

<img width="581" height="91" alt="consumer" src="https://github.com/user-attachments/assets/387b306e-0267-4a30-bd6d-e7634c1cb084" />

---
**Reference**<br>
- https://magpienote.tistory.com/252
- https://colevelup.tistory.com/18
- https://unit-15.tistory.com/136
- https://velog.io/@jwpark06/Kafka-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%A1%B0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0
- https://magpienote.tistory.com/251?category=1028131
