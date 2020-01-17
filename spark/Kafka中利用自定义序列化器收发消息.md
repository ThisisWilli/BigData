# Kafka中利用自定义序列化器收发消息

## 序列化器

* Kafka中，自带的序列化器有 int、short、long、float、double、String 、byte[]等数据类型，但这种序列化器很难满足复杂数据的收发需求，如发送一个商户的数据，数据中包含user_id,age_range,gender,merchant_id,label等信息，数据如下所示，此时就需要自定义序列化器

  ```
  user_id,age_range,gender,merchant_id,label
  163968,0,0,4378,-1
  163968,0,0,2300,-1
  163968,0,0,1551,-1
  163968,0,0,4343,-1
  163968,0,0,4911,-1
  163968,0,0,4043,-1
  163968,0,0,2138,-1
  ```

## 准备工作

* 将对象序列化字节流的包我们选择fastjson，在pox.xml中添加fastjson和kafka依赖

  ```xml
  		<dependency>
              <groupId>org.apache.kafka</groupId>
              <artifactId>kafka_2.12</artifactId>
              <version>0.11.0.3</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>fastjson</artifactId>
              <version>1.2.62</version>
          </dependency>
  ```

* 创建Person类

  ```java
  package streaming.kafka.bean;
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka.bean
   * \* author: Willi Wei
   * \* date: 2019-12-23 15:11:49
   * \* description:
   * \
   */
  public class Person {
      private String userId;
      private String ageRange;
      private String gender;
      private String merchantId;
      private String label;
  
      public Person() {
      }
  
      public Person(String userId, String ageRange, String gender, String merchantId, String label) {
          this.userId = userId;
          this.ageRange = ageRange;
          this.gender = gender;
          this.merchantId = merchantId;
          this.label = label;
      }
  
      public String getUserId() {
          return userId;
      }
  
      public void setUserId(String userId) {
          this.userId = userId;
      }
  
      public String getAgeRange() {
          return ageRange;
      }
  
      public void setAgeRange(String ageRange) {
          this.ageRange = ageRange;
      }
  
      public String getGender() {
          return gender;
      }
  
      public void setGender(String gender) {
          this.gender = gender;
      }
  
      public String getMerchantId() {
          return merchantId;
      }
  
      public void setMerchantId(String merchantId) {
          this.merchantId = merchantId;
      }
  
      public String getLabel() {
          return label;
      }
  
      public void setLabel(String label) {
          this.label = label;
      }
  
      @Override
      public String toString() {
          return "Person{" +
                  "userId='" + userId + '\'' +
                  ", ageRange='" + ageRange + '\'' +
                  ", gender='" + gender + '\'' +
                  ", merchantId='" + merchantId + '\'' +
                  ", label='" + label + '\'' +
                  '}';
      }
  }
  ```

* 创建序列化器和反序列化器

  ```java
  package streaming.kafka.bean;
  
  import com.alibaba.fastjson.JSON;
  import org.apache.kafka.common.serialization.Serializer;
  
  import java.util.Map;
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka.bean
   * \* author: Willi Wei
   * \* date: 2019-12-23 15:22:43
   * \* description:
   * \
   */
  public class PersonSerializer implements Serializer<Person> {
      public void configure(Map map, boolean b) {
  
      }
  
      public byte[] serialize(String topic, Person person) {
          return JSON.toJSONBytes(person);
      }
  
      public void close() {
  
      }
  }
  ```

  ```java
  package streaming.kafka.bean;
  import com.alibaba.fastjson.JSON;
  import org.apache.kafka.common.serialization.Deserializer;
  
  
  import java.util.Map;
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka.bean
   * \* author: Willi Wei
   * \* date: 2019-12-23 15:13:53
   * \* description:
   * \
   */
  public class PersonDeserializer implements Deserializer<Person> {
      public void configure(Map<String, ?> map, boolean b) {
  
      }
  
      public Person deserialize(String topic, byte[] data) {
          return JSON.parseObject(data, Person.class);
      }
  
      public void close() {
  
      }
  }
  ```

## 创建生产者和消费者

* 创建生产者，生产者读取本地csv文件

  ```java
  package streaming.kafka;
  
  import org.apache.kafka.clients.producer.KafkaProducer;
  import org.apache.kafka.clients.producer.ProducerRecord;
  import streaming.kafka.bean.Person;
  
  import java.io.BufferedReader;
  import java.io.FileReader;
  import java.util.Properties;
  import java.util.concurrent.TimeUnit;
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka
   * \* author: Willi Wei
   * \* date: 2019-12-22 20:15:26
   * \* description:测试发送序列化的person数据
   * \
   */
  public class JavaKafkaProducer {
      public static void main(String[] args) {
          Properties props = new Properties();
          props.put("bootstrap.servers", "node02:9092,node03:9092,node04:9092");
          props.put("acks", "all");
          props.put("retries", 0);
          props.put("batch.size", 16384);
          props.put("linger.ms", 0);
          props.put("buffer.memory", 33554432);
          // key的序列化方式
          props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
          // value的序列化方式
          props.put("value.serializer", "streaming.kafka.bean.PersonSerializer");
          KafkaProducer<String, Person> producer = new KafkaProducer<String, Person>(props);
          try {
              BufferedReader reader = new BufferedReader(new FileReader("data/test2000.csv"));
              reader.readLine();
              String line = null;
              while((line=reader.readLine())!=null){
                  //CSV格式文件为逗号分隔符文件，这里根据逗号切分
                  String items[] = line.split(",");
                  Person person = new Person();
                  person.setUserId(items[0]);
                  person.setAgeRange(items[1]);
                  person.setGender(items[2]);
                  person.setMerchantId(items[3]);
                  if (items.length == 4)
                  {
                      person.setLabel("null");
                  }else {
                      person.setLabel(items[4]);
                  }
                  producer.send(new ProducerRecord<String, Person>("topic1223",person.getUserId(), person));
              }
          } catch (Exception e) {
              e.printStackTrace();
          }
          producer.close();
      }
  }
  ```

* 创建消费者

  ```java
  package streaming.kafka;
  
  import org.apache.kafka.clients.consumer.ConsumerRecord;
  import org.apache.kafka.clients.consumer.ConsumerRecords;
  import org.apache.kafka.clients.consumer.KafkaConsumer;
  import org.apache.spark.streaming.Durations;
  import streaming.kafka.bean.Person;
  
  import java.util.Arrays;
  import java.util.Iterator;
  import java.util.Properties;
  
  /**
   * \* project: SparkStudy
   * \* package: streaming.kafka
   * \* author: Willi Wei
   * \* date: 2019-12-23 15:51:37
   * \* description:
   * \
   */
  public class JavaKafkaConsumer {
      public static void main(String[] args) {
          Properties props = new Properties();
          props.put("bootstrap.servers", "node02:9092,node03:9092,node04:9092");
          props.put("group.id", "group01");
          props.put("enable.auto.commit", "false");
          props.put("auto.offset.reset", "earliest");
          props.put("auto.commit.interval.ms", "1000");
          // key的序列化方式
          props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
          // value的序列化方式
          props.put("value.deserializer", "streaming.kafka.bean.PersonDeserializer");
          KafkaConsumer<String, Person> consumer = new KafkaConsumer<String, Person>(props);
          consumer.subscribe(Arrays.asList("topic1223"));
          try {
              while(true){
                  ConsumerRecords<String, Person> records = consumer.poll(5000);
                  System.out.println("接收到的信息总数为：" + records.count());
                  for (ConsumerRecord<String, Person> record : records) {
                      System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value().toString());
                  }
              }
          } catch (Exception e) {
              e.printStackTrace();
          }
  
      }
  }
  ```

  输出为

  ```
  offset = 14107, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='4898', label='-1'}
  offset = 14108, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='1405', label='-1'}
  offset = 14109, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='4760', label='-1'}
  offset = 14110, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='1953', label='-1'}
  offset = 14111, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='1036', label='-1'}
  offset = 14112, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='567', label='-1'}
  offset = 14113, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='4676', label='-1'}
  offset = 14114, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='1862', label='-1'}
  offset = 14115, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='2674', label='-1'}
  offset = 14116, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='4920', label='-1'}
  offset = 14117, key = 372096, value = Person{userId='372096', ageRange='0', gender='0', merchantId='517', label='-1'}
  ```

  