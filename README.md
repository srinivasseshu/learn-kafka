
# Learning Kafka

## Introduction/Jargon

- Topics
- Producers
- Consumers
- Nodes in a Kafka cluster are called Brokers.

Topic can be split into multiple partitions. Each partition will have it's offset.

Each message is uniquely identified by:
```
TopicName,Partition#,UniqueOffsetWithinPartition
```

Each partition is like a log file, a producer keeps writing to the partition and updates the offset. And multiple consumers will be reading from the partition picking up from where they left off using their specific offsets.

You can save the data in Kafka for a defined duration, a day, a week, depending on the number of brokers and the storage available.

HA is availabe by having partition replicas on different brokers.


Each partition has:
- Leader (original partition):
    - All Kafka reads/writes go to the leader partition. Producers/Consumers never ever interact with the replicas.
    - Leader is responsible for replicating the partition to it's followers.
- Follower (replicas of partition):
    - The replica partitions are updated just in case of broker failure. 
    - Followers acknowledge back to the leader whenever they get a message.


Producer can write to any partition in the topic. A randomizer is used to write to a partition for sometime, and then move to the next partition and so on, and ensure load-balancing.

You can customize load-balancing while writing to the partitions saying, for example,
- Customer names a-e go to partition-0
- Customer names f-m go to partition-1
- Customer names n-z go to partition-2

```
adfdfad
```
