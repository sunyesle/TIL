# Kafka 구성 요소

## Kafka Cluster
여러 대의 Kafka Broker로 구성된 시스템.
확장성과 고가용성을 위하여 Broker들이 클러스터로 구성된다.

## Broker
카프카를 구성하는 물리적인 서버.

## Topic
카프카 메시지를 묶는 논리적인 단위.
Topic은 한 개 이상의 Partition으로 나누어지게 된다.

## Partition
Topic 내에서 데이터가 분할되어 저장되는 물리적인 단위.
Partition이 여러 개일 경우 Producer가 보낸 메시지의 순서는 보장될 수 없지만, 각 Partition 안에서의 메시지 순서는 보장된다.
카프카의 복제는 Partition단위로 이루어진다.

## Segment
메시지를 저장하는 물리적인 단위. 각 메시지들은 세그먼트라는 로그 파일의 형태로 브로커의 로컬 디스크에 저장된다.

## Record
Kafka에서 전송되는 메시지 또는 데이터의 최소 단위.

## Producer
메지시를 생산해서 Topic으로 메시지를 보내는 애플리케이션.

## Consumer
Topic의 메시지를 가져와서 소비하는 애플리케이션.

---
**Reference**<br>
- https://magpienote.tistory.com/252
- https://colevelup.tistory.com/18
