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



