```
https://github.com/gwenshap/kafka-examples
```

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

In config/server.properties:
- broker.id=0
- listeners=PLAINTEXT://:9092
- port=9092
- log.dirs=/Users/schipp200/Documents/app/kafka_2.11-0.9.0.1/test/kafka-log-1

Starting Zookeeper:
```
bin/zookeeper-server-start.sh config/zookeeper.properties &
```

Starting Broker:
```
bin/kafka-server-start.sh config/server.properties &
```

cp server.properties server2.properties
Make changes to server2.properties as:
- broker.id=1
- listeners=PLAINTEXT://:9091
- port=9091
- log.dirs=/Users/schipp200/Documents/app/kafka_2.11-0.9.0.1/test/kafka-log-2


Starting Broker-2:
```
bin/kafka-server-start.sh config/server2.properties &
```

You must always mention the zookeeper information when working with the topics:
```
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic first --partitions 2 --replication-factor 2
bin/kafka-topics.sh --zookeeper localhost:2181 --list
```

* In-Sync Replica: If a replica has the exact same data as that of the leader partition.


Starting a Console Producer:
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic first
```

Starting a Console Consumer:
```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic first
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic first --from-beginning
```

Producers have to know the broker-list, where as the Consumers have to know the zookeeper.

The Consumers store their partition offsets (where they last read from) in the zookeeper.

### Producer

Messages in Kafka are serialized and sent as ByteArrays.

You can have the same Producer send a message to different topics. Topic is not part of the Producer configuration.

```
request.required.acks=0

```
Required Acknowledgements for a produce request:
- 0 = Means, it is succcessful after the message immediately got out of the hands.
- 1 = Means, Leader got it, but it didn't replicate to any of the followers yet.
- (-1) = Means, not just the Leader, but all the In-Sync Replicas (ISR) got it as well.
    - Safer, but have to wait more per message success


```
producer_type=sync

```
This is how long you want to wait for an acknowledgement from the server.

Asynchronous Producers batches the messages, and hence you get the messages to the consumer like a whoosh.

When using asynchronous producers, it's important to call the close() method, otherwise it would not flush the messages left in the buffer.

### Consumer

- High Level Consumer
    - Kafka's out of the box consumer.
    - Tracks which offsets were read in zookeeper/kafka
    - Automatically balances partitions between threads/processes.
- Simple Consumer
    - Low level API
    - Allows more control over how data is read and tracked.
    - Requires attention to low level details
- New Consumer
    - Automatically handles failures and balances partitions
    - Can choose between manual and automatic offset management


# Learning Flume

Flume Agent has the following:

- Sources
    - Sources are the modules which produces logs.
    - Example: Twitter, logs (written to disk, etc), webserver, Kafka topics
- Interceptors
    - Interceptors receives the data from source
    - They mask, re-format, validate the data received.
- Selectors
    - Helps choose what kind of processing happens to the data that is received.
    - Interceptors may mark some of the messages critical, suspicious, etc and this data can be sent to a specific channel for special handling.
    - Data is sent to a channel.
- Channels
    - Channel can be a memory (buffer memory) that receives the flume events.
    - Channel can be a file, but can be a single point of failure
    - Channel can be a Kafka, where these modified events (which went through interceptor, selector) can be produced to a new Kafka topic.
- Sinks
    - Final destination of the data.
    - Can be HDFS, HBase, Solr, Kafka.
