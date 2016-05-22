
# Learning Kafka

## Introduction/Jargon

Topics

Producers

Consumers

Nodes in a Kafka cluster are called Brokers.

Topic can be split into multiple partitions. Each partition will have it's offset.

Each message is uniquely identified by:
```
TopicName,Partition#,UniqueOffsetWithinPartition
```

Each partition is like a log file, a producer keeps writing to the partition and updates the offset. And multiple consumers will be reading from the partition picking up from where they left off using their specific offsets.

You can save the data in Kafka for a defined duration, a day, a week, depending on the number of brokers and the storage available.

Consumers can be:
Adhoc consumers
Batch consumers

HA is availabe by having partition replicas on different brokers.

All the Kafka reads/writes go to the leader-partition (the original parition). The replica partitions are updated just in case of broker failure. Producers/Consumers never ever interact with the replicas.




```
adfdfad
```
