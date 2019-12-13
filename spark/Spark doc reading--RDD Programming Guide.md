# Spark doc reading--RDD Programming Guide

## 初始化Spark

​		编写Spark程序之前必须创建SparkContext，这样能让Spark知道如何访问集群，在创建SparkContext之前必须创建SparkConf，这其中包含了你的application中含有的信息
​		每个JVM只能激活一个SparkContext。 在创建新的SparkContext之前，您必须stop（）活动的SparkContext。

```scala
val conf = new SparkConf().setAppName(appName).setMaster(master)
new SparkContext(conf)
```

```java
SparkConf conf = new SparkConf().setAppName(appName).setMaster(master);
JavaSparkContext sc = new JavaSparkContext(conf);
```

​		appName参数是您的应用程序在群集UI上显示的名称。 **master是一个Spark，Mesos或YARN群集URL**，或一个特殊的“本地”字符串，以本地模式运行。 实际上，当在集群上运行时，您将不希望对程序中的母版进行硬编码，而是希望通过spark-submit启动应用程序并在其中接收它。 但是，对于本地测试和单元测试，您可以传递“ local”以在内部运行Spark。

## RDD

​		Spark围绕弹性分布式数据集（RDD）的概念展开，RDD是可并行操作的元素的容错集合(fault tolerant)。 创建RDD的方法有两种：并行化驱动程序中的现有集合，或引用外部存储系统（例如共享文件系统，HDFS，HBase或提供Hadoop InputFormat的任何数据源）中的数据集。

### Parallelized Collections

​		通过在驱动程序中的现有集合（Scala Seq）上调用SparkContext的parallelize方法来创建并行集合。 复制集合的元素以形成可以并行操作的分布式数据集。 例如，以下是创建包含数字1到5的并行化集合的方法：

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

```java
List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
JavaRDD<Integer> distData = sc.parallelize(data);
```

​		Spark会根据集群自动配置分区数

### External Datasets(读取外部数据集)

​		可以通过读取HDFS，本地数据集来获取数据，通过`sc.textFile("data.txt")`来创建，也可以填写url

```scala
scala> val distFile = sc.textFile("data.txt")
distFile: org.apache.spark.rdd.RDD[String] = data.txt MapPartitionsRDD[10] at textFile at <console>:26
```

```java
JavaRDD<String> distFile = sc.textFile("data.txt");
```

#### 读取文件时一些要注意的地方

* 如果使用路径或者本地文件系统，必须保证worknode上的节点也能用同样的路径来访问数据，可以将文件复制到所有datanode，或者使用网络共享文件系统
* textFile方法还带有一个可选的第二个参数，用于控制文件的分区数。 默认情况下，**Spark为文件的每个块创建一个分区（HDFS中的块默认为128MB）**，但是您也可以通过传递更大的值来请求更大数量的分区。 请注意，**分区(partition)不能少于块(block)。**补充： https://www.zhihu.com/question/37310539 

#### 读取文件时支持的其他格式

* SparkContext.wholeTextFiles使您可以读取包含多个小文本文件的目录，并将每个小文本文件作为（文件名，内容）对返回。 这与textFile相反，后者将在每个文件的每一行返回一条记录。 分区由数据局部性决定，在某些情况下，数据局部性可能导致分区太少。 在这种情况下，wholeTextFiles提供了一个可选的第二个参数来控制最小数量的分区。
* 对于SequenceFiles，请使用SparkContext的sequenceFile [K，V]方法，其中K和V是文件中键的类型和值。 这些应该是Hadoop的Writable接口的子类，例如IntWritable和Text。 此外，Spark允许您为一些常见的可写对象指定本机类型。 例如，sequenceFile [Int，String]将自动读取IntWritables和Texts。
* 对于其他Hadoop InputFormat，可以使用SparkContext.hadoopRDD方法，该方法采用任意JobConf和输入格式类，键类和值类。 使用与使用输入源进行Hadoop作业相同的方式设置这些内容。 您还可以基于“新” MapReduce API（org.apache.hadoop.mapreduce）将SparkContext.newAPIHadoopRDD用于InputFormats。
* RDD.saveAsObjectFile和SparkContext.objectFile支持以包含序列化Java对象的简单格式保存RDD。 尽管它不如Avro这样的专用格式有效，但它提供了一种保存任何RDD的简便方法。

### RDD操作

​		RDD支持两种类型的操作：transform（从现有操作创建新数据集）和action（在数据集上运行计算后将值返回到驱动程序）。 例如，map是一个转换，它将每个数据集元素通过一个函数传递，并返回代表结果的新RDD。 另一方面，reduce是使用某些函数聚合RDD的所有元素并将最终结果返回给驱动程序的操作（尽管也有并行的reduceByKey返回分布式数据集）。
​		Spark中的所有转换都是惰性的，因为它们不会立即计算出结果。 取而代之的是，他们只记得应用于某些基本数据集（例如文件）的转换。 仅当动作要求将结果返回给驱动程序时才计算转换。 这种设计使Spark可以更高效地运行。 例如，我们可以认识到通过map创建的数据集将用于reduce中，并且仅将reduce的结果返回给驱动程序，而不是将较大的maped数据集返回给驱动程序。
​		默认情况下，**每次在其上执行操作时，可能都会重新计算每个转换后的RDD**。 但是，您也可以使用persist（或缓存）方法将RDD保留在内存中，在这种情况下，Spark会将元素保留在集群中，以便下次查询时可以更快地进行访问。 还支持将RDD持久存储在磁盘上，或在多个节点之间复制。

#### 基本操作

```scala
val lines = sc.textFile("data.txt")
val lineLengths = lines.map(s => s.length)
val totalLength = lineLengths.reduce((a, b) => a + b)
```

```java
JavaRDD<String> lines = sc.textFile("data.txt");
JavaRDD<Integer> lineLengths = lines.map(s -> s.length());
int totalLength = lineLengths.reduce((a, b) -> a + b);
```








