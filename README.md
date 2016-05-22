
# Learning Kafka

## Introduction/Jargon

Kafka provides a fast, distributed, highly scalable, highly available pub-sub messaging system.

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


### Consumer Groups

Consumers can be a part of Consumer Group.

The number of consumers in a ConsumerGroup can be from 1 to Total#ofPartitionsForThatTopic. i.e. A maximum of consumer per partition is possible in a ConsumerGroup

If consumers are part of a ConsumerGroup, then each consumer will get part of the data for that topic. Each consumer in the ConsumerGroup gets different set of the data, so that there are no duplicates. It allows for scaling, as if one of the consumer slows down, we can just add more consumers within the ConsumerGroup and does load balancing on the consumer end.

For Example, for a topic, the partitions are:
- Server1
    - P0
    - P3
- Server2
    - P1
    - P2

Then the Consumers can be organized in a ConsumerGroup as:
- Consumer Group A
    - C1 - Reads P0, P3 from Server1 
    - C2 - Reads P1, P2 from Server2

- Consumer Group B
    - C3 - Reads P0 from Server1 
    - C4 - Reads P3 from Server1 
    - C5 - Reads P1 from Server2 
    - C6 - Reads P2 from Server2 


## Setting up Kafka Cluster

### Zookeeper

Kafka depends on Zookeeper for:
- Maintaining state
- Know which brokers exists
- Who's the controller
- Who's the leader
- Topics available, etc...

Starting Zookeeper:
```
bin/zookeeper-server-start.sh config/zookeeper.properties &
```

Starting Broker:
```
bin/kafka-server-start.sh config/server.properties &
```
