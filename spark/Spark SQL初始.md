# Spark SQL初始

## SparkSQL初始

* Hive是Shark的前身，Shark是SparkSQL的前身 
* 相对于Shark，SparkSQL有什么优势呢？
  * SparkSQL产生的根本原因，其完全脱离了Hive的限制 
  * SparkSQL支持查询原生的RDD，这点就极为关键了。RDD是Spark平台的核心概念，是   Spark能够高效的处理大数据的各种场景的基础 
  * 能够在Scala中写SQL语句。支持简单的SQL语法检查，能够在Scala中写Hive语句访问Hive数   据，并将结果取回作为RDD使用 
* 最进出来另个组合名称： 
    * Spark on Hive
      * Hive只是作为了存储的角色 
      * SparkSQL作为计算的角色 
      * Hive on Spark
        * Hive承担了一部分计算（解析SQL，优化SQL...）的和存储 
        * Spark作为了执行引擎的角色 

## Dataframe

​		与RDD类似，DataFrame也是一个分布式数据容器。然而DataFrame更像传统数据库的二维 表格，除了数据以外，还掌握数据的结构信息，即schema。同时，与Hive类似， DataFrame也支持嵌套数据类型（struct、array和map）。从API易用性的角度上 看， DataFrame API提供的是一套高层的关系操作，比函数式的RDD API要更加友好，门槛更低 

## RDD与DataFrame进行比较

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/RDD%E4%B8%8EDataFrame%E8%BF%9B%E8%A1%8C%E6%AF%94%E8%BE%83.png )

## Spark SQL底层架构

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SparkSQL%E5%BA%95%E5%B1%82%E6%9E%B6%E6%9E%84.png )

## DataFrame创建方式

* 读json文件 
  * sqlContext.read().format(“json”).load(“path”)
  * sqlContext.read().json(“path”) 
* 读取json格式的RDD •  RDD的元素类型是String，但是格式必须是JSON格式
* 读取parquet文件创建DF
*  –  RDD<String> 
  * 通过非json格式的RDD来创建出来一个DataFrame
    * 通过反射的方式
    * 动态创建schema的方式 

## SparkSQL 1.6与2.0之后版本的区别

* 1、Spark1.6中要创建SQLContext(SparkContext)，Spark2.0+使用的SparkSession

* 2、得到DataFrame之后注册临时表不一样，Spark1.6中是`df.registerTempTable("t1");`，Spark2.0+为`df.createOrReplaceTempView("t1");`，`df.createOrReplaceGlobalTempView("t2");`

### 创建DataFrame的方式

#### 读取json格式的文件

* 根据json的数据名自动成为列，列的类型会自动推断
* 读取json格式的文件，列会按照Ascii排序
* 读取json格式文件两种方式
  * `sparksession.read().json(...)`
  * `Session.read().format("json")`
* `df.show(num)`默认显示前20行数据
* 创建临时表的两种方式和区别`df.createOrReplaceTempView("t1");`，`df.createOrReplaceGlobalTempView("t2");`，前者可以跨Session