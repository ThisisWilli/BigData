# Kafka

## 概念

* kafka是一个分布式消息存储系统，默认消息是存储磁盘，默认保存7天
* producer：消息生产者，两种机制，1.轮询，2.key的hash。如果key是null，就是轮询，如果key非null，就按照key的hash
* broker：kafka集群的节点，broker之间没有主从关系，依赖于zookeeper协调，broker负责消息的读写和存储，每隔broker可以管理多个partition
* topic
  * 一类消息/消息队列
  * 每个topic是由多个partition组成，为了提高并行度，由几个组成，可以创建指定
* **一个topic中包含多个broker，每个broker对应一个或者多个partition，但是一个partition只能对应一个broker**
* partition
  * 是组成partition的单元，直接接触磁盘，消息是append到每个partition上的
  * 每个partition内部消息是强有序的，FIFO
  * 每个partition有副本，几个副本，创建topic时，可以指定
* consumer
  * 每个consumer都有自己的消费者组
  * 每个消费者组在消费同一个topic时，这个topic中的数据只能被消费一次
  * 不同的消费者组消费同一个topic互不影响
  * kafka0.8之前，consumer自己在zookeeper中维护消费者offset，0.8之后，消费者offset是通过kafka集群来维护的
* zookeeper存储原数据
  * broker，topic，partition
  * kafka0.8之前还可以存储消费者offset

## Kafka集群操作

### kafka集群搭建

* 上传kafka_2.10-0.8.2.2.tgz到不同的三个节点，并解压`tar zxvf kafka_2.11-0.11.0.3.tgz`

* 改名`[root@node01 kafka]# mv kafka_2.11-0.11.0.3 kafka`

* 配置文件`[root@node01 kafka]# vim ./config/server.properties`，配置broker id，log存放的文件夹，以及zookeeper节点的ip以及端口

  ```
  broker.id=1
  log.dirs=/kafkalog/kafka-logs
  zookeeper.connect=node02:2181,node03:2181,node04:2181
  ```

* 编写脚本文件放入kafka文件夹下

  ```sh
  nohup bin/kafka-server-start.sh   config/server.properties > kafka.log 2>&1 &
  ```

* 将kafka文件夹发送大每个kafka节点上

  ```
  [root@node01 kafka]# scp -r /root/kafka/ node02:/root
  [root@node01 kafka]# scp -r /root/kafka/ node03:/root
  ```

* 更改脚本权限`chmod 755 startkafka.sh`

* 在三个节点中启动集群`[root@node01 kafka]# ./startkafka.sh `

### kafka集群操作

* 查看当前kafka集群中有几个topic

  ```
  [root@node01 bin]# ./kafka-topics.sh --zookeeper node02:2181,node03:2181,node04:2181 --list
  ```

* 创建一个新的topic

  ```
  [root@node01 bin]# ./kafka-topics.sh --zookeeper node02:2181,node03:2181,node04:2181 --create --topic topic1219 --partitions 3 --replication-factor 3
  ```

* 用一台节点console来当kafka的生产者

  ```
  [root@node01 bin]# ./kafka-console-producer.sh --topic topic1219 --broker-list node01:9092,node02:9092,node03:9092
  ```

* 用另一台节点的控制台来当kafka的消费者

  ```
  [root@node02 bin]# ./kafka-console-consumer.sh --zookeeper node02:2181,node03:2181,node04:2181 --topic topic1219
  ```

* 二者卡住之后可以开始生产和消费数据

* 在zookeeper一个节点下进入zookeeper客户端

  ```
  [root@node04 zookeeper-3.4.6]# cd bin/
  [root@node04 bin]# ./zkCli.sh
  ```

* 进入broker文件夹下查看topic信息

  ```
  [zk: localhost:2181(CONNECTED) 0] ls /
  [cluster, controller, brokers, zookeeper, yarn-leader-election, MasterHA1125, hadoop-ha, admin, isr_change_notification, hiverserver2, controller_epoch, consumers, 
  latest_producer_id_block, config, hbase][zk: localhost:2181(CONNECTED) 1] ls brokers
  Command failed: java.lang.IllegalArgumentException: Path must start with / character
  [zk: localhost:2181(CONNECTED) 2] ls /brokers
  [ids, topics, seqid]
  [zk: localhost:2181(CONNECTED) 3] ls /brokers/topics
  [topic1219]
  [zk: localhost:2181(CONNECTED) 4] ls /brokers/topics/topic1219/partitions
  [0, 1, 2]
  [zk: localhost:2181(CONNECTED) 5] ls /brokers/topics/topic1219/partitions/
  
  0   1   2
  [zk: localhost:2181(CONNECTED) 5] ls /brokers/topics/topic1219/partitions/0
  [state]
  [zk: localhost:2181(CONNECTED) 6] get /brokers/topics/topic1219/partitions/0/state
  {"controller_epoch":2,"leader":2,"version":1,"leader_epoch":0,"isr":[2,3,1]}
  cZxid = 0x3e0000004b
  ctime = Thu Dec 19 14:24:26 CST 2019
  mZxid = 0x3e0000004b
  mtime = Thu Dec 19 14:24:26 CST 2019
  pZxid = 0x3e0000004b
  cversion = 0
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 76
  numChildren = 0
  [zk: localhost:2181(CONNECTED) 7] 
  ```

* 也可以维护消费者信息

  ```
  [zk: localhost:2181(CONNECTED) 7] ls /
  [cluster, controller, brokers, zookeeper, yarn-leader-election, MasterHA1125, hadoop-ha, admin, isr_change_notification, hiverserver2, controller_epoch, consumers, 
  latest_producer_id_block, config, hbase][zk: localhost:2181(CONNECTED) 8] ls /con
  
  controller         controller_epoch   consumers          config
  [zk: localhost:2181(CONNECTED) 8] ls /consumers
  [console-consumer-84691]
  [zk: localhost:2181(CONNECTED) 9] ls /consumers/console-consumer-84691
  [ids, owners, offsets]
  [zk: localhost:2181(CONNECTED) 10] ls /consumers/console-consumer-84691/offsets
  [topic1219]
  [zk: localhost:2181(CONNECTED) 11] ls /consumers/console-consumer-84691/offsets/topic1219
  [0, 1, 2]
  [zk: localhost:2181(CONNECTED) 12] get /consumers/console-consumer-84691/offsets/topic1219/0
  1
  cZxid = 0x3e0000005f
  ctime = Thu Dec 19 14:32:40 CST 2019
  mZxid = 0x3e0000005f
  mtime = Thu Dec 19 14:32:40 CST 2019
  pZxid = 0x3e0000005f
  cversion = 0
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 1
  numChildren = 0
  [zk: localhost:2181(CONNECTED) 13] get /consumers/console-consumer-84691/offsets/topic1219/1
  1
  cZxid = 0x3e00000062
  ctime = Thu Dec 19 14:32:40 CST 2019
  mZxid = 0x3e00000062
  mtime = Thu Dec 19 14:32:40 CST 2019
  pZxid = 0x3e00000062
  cversion = 0
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 1
  numChildren = 0
  [zk: localhost:2181(CONNECTED) 14] get /consumers/console-consumer-84691/offsets/topic1219/2
  1
  cZxid = 0x3e00000065
  ctime = Thu Dec 19 14:32:40 CST 2019
  mZxid = 0x3e00000065
  mtime = Thu Dec 19 14:32:40 CST 2019
  pZxid = 0x3e00000065
  cversion = 0
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 1
  numChildren = 0
  ```

* 查看kafka中topic的描述信息

  ```
  [root@node01 bin]# ./kafka-topics.sh --zookeeper node02:2181,node03:2181,node04:2181 --describe
  ```

  输出为,Replicas表示副本

  ```
  Topic:topic1219	PartitionCount:3	ReplicationFactor:3	Configs:
  	Topic: topic1219	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
  	Topic: topic1219	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
  	Topic: topic1219	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3
  ```

* 彻底删除某一个topic中的数据

  * 首先用命令删除一次，这样删除之后会显示topic已经标记删除，但是彻底删除会在7天之后

    ```
    [root@node01 bin]# ./kafka-topics.sh --zookeeper node02:2181, node03:2181,node04:2181 --delete --topic topic1219
    ```

  * 在zookeeper cli命令行中删除topic数据

    ```
    [zk: localhost:2181(CONNECTED) 15] ls /brokers/topics  
    [topic1219]
    [zk: localhost:2181(CONNECTED) 16] rmr /brokers/topics/topic1219
    ```
    
  * 之后在config文件夹中删除topic信息
  
    ```
    [zk: localhost:2181(CONNECTED) 20] ls /config/topics
    [topic1219]
    [zk: localhost:2181(CONNECTED) 21] rmr /config/topics/topic1219
    ```
  
  * 或者直接配置删除也可以。。。



​    

​    