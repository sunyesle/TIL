# Kafka 구성 요소

## Broker
카프카를 구성하는 물리적인 서버.

프로듀서와 컨슈머 사이에서 메시지를 저장, 관리한다.

![broker](https://github.com/user-attachments/assets/6607fcf8-5e3d-4fdb-8c08-620033fb5276)

## Kafka Cluster
여러 대의 Kafka Broker로 구성된 시스템.

확장성과 고가용성을 위하여 Broker들이 클러스터로 구성된다.

![kafka cluster](https://github.com/user-attachments/assets/41fd6c0b-f525-4388-a31b-986a4fc662df)

## Topic
카프카 메시지를 묶는 논리적인 단위.

Topic은 한 개 이상의 Partition으로 나누어지게 된다.

![topic](https://github.com/user-attachments/assets/52e7f0e8-fa52-4cde-b613-064f9ddd66ed)

## Partition
Topic 내에서 데이터가 분할되어 저장되는 물리적인 단위.

Partition은 1개의 leader replica와 0개 이상의 follower replica로 구성된다.

Partition이 여러 개일 경우 Producer가 보낸 메시지의 순서는 보장될 수 없지만, 각 Partition 안에서의 메시지 순서는 보장된다.

카프카의 복제(Replication)는 Partition 단위로 이루어진다.

![partition](https://github.com/user-attachments/assets/a4315f35-ed73-4994-a305-217e3d0d2020)

### Leader Partition
프로듀서 또는 컨슈머와 직접 통신하여 읽기, 쓰기를 수행하는 파티션.

### Follower Partition
리더 파티션의 데이터를 복제하여 보관하는 파티션.

리더 파티션이 속해있는 Broker에 장애가 발생한다면, 팔로워 파티션 중 하나가 리더로 선출된다.

![reader-follwer](https://github.com/user-attachments/assets/d458adac-d329-4a12-a509-a177ce955309)

### Replication
Kafka는 높은 가용성을 보장하기 위해 파티션을 여러 브로커에 복제한다. 이를 통해, 하나의 브로커에 장애가 발생해도 시스템은 계속 작동할 수 있다.

각 Topic 별로 Replication Factor 값을 설정할 수 있다.

![replication_0](https://github.com/user-attachments/assets/2f476e37-5b4e-42cc-a736-d0850337f4f4)

![replication_1](https://github.com/user-attachments/assets/157a9c97-443d-45ec-9a21-cb54ed97ee7f)

## Segment
메시지를 저장하는 물리적인 단위.

각 메시지들은 세그먼트라는 로그 파일의 형태로 브로커의 로컬 디스크에 저장된다.

세그먼트가 일정 크기에 도달하면 새로운 세그먼트가 생성된다.

![segment](https://github.com/user-attachments/assets/a744efe9-0dce-4b0d-af03-a6b5eadf64bd)

## Record
Kafka에서 전송되는 메시지 또는 데이터의 최소 단위.

## Producer
메지시를 생산해서 Topic으로 메시지를 보내는 애플리케이션.

## Consumer
Topic의 메시지를 가져와서 소비하는 애플리케이션.

![producer_consumer](https://github.com/user-attachments/assets/7861b220-83e1-4af0-8c01-995fc224c09a)


---
**Reference**<br>
- https://magpienote.tistory.com/252
- https://colevelup.tistory.com/18
- https://unit-15.tistory.com/136
- https://velog.io/@jwpark06/Kafka-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%A1%B0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0
- https://magpienote.tistory.com/251?category=1028131
