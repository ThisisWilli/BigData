# Spark复习

## SparkCore

### Spark

* Spark是基于内存的计算框架
* 与MR的区别：
  * Spark是基于内存迭代计算，MR是基于磁盘迭代计算
  * Spark中有DAG有向无环图
  * MR中只有map和reduce，相当于Spark中的map和reducebykey，Spark中有各种算子应对

### Spark技术栈

* HDFS，MR，Yarn，Hive
*  SparkCore
* SparkSQL
* SparkStreaming

### Spark运行模式

* local：多用于本地测试，一般在idea中运行使用local模式
* Standalone：Spark自带的资源调度框架，支持分布式搭建
* Yarn：Hadoop生态圈中资源调度框架，Spark可以基于Yarn运行
* Mesos：资源调度框架

### Spark 核心RDD

* RDD：弹性分布式数据集
* RDD 内其实不存数据的partition也是不存数据的
* RDD五大特性
  * RDD是由partition组成
  * 算子(函数)作用在partition上
  * RDD之间有依赖关系
  * 分区器是作用在K，V格式的RDD上
  * partition对外提供最佳计算位置，利于数据处理的本地化
* 注意：
  * textfile读取HDFS文件的方法底层调用的是**MR读取HDFS文件的方法**，首先split，每个split对应一个block，每个split对应一个partition
  * 什么是K，V格式的RDD：RDD的数据是一个个的二元组
  * 哪里体现了RDD的弹性(容错)：1、RDD的分区可多可少，2、RDD之间有依赖关系
  * 哪里体现了RDD的分布式：RDD的partition是分布在多个节点上

### Spark代码流程

* `val conf = new SparkConf().setAppName...setMaster`
* `val sc = new SparkContext(conf)`
* 由sc得到RDD
* 对RDD使用transformation类算子转换
* 对RDD使用Action算子触发Transformations类算子执行
* `sc.stop()`

### Spark算子

* 转换算子：Transformation，懒执行，需要Action触发执行
  * filter
  * map
  * flatmap
  * mapToPair
  * sample
  * sortBy
  * sortByKey
  * reduceByKey
* 行动算子：Action，
  * 触发transformation类算子执行，一个application中有一个action算子就有一个job
  * foreach，count(结果会拿到Driver端)，collect，first，take
* 持久化算子

### 创建RDD方式

* textfile
* parallelize
* makeRDD

### 术语

* 资源层面：master->worker->executor->threadpool
* 任务层面：application->job->stage->tasks，task发送到threadpool中

### RDD的款窄依赖

* 宽依赖：父RDD与子RDD partition之间的关系是一对多的
* 窄依赖：父RDD与子RDD partition直接按的关系是多对一的

### Spark中job，stage，task之间的关系

* job时提交给Spark的任务
* stage时每个job处理过程要分为的几个阶段
* Task是每一个job处理过程要分几为几次任务。Task是任务运行的最小单位。最终是要以task为单位运行在executor中。
* 一个job任务可以有一个或多个stage，一个stage又可以有一个或多个task。所以一个job的task数量是  （stage数量 * task数量）的总和

### Stage

* 由一组并行的task组成
* **RDD不存数据**，partition中存的是逻辑
* RDD之间由宽依赖会划分stage
* Spark计算模式为pipeline管道计算模式
* 管道中的数据何时落地
  * shuffle write时
  * 对RDD进行持久化时
* Stage中的finalRDD partition个数决定
* 如何提高stage的并行度
  * 增大RDD partition的个数
  * reduceByKey：对相同key值的数据的value进行操作
  * distinct：对RDD的数据进行去重，但是数据出来可能是无序的
  * join：
  * repartition：可以增加或者减少分区，但是多用于增加分区

### Spark的资源调度和任务调度

#### 资源调度

* 1、集群启动，Worker向Master汇报资源，Master掌握集群资源
* 2、当代码`new SparkContext()`时，会创建DAG Scheduler和Task Scheduler
* 3、TaskScheduler向Master申请资源
* 4、Master找到满足资源的节点，启动Executor
* 5、Executor启动注册后，反向注册给Driver，Driver掌握了一批计算资源

#### 任务调度

* 6、action算子触发job，job中RDD之间的依赖关系形成DAG有向无环图
* 7、DAGscheduler按照RDD之间的款窄依赖关系，**切割每个job，划分stage**，将stage以TaskSet形成TaskScheduler
* 8、TaskScheduler遍历TaskSet，拿到一个个的Task，发送到Executor中的线程池中执行
* 9、TaskScheduler监控task，回收结果

#### 总结

* 1、Task如果发送失败，由TaskScheduler重试，如果重试3次失败之后，依然失败，那么由DAGScheduler重试stage，重试4次之后，如果失败，那么stage所在的job就失败，application就失败了

* 2、TaskScheduler不仅可以重试执行失败的task，还可以重试执行缓慢的task，这就是spark中的推测机制，默认是关闭的，对于ETL(extract transform load)业务，要关闭推测执行

* 3、如果在task执行过程中，发现某些task执行非常缓慢，

  * 1、是否有数据倾斜
  * 2、是否开启的推测执行

* ETL

  * 将非结构化数据，进行结构化

### 粗粒度资源和细粒度资源

#### 粗粒度资源申请
* 当application执行之前，首先将所有的资源申请完毕，如果申请不到一直等待，如果申请的到，执行application，task执行过程中就不需要自己申请资源，task执行快，application执行快
* 优点：application执行快
* 缺点：集群资源不能充分利用

#### 细粒度资源申请

* 当application执行之前，不会将所有的资源申请完毕，task执行时，自己申请资源，自己释放资源，task执行相对慢
* 优点：集群资源可以充分利用
* 缺点：application执行相对慢

### Spark Submit参数

* --master：指定提交模式
* --deploy-mode：client还是cluster模式
* --conf
* --name
* --jar：executor端依赖的一些jar包
* --files：executor依赖的一些文件
* --driver-cores
* --driver-memory
* --executor-cores
* --executor-memory
* --total-executor-cores
* --num-executors

### 源码分析

* master启动
* worker启动
* Spark Submit提交任务
  * Driver启动
  * Driver向Master注册Application
* Spark 资源调度
  * Executor在集群中是分散启动的，利于数据处理的稳定
  * 如果提交任务什么都不指定，集群中每台Worker为当前的application 启动一个Executor，这个Executor会使用当前节点的所有core和1G内存
  * 如果想要一台Worker上启动多个Executor，要指定--executor-cores
  * 提交任务指定 --total-executor-cores 会为当前application申请指定core个数的资源
  * 启动Executor不仅和core有关还和内存有关 --executor-memory
* Spark 任务调度
  * 从一个action算子开始，实现看自己写的业务逻辑

### 二次排序

* Spark中大于两列的排序都叫二次排序
* 封装对象，实现对象的排序，对象中的属性就是要排序的列(soryByKey)

### 分组取topN

* 原生的集合排序：有OOM风险
* 定长数组

### 广播变量

* 当Executor端使用到Driver端的变量时，如果不使用广播变量那么在每个Executor中有多少task，就会有多少变量副本
* 如果使用广播变量，在每个Executor中只有一份Driver端的变量副本，可以大大节省Executor端内存
* 注意点
  * 不能将RDD广播出去
  * 广播变量只能在Driver端定义，在Executor端使用，Executor端不能改变广播变量的值

### 累加器

* 相当于集群中的统筹变量
* 注意：累加器只能在Driver端定义，在Executor端使用，1.6版本不能在Executor中accumulator.value()获取累加器的值

### Spark WEBUI

* Spark-shell
  * Spark-Scala的REPL
  * 上传文件，Spark读取HDFS中文件
* 点击job->stage->task
* Jobs,Stages,Storage,Environment,SQL,Streaming

### 端口

* 50070：HDFS的web ui
* 9000：HDFS读写端口
* 8020：
* 2181：zookeeper端口
* 60010：Hbase
* 6379：redis
* 9083：hive中拿元数据
* 8080：Spark中master
* 8081：Spark中worker

### Spark History-Server

* 在客户端(node04)/Spark/conf/spark-defaults.conf中进行配置四个参数
* 在客户端spark/sbin/start-history-server.sh启动历史日志服务器
* 访问：node04:18080

### Master-HA

* 当提交任务启动Driver、向Master注册Application、申请Application资源 都要连接Master，如果Master不是Alive，就会失败

* Master高可用

  * 本地文件系统

  * zookeeper

    * 管理原数据
    * 自动选举
    * 搭建：
      * 1、在Master-Alive中./conf/spark-env.sh中配置四个参数
      * 2、找一台Master-Standyby配置./conf/spark-env.sh --SPARK_MASTER_HOST=node02
      * 3、启动zookeeper
      * 4、在Master-Alive中启动集群./start-all.sh
      * 5、测试切换

### Spark Shuffle

* 两种shuffleManager，一种是sortShuffleManager，1.6之后默认使用SortShuffleManager，2.0之后HashShuffleManager被丢弃

* HashShuffleManager

  * 普通机制

    * 产生磁盘小文件个数M*R
    * 流程
      * 1、map task处理完数据之后写往buffer缓存区，默认大小为32k，写往buffer缓冲区个数的与reduce task个数一致
      * 2、缓冲区满32k溢写磁盘，每个buffer缓存区对应一个磁盘小文件
      * 3、reduce端拉取数据
    * 问题：产生磁盘小文件多
      * shuffle write对象多
      * shuffle read对象多
      * 节点之间拉取数据的连接多，遇到网络连接不稳定导致拉取数据失败的概率大，会加大数据处理的时间

  * 优化机制
  * 产生磁盘小文件个数：C*R
    * 流程
      * 1、map task处理完数据之后写往buffer缓存区，默认大小为32k，写往buffer缓冲区个数的与reduce task个数一致
      * 2、同一个core中的task公用一份buffer缓存区
      * 3、缓冲区满32k溢写磁盘，每个buffer缓存区对应一个磁盘小文件
      * 4、reduce端拉取数据
    * 相对于普通情况，shuffle文件大大减少，当reduce task个数多，或者core的个数多的时候，产生磁盘小文件的个数还是比较大
  
  * SortShuffleManager
  * 普通机制
      * 产生磁盘小文件个数：2*M
      * 过程
        * 1、map task处理完数据先写往5M内存数据结构，默认有估算 机制，当估计内存不够
        * 2、如果内存能够申请到，继续往内存中写数据，如果申请不到，溢写磁盘，溢写时有排序，每批次是1w条溢写
        * 3、多次溢写的文件合并成两个文件，一个是索引文件，一个是数据文件
        * 4、reduce task拉取数据
    * bypass机制
      * 产生磁盘小文件个数：2*M
      * 过程
        * 与普通机制相比，溢写磁盘没有排序
      * 条件
        * Spark算子没有map端的combine聚合时，可以使用bypass机制，如果有map端combine 想使用bypass也不能使用
        * 开启bypass机制的条件：spark.shuffle.sort.bypassMergeThreshold，当reduce task小于这个参数时，默认开启bypass机制
  
###  Shuffle文件的寻址

* 对象

  * MapOutputTracker
    * MapOutputTrackerMaster-Driver
    * MapOutputTrackerMaster-Excutor
  * BlockManager
    * BlockManagerMaster-Driver
      * DiskStore:管理磁盘数据
      * MemoryStore：管理内存数据
      * BlockTansferService：负责拉取数据
    * BlockManagerSlaves-Executor
      * DiskStore：管理磁盘数据
      * MemoryStore：管理内存数据
      * BlockTransferService：负责拉取数据

* 过程

  * 1、map task处理完数据，将数据结果和落地的磁盘小文件的位置信息封装到MapStatus对象中，通过Worker的MapOutputTrackerWorker汇报给Driver中的MapOutputTrackerMaster，Driver掌握的磁盘小文件的位置信息
  * 2、reduce task拉取数据，首先向Driver要小磁盘文件的位置信息，Driver返回
  * 3、reduce端连接数据所在的节点，由BlockTransferService拉取数据
  * 4、BlockTransferService默认启动5个线程拉取数据，默认最多一次拉取48M
  * 5、拉取来的数据放在了Executor中的shuffle聚合内存中

* reduce OOM问题
  * 1、减少拉取数据量
  * 2、增大Executor端的整体内存
  * 3、增大Executor shuffle聚合内存的比例

### Spark内存管理

* 静态内存管理和统一内存管理 1.6之后引入的统一内存管理

* 使用哪种内存管理 选择参数

* 静态内存管理

  * 0.2：task运行
  * 0.2
    * 0.2：预留内存
    * 0.8：shuffle聚合内存
  * 0.6
    * 0.1：预留内存
    * 0.9
      * 0.2：反序列化
      * 0.8：RDD的缓存和广播变量

* 统一内存管理

  * 300M基础内存
  * 总-300M
    * 0.4(1.6版本-0.25)：task执行
    * 0.6(1.6版本-0.75)
      * 0.5：shuffle聚合内存
      * 0.5：RDD缓存和广播

  
### Shuffle调优

## SparkSQL

### SparkSQL

* 支持使用SQL查询分布式的数据
* Hive中写的hql，底层解析成MRjob
* SparkSQL发展过程：Hive->Shark->SparkSQL

### Shark与SparkSQL

* Shark中语法支持Hive中的语法
* SparkSQL的出现是的Spark脱离Hive，解耦，不在依赖于Hive的解析优化
* SparkSQL兼容所有Hive和Shark的语法
* SparkSQL支持查询原生的RDD，还可以将结果拿回当作RDD使用

### Spark on Hive-SparkSQL

* Spark：解析优化，执行引擎
* Hive：只是存储

### Hive on Spark

* Spark：执行引擎
* Hive：解析优化，存储

### DataFrame

* **Spark Core底层操作的是RDD，SparkSQL底层操作的就是DataFrame**
* DataFrame更像一张二维表格，有数据，也有列的信息
* **想用SQL查询分布式数据，必须创建出来DataFrame**，有了DataFrame就可以注册视图，通过视图查询
* 想要创建DataFrame，在Spark1.6中需要创建SQLContext，在Spark2.0+需要创建SparkSession

### 谓词下推，SparkSQL优化job，使用到了谓词下推
### SparkSQL 1.6与2.0之后版本的区别

* 1、Spark1.6中要创建SQLContext(SparkContext)，Spark2.0+使用的SparkSession
* 2、得到DataFrame之后注册临时表不一样，Spark1.6中是`df.registerTempTable("t1");`，Spark2.0+为`df.createOrReplaceTempView("t1");`，`df.createOrReplaceGlobalTempView("t2");`
* 3、Spark2.0+引入DataSet
  * 1、DataSet内部序列化机制与RDD不同，可以不用反序列化成对象调用
  * 2、DataSet是强类型的，默认列名是value，可以操作上的方法，比RDD多，RDD有的算子，DataSet中都有

### 创建DataFrame的方式

#### 读取json格式的文件

* 根据json的数据名自动成为列，列的类型会自动推断
* 读取json格式的文件，列会按照Ascii排序
* 读取json格式文件两种方式
  * `sparksession.read().json(...)`
  * `Session.read().format("json")`
* `df.show(num)`默认显示前20行数据
* 创建临时表的两种方式和区别`df.createOrReplaceTempView("t1");`，`df.createOrReplaceGlobalTempView("t2");`，前者可以跨Session
* 读取嵌套格式的json数据，使用列名、属性即可
* DataFrame结果拿回转化成RDD使用
* 读取jsonArray格式的数据，explode()函数，导入隐式转换

#### 读取json格式的RDD/DataSet

* Spark1.6中读取 json格式的RDD，Spark2.0以上只有读取json格式的DataSet

#### 读取RDD创建DataFrame

##### 1.反射的方式:

* 1.首先将RDD转换成自定义类型的RDD

* 2.rdd.toDF()

* Spark1.6中java:sqlContext.createDataFrame(personRDD, Person.class)

##### 2.动态创建Schema

* 1.创建row类型的RDD
* 2.使用spark.createDataFrame(rowRDD, structType)映射成DataFrame
* 注意:动态创建的ROW中的数据的顺序要与创建Schema的顺序一致

#### 读取parquet格式的数据加载DataFrame

* 与读取json格式的数据一样

#### 读取Mysql中的数据加载成DataFrame



#### 读取Hive中的数据加载DataFrame

* Spark1.6要使用HiveContext操作Hive数据
* Spark2.0以上,SparkSession将SQLContext和HiveContext相当于封装,但是要读取Hive中的数据要开启Hive支持enableHiveSupport()

### 保存DataFrame

* 将DataFrame文件保存为parquet文件:`df.write(SaveMode.Append).format("parquet").save("./data/parquet")`
* 将DataFrame保存到mysql表中

### 配置Spark on Hive

* 1.在客户端 ../conf/中创建hive-site.xml，让SparkSQL找到Hive原数据
* 2.在hive的服务端启动metaStore服务，：hive --service metastore
* 支持enableHiveSupport()，同时也要启动HDFS

### UDF

* user defined function，用户自定义函数
* java:

将普通RDD加载成DataFrame，1.通过动态创建schema，2.通过反射

### UDAF

* user defined aggregate function：用户自定义聚合函数
* count,sum,min，特点是多对一，`select name, count(*) from table group by name`**多对一之后必须group by**

## SparkStreaming







  


















​    

​    

​    





