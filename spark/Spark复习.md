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
* 有点：集群资源可以充分利用
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



### 累加器



​    





