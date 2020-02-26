# BigDataPlatform项目

## Springboot中多模块项目创建

### 配置项目热部署

* 添加依赖

  ```xml
   <!--添加热部署-->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-devtools</artifactId>
              <optional>true</optional>
              <scope>true</scope>
          </dependency>
  ```

* 在plugin中添加

  ```xml
  						<plugin>
                  <!--热部署配置-->
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
                  <configuration>
                      <!--fork:如果没有该项配置,整个devtools不会起作用-->
                      <fork>true</fork>
                  </configuration>
              </plugin>
  ```

* File-Settings-Compiler勾选 Build Project automatically



* 创建pom项目将项目中无关的文件都删除，将父模块中的pom打包方式进行更换

  ```xml
  <packaging>pom</packaging>
  ```

* 创建web模块依赖父模块，并配置pom文件

## 配置前端页面

* 将css，js文件放在resource下的static文件夹

* html页面放在resource下的template文件夹下

* 添加themeleaf依赖

  ```xml
  <!--thymeleaf模板-->
          <dependency>
              <groupId>org.thymeleaf</groupId>
              <artifactId>thymeleaf-spring5</artifactId>
          </dependency>
          <dependency>
              <groupId>org.thymeleaf.extras</groupId>
              <artifactId>thymeleaf-extras-java8time</artifactId>
          </dependency>
  ```

  

* 新建config文件夹，创建类实现webMvcConfig实现url到html的映射

  ```java
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.servlet.config.annotation.EnableWebMvc;
  import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
  import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
  
  /**
   * @program: BigDataPlatform
   * @description:
   * @author: Hoodie_Willi
   * @create: 2020-02-03 12:07
   **/
  @Configuration
  public class MyMvcConfig implements WebMvcConfigurer {
      @Override
      public void addViewControllers(ViewControllerRegistry registry) {
          registry.addViewController("login").setViewName("pages-login");
          registry.addViewController("index").setViewName("index");
      }
  }
  ```

* 在html文件中使用thymeleaf接管前端页面，例如下面的样式

  ```html
  				<html lang="en" xmlns:th="http://www.thymeleaf.org">
  				<link th:href="@{/css/bootstrap.min.css}" rel="stylesheet" type="text/css">
          <link th:href="@{/css/icons.css}" rel="stylesheet" type="text/css">
          <link th:href="@{/css/style.css}" rel="stylesheet" type="text/css">
  ```


## 配置kafka

* 在kafka的配置文件夹下配置zookeeper地址和kafkalog的存放地址

  ```
  log.dirs=/Users/williwei/BigDataComponent/kafka/kafka_logs
  ```

* 启动kafka服务

  ```
  ./kafka-server-start.sh -daemon ../config/server.properties
  ```

* 创建一个topic进行测试

  ```
  bin ./kafka-topics.sh --zookeeper localhost:2181 --create --topic 20200202 --partitions 3 --replication-factor 1
  ```

## 代码进行生产者和消费者测试

* 在terminal中启动一个kafka-console-consumer.sh当作消费者

  ```
  bin ./kafka-console-consumer.sh --bootstrap-server localhost:2182 --topic raw_data
  ```

* 编写kafkaproducer进行测试

  ```java
  public class Producer {
              public static void main(String[] args) {
                  Properties props = new Properties();
                  props.put("bootstrap.servers", "localhost:9092");
                  props.put("acks", "all");
                  props.put("retries", 0);
                  props.put("batch.size", 16384);
                  props.put("linger.ms", 0);
                  props.put("buffer.memory", 33554432);
                  // key的序列化方式
                  props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
                  // value的序列化方式
                  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
                  KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
                  System.out.println("start send message....");
                  // 开始发送消息
                  for (int i = 0; i < 10; i++){
                      String key = "key = " + i;
                      String value = "value = " + i;
                      ProducerRecord<String, String> record = new ProducerRecord<String, String>("20200202", key, value);
                      try{
                          producer.send(record);
                          sleep(2000);
                          System.out.println("发送完毕");
                      }catch (Exception e){
                          e.printStackTrace();
                      }
                  }
  
      }
  }
  
  ```

## 在springboot项目中配置kafka

* 创建rtms-kafka模块

* 引入依赖

  ```XML
  <!-- springBoot集成kafka -->
  	  <dependency>
  			<groupId>org.springframework.kafka</groupId>
  			<artifactId>spring-kafka</artifactId>
  		</dependency>
  ```

* 创建kafka配置文件

  ```java
  @Configuration
  @EnableKafka
  public class KafkaConfig {
      // kafka中的生产者工厂和消费者工厂
      // 配置消费者的一些参数
  
      @Value("${spring.kafka.bootstrap-servers}")
      private String bootstrapServers;
  
      @Value("${spring.kafka.consumer.group-id}")
      private String consumerGroupId;
  
      @Value("${spring.kafka.consumer.auto-offset-reset}")
      private String autoOffsetReset;
  
      @Bean
      KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>> kafkaListenerContainerFactory() {
          ConcurrentKafkaListenerContainerFactory<Integer, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
          factory.setConsumerFactory(consumerFactory());
          factory.setConcurrency(3);
          factory.getContainerProperties().setPollTimeout(3000);
          return factory;
      }
  
      @Bean
      public ConsumerFactory<Integer, String> consumerFactory() {
          return new DefaultKafkaConsumerFactory<>(consumerConfigs());
      }
  
      // 配置kafka消费者的一些参数
      @Bean
      public Map<String, Object> consumerConfigs() {
          Map<String, Object> props = new HashMap<>();
          props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
          props.put(ConsumerConfig.GROUP_ID_CONFIG, consumerGroupId);
          props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
          props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
          props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
          return props;
      }
  }
  ```

* 在yaml文件中增加kafka的配置

  ```yml
  spring:
    # 关闭模版引擎的缓存
    thymeleaf:
      cache: false
    # 配置文件的真实位置
    messages:
      basename: i18n.login
    kafka:
      bootstrap-servers: localhost:9092
      consumer:
        group-id: test
        auto-offset-reset: earliest
  ```

* 写一个controller进行测试

  ```java
  @Controller
  public class SendKafkaDataController {
  
      @RequestMapping("/getKafkaData")
      public String testSendData() throws InterruptedException {
          for (int i = 0; i < 10; i++){
              WebSocketServer.sendInfo("发送kafka数据，这是第 " + i + " 条数据");
              Thread.sleep(2000);
          }
          System.out.println("发送数据完毕");
          return "websocket";
      }
  
      @KafkaListener(topics = {"topic20200213"}, groupId = "test")
      public void listen(String msg) throws InterruptedException {
          System.out.println("接收到了消息" + msg.toString() + new Date());
          Thread.sleep(5000);
      }
  
  }
  ```

  

* 在yaml文件中配置kafka地址

  ```
  
  ```

* 查看运行在zookeeper上的消费者组，同样对应的有查看运行在kafka自带zookeeper上的consumer-group

  ```
  ./kafka-consumer-groups.sh --zookeeper localhost:2181 --list
  ```

* ```
  kafka:
      # kafka服务器地址(可以多个)
      bootstrap-servers: localhost:9092
      consumer:
        # 指定一个默认的组名
        # earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
        # latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
        # none:topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
        auto-offset-reset: earliest
        # key/value的反序列化
        key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        group-id: console-consumer-13149
      producer:
        # key/value的序列化
        key-serializer: org.apache.kafka.common.serialization.StringSerializer
        value-serializer: org.apache.kafka.common.serialization.StringSerializer
        # 批量抓取
        batch-size: 65536
        # 缓存容量
        buffer-memory: 524288
        # 服务器地址
        bootstrap-servers: localhost:9092
  ```

* ```
  package com.willi.contorller;
  
  import org.apache.kafka.clients.consumer.ConsumerRecord;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.kafka.annotation.KafkaListener;
  import org.springframework.kafka.core.KafkaTemplate;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  /**
   * @program: BigDataPlatform
   * @description:从sparkstreaming清洗完写回kafka topic中拉取数据
   * @author: Hoodie_Willi
   * @create: 2020-02-02 12:40
   **/
  @RestController
  public class KafkaController {
      /**
       * 注入kafkaTemplate
       */
      @Autowired
      // private KafkaTemplate<String, String> kafkaTemplate;
      private KafkaTemplate<String, String> kafkaTemplate;
  
      /**
       * 发送消息的方法
       *
       * @param key
       *            推送数据的key
       * @param data
       *            推送数据的data
       */
      private void send(String key, String data) {
          // topic 名称 key data 消息数据
          kafkaTemplate.send("20200202", key, data);
  
      }
  
      // test 主题 1 my_test 3
  
      @RequestMapping("/kafka")
      public String testKafka() {
          int iMax = 6;
          for (int i = 1; i < iMax; i++) {
              send("key" + i, "data" + i);
          }
          return "success";
      }
  
  //    public static void main(String[] args) {
  //        SpringApplication.run(KafkaController.class, args);
  //    }
  
      /**
       * 消费者使用日志打印消息
       */
      @KafkaListener(topics = "20200202")
      public void receive(ConsumerRecord<?, ?> consumer) {
          System.out.println("topic名称:" + consumer.topic()
                  + ",key:" + consumer.key() + ",分区位置:" + consumer.partition()
                  + ", 下标" + consumer.offset());
      }
  
  }
  
  ```

* 

## 在springboot中配置websocket

* 引入依赖

  ```xml
  				<dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-websocket</artifactId>
          </dependency>
  ```

* 在config文件夹下创建一个WebSocketConfig文件

  ```java
  @Configuration
  public class WebSocketConfig {
      @Bean
      public ServerEndpointExporter serverEndpointExporter(){
          return new ServerEndpointExporter();
      }
  }
  ```
  
* 编写一个webSocketServer

  ```java
  import java.io.IOException;
  import java.util.Set;
  import java.util.concurrent.ConcurrentHashMap;
  
  import javax.websocket.OnClose;
  import javax.websocket.OnError;
  import javax.websocket.OnMessage;
  import javax.websocket.OnOpen;
  import javax.websocket.Session;
  import javax.websocket.server.ServerEndpoint;
  
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.stereotype.Component;
  /**
   * Type: WebSocketServer
   * Description: WebSocketServer,实现服务器客户端平等交流，
   * 		达到服务器可以主动向客户端发生消息
   * @author LYM
   * @date Dec 18, 2018
   */
  @ServerEndpoint(value = "/websocket")
  @Component
  public class WebSocketServer {
  
      //日志记录器
      private static final Logger LOGGER = LoggerFactory.getLogger(WebSocketServer.class);
  
      //使用道格李的ConcurrentHashSet, 放的是WebSocketServer而不是session为了复用自己方法
      private static transient volatile Set<WebSocketServer> webSocketSet = ConcurrentHashMap.newKeySet();
  
      //与某个客户端的连接会话，需要通过它来给客户端发送数据
      private Session session;
  
      /**
       * 连接建立成功调用的方法*/
      @OnOpen
      public void onOpen(Session session) {
          this.session = session;
          webSocketSet.add(this);     //加入set中
          sendMessage("连接成功");
      }
  
      /**
       * 连接关闭调用的方法
       */
      @OnClose
      public void onClose() {
          webSocketSet.remove(this);  //从set中删除
      }
  
      /**
       * 收到客户端消息后调用的方法
       * @param message 客户端发送过来的消息*/
      @OnMessage
      public void onMessage(String message, Session session) {
          LOGGER.info("来自客户端(" + session.getId() + ")的消息:" + message);
          sendMessage("Hello, nice to hear you! There are " + webSocketSet.size() + " users like you in total here!");
          System.out.println("接收到消息：" + message);
      }
  
      /**
       * Title: onError
       * Description: 发生错误时候回调函数
       * @param session
       * @param error
       */
      @OnError
      public void onError(Session session, Throwable error) {
          LOGGER.error("webSocket发生错误:" + error.getClass() + error.getMessage());
      }
  
      /**
       * Title: sendMessage
       * Description: 向客户端发送消息
       * @param message
       * @throws IOException
       */
      public boolean sendMessage(String message) {
          try {
              this.session.getBasicRemote().sendText(message);
              System.out.println("发送回信成功");
              return true;
          } catch (IOException error) {
              LOGGER.error("webSocket-sendMessage发生错误:" + error.getClass() + error.getMessage());
              return false;
          }
      }
  
  
      /**
       * 群发自定义消息
       * */
      public static void sendInfo(String message) {
          LOGGER.info("webSocket-sendInfo群发消息：" + message);
          for (WebSocketServer item : webSocketSet) {
              item.sendMessage(message);
          }
      }
  
      /**
       * Title: getOnlineCount
       * Description: 获取连接数
       * @return
       */
      public static int getOnlineCount() {
          return webSocketSet.size();
      }
  }
  ```

* 编写html测试页面

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>WebSocket测试</title>
      <meta charset="utf-8">
      <script th:src="@{/js/jquery.min.js}"></script>
  <!--    <script th:src="@{/js/sockjs.min.js}"></script>-->
      <script src="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js"></script>
  
  </head>
  <body>
  <!-----start-main---->
  <div class="main">
      <h2>socketTest</h2>
      <input type="button" id="send" value="点击向服务器发送消息">
      <p id="recive"></p>
  
  </div>
  <!-----//end-main---->
  </body>
  <script type="text/javascript">
      var ws = null;
      var ws_status = false;
      function openWebSocket(){
          //判断当前浏览器是否支持WebSocket
          if ('WebSocket' in window) {
              ws = new WebSocket("ws://"+window.location.host+"/websocket");
          } else if ('MozWebSocket' in window) {
              websocket = new MozWebSocket("ws://"+window.location.host+"/websocket");
          } else {
              ws = new SockJS("http://"+window.location.host+"/websocket");
          }
          ws.onopen = function () {
  
          };
          //这个事件是接受后端传过来的数据
          ws.onmessage = function (event) {
              //根据业务逻辑解析数据
              console.log("Server:");
              console.log(event);
          };
          ws.onclose = function (event) {
              console.log("Connection closed!");
          };
          ws.onopen = function (event){
              ws_status = true;
              console.log("Connected!");
          };
          ws.onerror = function (event){
              console.log("Connect error!");
          };
      }
      //如果连接失败，每隔两秒尝试重新连接
      setInterval(function(){
          if(!ws_status){
              openWebSocket();
          }
      }, 2000);
      $("#send").click(function(){
          ws.send("Hello, server, I am browser.");
      });
  </script>
  </html>
  ```


## 数据详情

```
id
name
host_id
host_name
neighbourhood_group
neighbourhood
latitude
longitude	
room_type	
price
minimum_nights
number_of_reviews
last_review
reviews_per_month
calculated_host_listings_count
availability_365
```

## 配置kafka生产者以及flinkETL

### 配置生产者



## 遇到问题

* websocket并发发送消息时要加锁

