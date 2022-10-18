# Kafka Administration for beginners

## 1 Installation

### 1.1 Install docker and docker-compose

### 1.2 Checkout git repo
```bash
git clone git@github.com:gintek/dataops-poland.git
cd dataops-poland
```

### 1.3 Create kafka cluster
```bash
docker-compose up -d
docker-compose ps
                   Name                                  Command               State                         Ports
-----------------------------------------------------------------------------------------------------------------------------------------
gintek-kafka-meetup-warsaw_kafka_1            start-kafka.sh                   Up      0.0.0.0:32789->9092/tcp, 0.0.0.0:9998->9998/tcp
gintek-kafka-meetup-warsaw_zookeeper_1        /bin/sh -c /usr/sbin/sshd  ...   Up      0.0.0.0:2181->2181/tcp, 22/tcp, 2888/tcp, 3888/tcp
```

## 2 Basics

### 2.1 Enter into broker
```bash
docker exec -ti  kafka_kafka_1 bash
```

### 2.2 Topic list
```bash
kafka-topics.sh --zookeeper zookeeper --list
```

### 2.3 Create new topic
```bash
kafka-topics.sh --zookeeper zookeeper --create -topic warsaw
Missing required argument "[partitions]"
...
```

### 2.4 Create new topic
```bash
kafka-topics.sh --zookeeper zookeeper --create -topic dataops --partitions 1 --replication-factor 1
Created topic dataops.
kafka-topics.sh --zookeeper zookeeper --create -topic warsaw --partitions 1 --replication-factor 1
Created topic warsaw.
```

### 2.5 Show and describe topic

```bash
kafka-topics.sh --zookeeper zookeeper  --list
kafka-topics.sh --zookeeper zookeeper  --describe
```

### 2.6 Produce message from console
```bash
kafka-console-producer.sh --broker-list kafka:9092 --topic warsaw
>warsaw
>dataops
>meetup
```
At the > prompt, type messages and then press Ctrl-d to exit the console Producer.

### 2.7 Read message with CLI
```bash
kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic warsaw --from-beginning
warsaw
dataops
meetup
^CProcessed a total of 3 messages
```
Press Ctrl-c to exit the console consumer.

### 2.8 Read message from log file
```bash
cd /kafka/kafka-logs-*/warsaw-0

ls -1
00000000000000000000.index
00000000000000000000.log
00000000000000000000.timeindex
leader-epoch-checkpoint

kafka-run-class.sh kafka.tools.DumpLogSegments --print-data-log --files 00000000000000000000.log
Dumping 00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1570099704213 size: 74 magic: 2 compresscodec: NONE crc: 2854882044 isvalid: true
| offset: 0 CreateTime: 1570099704213 keysize: -1 valuesize: 6 sequence: -1 headerKeys: [] payload: warsaw
baseOffset: 1 lastOffset: 1 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 74 CreateTime: 1570099706390 size: 73 magic: 2 compresscodec: NONE crc: 1462375733 isvalid: true
| offset: 1 CreateTime: 1570099706390 keysize: -1 valuesize: 5 sequence: -1 headerKeys: [] payload: kafka
baseOffset: 2 lastOffset: 2 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 147 CreateTime: 1570099707938 size: 74 magic: 2 compresscodec: NONE crc: 581971986 isvalid: true
| offset: 2 CreateTime: 1570099707938 keysize: -1 valuesize: 6 sequence: -1 headerKeys: [] payload: meetup

date -d @$((1570099707938/1000))
```

### 2.9 Manage topics
```bash
kafka-topics.sh --zookeeper zookeeper --alter --topic basics --partitions 10
kafka-topics.sh --zookeeper zookeeper --alter --topic basics --partitions 1
kafka-topics.sh --zookeeper zookeeper --delete -topic basics
```

### 2.10 Topics config
```bash
kafka-configs.sh --describe --zookeeper zookeeper:2181 --entity-type topics
Configs for topic 'dataops' are
Configs for topic '__consumer_offsets' are segment.bytes=104857600,cleanup.policy=compact,compression.type=producer
Configs for topic 'warsaw' are
```

## 3 Performance tests

### 3.1 one partition
```bash
kafka-producer-perf-test.sh --print-metrics  --topic warsaw --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196

286926 records sent, 57305.0 records/sec (5.47 MB/sec), 2309.7 ms avg latency, 3298.0 ms max latency.
1000000 records sent, 102197.240675 records/sec (9.75 MB/sec), 3721.68 ms avg latency, 5121.00 ms max latency, 3947 ms 50th, 5006 ms 95th, 5101 ms 99th, 5118 ms 99.9th.
```

### 3.2 multiple partitions
```bash
kafka-topics.sh --zookeeper zookeeper --alter --topic warsaw --partitions 100
WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affected
Adding partitions succeeded!

kafka-producer-perf-test.sh --print-metrics  --topic warsaw --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196

1000000 records sent, 307976.593779 records/sec (29.37 MB/sec), 502.68 ms avg latency, 901.00 ms max latency, 547 ms 50th, 873 ms 95th, 890 ms 99th, 895 ms 99.9th.
```

## 4 Kafka HA

### 4.1 Kafka cluster 3 brokers
```
docker-compose scale kafka=3
docker-compose ps
```

### 4.2 Zookeeper down
```
docker stop kafka_zookeeper_1
```
### 4.3 Create topic
```
kafka-topics.sh --zookeeper zookeeper --create -topic warsaw_test --partitions 1 --replication-factor 1
  [2019-10-01 09:50:06,263] WARN Session 0x0 for server zookeeper:2181, unexpected error, closing socket connection and attempting reconnect (org.apache.zookeeper.ClientCnxn)
  java.lang.IllegalArgumentException: Unable to canonicalize address zookeeper:2181 because it's not resolvable
  	at org.apache.zookeeper.SaslServerPrincipal.getServerPrincipal(SaslServerPrincipal.java:65)
  	at org.apache.zookeeper.SaslServerPrincipal.getServerPrincipal(SaslServerPrincipal.java:41)
  	at org.apache.zookeeper.ClientCnxn$SendThread.startConnect(ClientCnxn.java:1001)
  	at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1060)
  [2019-10-01 09:50:07,373] WARN Session 0x0 for server zookeeper:2181, unexpected error, closing socket connection and attempting reconnect (org.apache.zookeeper.ClientCnxn)
```

### 4.4 Start zookeeper
```
docker start kafka_zookeeper_1
```

### 4.4 Create topic meetup
```
docker exec -ti kafka_zookeeper_1_1 bash
kafka-topics.sh --zookeeper zookeeper --create -topic meetup --partitions 3 --replication-factor 2
  Created topic meetup.
```

### 4.5 Describe topic meetup
```
kafka-topics.sh --zookeeper zookeeper --describe --topic meetup
  Topic:meetup	PartitionCount:3	ReplicationFactor:2	Configs:
  	Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003,1001
  	Topic: meetup	Partition: 1	Leader: 1001	Replicas: 1001,1002	Isr: 1001,1002
  	Topic: meetup	Partition: 2	Leader: 1002	Replicas: 1002,1003	Isr: 1002,1003
```

### 4.6 Stop kafka broker
```
docker stop kafka-meetup-warsaw_kafka_1
```

### 4.7 ISR and unavailable partition/topic
```
docker exec -ti kafka_kafka_2 bash
kafka-topics.sh --zookeeper zookeeper:2181 --describe
  Topic:meetup	PartitionCount:3	ReplicationFactor:2	Configs:
  	Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003
  	Topic: meetup	Partition: 1	Leader: 1002	Replicas: 1001,1002	Isr: 1002
  	Topic: meetup	Partition: 2	Leader: 1002	Replicas: 1002,1003	Isr: 1002,1003
  Topic:warsaw	PartitionCount:9	ReplicationFactor:1	Configs:
  	Topic: warsaw	Partition: 0	Leader: -1	Replicas: 1001	Isr: 1001
  	Topic: warsaw	Partition: 1	Leader: -1	Replicas: 1001	Isr: 1001
  	Topic: warsaw	Partition: 2	Leader: -1	Replicas: 1001	Isr: 1001
```

### 4.8 Under replicated partitions
```
kafka-topics.sh --zookeeper zookeeper:2181 --describe --under-replicated-partitions
  Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003
  Topic: meetup	Partition: 1	Leader: 1002	Replicas: 1001,1002	Isr: 1002
```

### 4.9 Start kafka broker
```
docker start kafka_kafka_1
kafka-topics.sh --zookeeper zookeeper:2181 --describe
  Topic:meetup	PartitionCount:3	ReplicationFactor:2	Configs:
    	Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003,1001
    	Topic: meetup	Partition: 1	Leader: 1002	Replicas: 1001,1002	Isr: 1002,1001
    	Topic: meetup	Partition: 2	Leader: 1002	Replicas: 1002,1003	Isr: 1002,1003
  Topic:warsaw	PartitionCount:9	ReplicationFactor:1	Configs:
    	Topic: warsaw	Partition: 0	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 1	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 2	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 3	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 4	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 5	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 6	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 7	Leader: 1001	Replicas: 1001	Isr: 1001
    	Topic: warsaw	Partition: 8	Leader: 1001	Replicas: 1001	Isr: 1001
```

## 5 Kafka HA

### 5.1 Produce messageses
```
kafka-producer-perf-test.sh --print-metrics  --topic meetup --num-records 1000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196
```
### 5.2 Kafka consumer group

kafka-consumer-groups.sh --bootstrap-server kafka:9092  --list

### 5.3 Create kafka consumer
```
kafka-console-consumer.sh  --bootstrap-server kafka:9092 --topic meetup --from-beginning --consumer-property group.id=dataops-meetup
```

### 5.4 List consumer group

```
kafka-consumer-groups.sh --bootstrap-server kafka:9092  --list
  dataops-meetup
```

### 5.5 Describe consumer
```
kafka-consumer-groups.sh --bootstrap-server kafka:9092  --describe  --group dataops-meetup
  GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
  dataops-meetup   meetup          0          333             333             0               consumer-1-49ca5078-564b-4e5b-98ae-9b17c65e4ee3 /172.21.0.5     consumer-1
  dataops-meetup   meetup          1          333             333             0               consumer-1-49ca5078-564b-4e5b-98ae-9b17c65e4ee3 /172.21.0.5     consumer-1
  dataops-meetup   meetup          2          334             334             0               consumer-1-49ca5078-564b-4e5b-98ae-9b17c65e4ee3 /172.21.0.5     consumer-1
```
### 5.6 Stop kafka-console-consumer.sh

### 5.7 Produce messages
```
kafka-console-producer.sh --broker-list kafka:9092 --topic meetup
  >warsaw
  >kafka
  >meetup
```

### 5.8 Check lag
```
kafka-consumer-groups.sh --bootstrap-server kafka:9092  --describe  --group dataops-meetup
  GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
  dataops-meetup   meetup          2          334             335             1               -               -               -
  dataops-meetup   meetup          1          333             334             1               -               -               -
  dataops-meetup   meetup          0          333             334             1               -               -               -
```

### 5.9 Manually replica assignment
```
kafka-topics.sh --zookeeper zookeeper --create --topic moving --replica-assignment 1001:1002,1001:1002
Created topic moving.
```

### 5.10 Describe topic moving
```
kafka-topics.sh --zookeeper zookeeper --describe --topic moving
  Topic:moving	PartitionCount:2	ReplicationFactor:2	Configs:
    	Topic: moving	Partition: 0	Leader: 1001	Replicas: 1001,1002	Isr: 1001,1002
    	Topic: moving	Partition: 1	Leader: 1001	Replicas: 1001,1002	Isr: 1001,1002

```

## 6 Zookeeper

### 6.1 Connect to ZK
```
docker exec -ti kafka_zookeeper_1  bash
/opt/zookeeper-3.4.13/bin/zkCli.sh
  Connecting to localhost:2181
  2019-10-03 11:26:18,007 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
  2019-10-03 11:26:18,009 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=aa49f8beaf3f
  2019-10-03 11:26:18,009 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_65

```

### 6.2 What ZK know
```
[zk: localhost:2181(CONNECTED) 0] ls /
[log_dir_event_notification, isr_change_notification, zookeeper, admin, consumers, cluster, config, latest_producer_id_block, controller, brokers, controller_epoch]
[zk: localhost:2181(CONNECTED) 1]
```

### 6.3 Brokers in ZK
```
[zk: localhost:2181(CONNECTED) 8]  ls /brokers/ids
  [1003, 1001, 1002]
[zk: localhost:2181(CONNECTED) 9] get /brokers/ids/1001
  {"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT"},"endpoints":["PLAINTEXT://53c57203012f:9092"],"jmx_port":-1,"host":"53c57203012f","timestamp":"1570093574374","port":9092,"version":4}
  cZxid = 0xc9
  ctime = Thu Oct 03 09:06:14 UTC 2019
  mZxid = 0xc9
  mtime = Thu Oct 03 09:06:14 UTC 2019
  pZxid = 0xc9
  cversion = 0
  dataVersion = 1
  aclVersion = 0
  ephemeralOwner = 0x10004e5789c0004
  dataLength = 194
  numChildren = 0
```

### 6.4 Topics in ZK
```
[zk: localhost:2181(CONNECTED) 10] get /brokers/topics/meetup
{"version":1,"partitions":{"2":[1003,1001],"1":[1002,1003],"0":[1001,1002]}}
cZxid = 0x15c
ctime = Thu Oct 03 09:34:03 UTC 2019
mZxid = 0x15c
mtime = Thu Oct 03 09:34:03 UTC 2019
pZxid = 0x15d
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 76
numChildren = 1
```

### Survey
[Your opinion matters!](https://forms.gle/36MPgKxqZBQV7s1f8 "Survey")
