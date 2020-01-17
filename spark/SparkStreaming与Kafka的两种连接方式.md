# Spark Streaming与Kafka的两种连接方式

## Spark Streaming + Kafka Receiver模式

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SparkStreamingkafka%20Receiver%E6%A8%A1%E5%BC%8F.jpg )

​		Spark Streaming + Kafka模式处理数据采用了receiver接收器的模式，需要一个task一直处于占用接受数据，接收来的数据存储级别：MEMORY_AND_DISK_SER_2这种模式几乎没有用的

* **存在丢失数据的问题**

  当接收完消息后，更新zookeeper offset后，如果Driver挂掉，Driver下的Executor也会被killed，在Executor内存中的数据多少会有丢失。

* **如何解决丢失数据问题**

  开启WAL(write ahead log)，预写日志机制，当Executor备份完数据之后，向HDFS中也备份一份数据，备份完成之后，再去更新消费者offset。如果开启WAL机制，可以将接受来的数据存储级别降级，例如MEMORY_AND_DISK_SER，开启WAL机制要设置checkpoint。

* **开启WAL机制，带来了新的问题**

  必须数据备份到HDFS完成之后，才会更新offset，下一步才会汇报数据位置，再发task处理数据，会造成数据处理的延迟加大

  Receiver模式的并行度：每一批次生成的DStream中的RDD的分区数

  Spark Streaming.blockInterval = 200ms。在batchInterval内每隔200ms，将接收来的数据封装到一个block中，batchInterval时间内生成的这些block组成了当前这个batch。假设batchInterval=5s，5s内生成的一个batch中就有25个block。RDD->batch，RDD->partition,batch->block，这里每一block就是对应RDD中的一个个partition。

  **如何提高RDD的并行度？**当在batchInterval时间的一定情况下，减少spark.streaming.blockInterval值，建议这个值不要低于50ms(如果要处理的数据过少的情况下，反而会拉低效率)，如果将spark.streaming.blockInterva改为100ms，就会生成50个block

### 总结

* 1.存在丢失数据问题，不常用
* 2.就算开启WAL机制解决了丢失数据问题，带来了新的问题，**数据处理延迟大**
* 3.receiver模式底层消费kafka，采用的是High Level Consumer API实现，不关心消费者offset，无法从没批次中获取消费者offset和指定某个offset继续消费数据
* 4.Receiver模式采用zookeeper来维护消费者offset，有offset能够指定位置去消费数据

## Spark Streaming + Kafka Direct模式

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SparkStreaming%E7%9A%84kafka%20Direct%20%E6%A8%A1%E5%BC%8F.jpg )

不需要一个task一直接收数据，当前批次处理数据时，直接读取数据处理。Direct模式并行度与读取的topic中的partition一一对应。Direct模式使用Spark自己来维护消费者offset，默认offset存储在内存中，如果设置了checkpoint，在checkpoint中也有一份，Direct模式可以做到手动维护消费者offset。

如何提高并行度

* 增大读取topic中partition中个数
* 读取过来DStream之后，可以重新分区

Direct模式相对于Receiver模式

* 1.简化了并行度，默认的并行度与读取kafka中topic中partition个数一对一
* 2.Receiver模式采用zookeeper来维护消费者offset，Direct模式使用Spark自己来维护消费者offset
* 3.Receiver模式采用消费kafka的High Level Consumer API实现，Direct模式采用的是读取Kafka的Simple Consumer API可以做到手动维护offset

### 代码实现

* 在pom.xml文件中添加依赖

  ```xml
  	   <dependency>
              <groupId>org.apache.kafka</groupId>
              <artifactId>kafka_2.12</artifactId>
              <version>0.11.0.3</version>
          </dependency>
  		<dependency>
              <groupId>org.apache.spark</groupId>
              <artifactId>spark-streaming_2.11</artifactId>
              <version>2.3.1</version>
              <scope>provided</scope>
          </dependency>
  ```

* 启动kafka集群，并创建一个新的topic

  ```
  [root@node01 bin]# ./kafka-topics.sh --zookeeper node02:2181,node03:2181,node04:2181 --create --topic SparkStreaming-Kafka-1222 --partitions 3 --replication-factor 3
  ```

* 创建ProducerDemo

  ```scala
  package streaming.kafka
  
  import java.util.Properties
  
  import org.apache.kafka.clients.producer.{KafkaProducer, Producer, ProducerRecord}
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka
   * \* author: Willi Wei
   * \* date: 2019-12-22 14:48:41
   * \* description: 
   * \*/
  // kafka生产者详解：https://juejin.im/post/5dcd59abf265da0be53e9db3
  object ProducerDemo {
    def main(args: Array[String]): Unit = {
      val prop = new Properties()
      // 指定请求的kafka集群列表
      prop.put("bootstrap.servers", "node02:9092,node03:9092,node04:9092")
      //    //prop.put("acks", "0")
      prop.put("acks", "all")
      // 请求失败重试次数
      //prop.put("retries", "3")
      // 指定key的序列化方式, key是用于存放数据对应的offset
      prop.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer")
      // 指定value的序列化方式
      prop.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer")
      // 配置超时时间
      prop.put("request.timeout.ms", "60000")
      val producer: KafkaProducer[String, String] = new KafkaProducer[String, String](prop)
      // 测试发送一条简单的消息
      for (i <-0 to(10)){
        producer.send(new ProducerRecord[String, String]("SparkStreaming-Kafka-1222", 0, "python", "斯巴克"))
      }
      // 如果不关闭producer，消息发不出去。。。
      producer.close()
    }
  }
  ```

* 创建一个消费者实例

  ```scala
  package streaming.kafka
  
  import java.util.{Collections, Properties}
  
  import org.apache.kafka.clients.consumer.{ConsumerRecords, KafkaConsumer}
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka
   * \* author: Willi Wei
   * \* date: 2019-12-22 16:00:13
   * \* description: 
   * \*/
  object ConsumerDemo {
    def main(args: Array[String]): Unit = {
      // 配置信息
      val prop = new Properties()
      prop.put("bootstrap.servers", "node02:9092,node03:9092,node04:9092")
      // 指定消费者组
      prop.put("group.id", "group01")
      // 指定消费位置: earliest/latest/none
      prop.put("auto.offset.reset", "earliest")
      // 指定消费的key的反序列化方式
      prop.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
      // 指定消费的value的反序列化方式
      prop.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
      prop.put("enable.auto.commit", "true")
      prop.put("session.timeout.ms", "30000")
      // 得到Consumer实例
      val kafkaConsumer = new KafkaConsumer[String, String](prop)
      // 首先需要订阅topic
      kafkaConsumer.subscribe(Collections.singletonList("SparkStreaming-Kafka-1222"))
      // 开始消费数据
      while (true) {
        // 如果Kafak中没有消息，会隔timeout这个值读一次。比如上面代码设置了2秒，也是就2秒后会查一次。
        // 如果Kafka中还有消息没有消费的话，会马上去读，而不需要等待。
        val msgs: ConsumerRecords[String, String] = kafkaConsumer.poll(20)
        // println(msgs.count())
        val it = msgs.iterator()
        while (it.hasNext) {
          val msg = it.next()
          println(s"partition: ${msg.partition()}, offset: ${msg.offset()}, key: ${msg.key()}, value: ${msg.value()}")
        }
      }
    }
  }
  ```

* 

  



