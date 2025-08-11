# Kafka Replication
## Replication
Kafka에서는 높은 가용성을 위하여 **Replication**이라는 기능을 제공한다.<br>
Replication은 각 Topic의 Partition들을 Kafka Cluster 내의 다른 Broker들로 복제하는 것을 말하며, Topic 생성 시 Replication의 수를 지정할 수 있다.<br>
생성된 Replication은 Leader와 Follower로 나뉘어 ISR(In Sync Replica)이라는 일종의 Replication Group을 형성하여 관리된다.

## Replication factor
**replication factor**(복제 계수)는 Topic 단위로 설정할 수 있다.<br>
기본값은 1로 설정되어 있으며, 이를 변경하고 싶으면 Kafka 설정 파일에서 default.replication.factor 값을 변경하면 된다.

replication factor 값을 높이게 되면 가용성이 증가하지만, 그에 비례하여 디스크 사용량이 배가 되며 Broker의 리소스 사용량 또한 증가한다.<br>
따라서 Topic에 저장되는 Data의 중요도에 따라 적절히 값을 조정해야 한다.

## Leader & Follower
Kafka의 Topic은 1개 이상의 Partition으로 구성되고, 각 Partition은 1개의 **leader replica**와 0개 이상의 **follower replica**로 구성된다.

**leader replica**는 일관성을 보장하기 위해 모든 Producer와 Consumer의 요청을 처리하며 데이터를 저장한다.

**followr replica**는 leader replica의 데이터를 복제하여 동일하게 유지하다가 leader replica가 중단되는 경우 followr replica중 하나가 해당 파티션의 새로운 leader replica로 선출된다.

만약 기존 leader replica가 중단되어 새로운 leader replica가 선출되어야 하는 상황이 발생했을 때 제대로 데이터를 복제해오지 못한 follower replica가 존재한다면, 해당 개체를 제외한 나머지 follower replica 중에서 leader replica를 선출해야할 것이다. <br>
이를 위해서 Kafka에서는 **ISR**(In Sync Replica)이라는 개념을 도입했다.

## ISR(In Sync Replica)
**ISR**은 일종의 replication group으로 leader replica와 제대로 **동기화**가 이루어진 follower replica들로 구성된다.

follower replica가 일정 시간 이상 Fetch 요청을 하지 않거나, 요청은 했지만 시간 안에 Leader replica의 마지막 Offset의 메시지를 복제하지 못한다면 동기화에 실패한 것으로 간주하여 해당 follower replica를 ISR에서 제거한다.<br>

---
**Reference**<br>
- https://damdam-kim.tistory.com/17
