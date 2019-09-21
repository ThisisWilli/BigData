# SparkCore

## Spark RDD

### 简介

![](pic\Spark核心RDD.png)

* RDD之间的依赖关系可以让他们相互还原
* 分区器难理解
* 第5点是数据在哪个partition上就把task发送到哪个节点上，否则要把数据拉到task发送到的分区上
* 弹性也叫容错
* 体现弹性是指启动多个task并行处理数据时可能会出现计算恒宇

## Spark任务执行原理

![](pic\Spark任务执行原理.png)

Driver和Worker是启动在节点上的进程，运行在JVM中的进程。

* Driver与集群节点之间有频繁的通信。
* Driver负责任务(tasks)的分发和结果的回收。任务的调度。如果task的计算结果非常大就不要回收了。会造成oom
* Worker是Standalone资源调度框架里面资源管理的从节点。也是JVM进程。
* Master是Standalone资源调度框架里面资源管理的主节点。也是JVM进程。
* 图上少画一个管理worker的master节点,也是一个JVM进程

## Spark代码流程

* 创建SparkConf对象
  * 可以设置Application name。(`conf.setAppName("test")`)
  * 可以设置运行模式及资源需求。
* 创建SparkContext对象
* 基于Spark的上下文创建一个RDD，对RDD进行处理。
* 应用程序中要有Action类算子来触发Transformation类算子执行。
* 关闭Spark上下文对象SparkContext。

## Spark中的算子

Spark算子大方向来说可以分为以下两类transformation算子(也叫**懒加载执行**)和action算子，从小方向细分来说，可以分成以下三类

* Value数据类型的transformation算子，这种变换并不触发提交作业，针对处理的数据项是value型的数据
* key-value数据类型的transformation算子，这种变换并不触发提交作业，针对处理的数据项是key-value型的数据对
* action算子，这类算子会触发SparkContext提交Job作业

### Value数据类型的Transformation算子

#### 输入分区与输出分区一对一类型

* map：将一个RDD中的每个数据项，通过map中的函数映射变为一个新的元素。特点：输入一条，输出一条数据。

  ![](pic\map算子示意图.png)
  
* flatMap：将原来 RDD 中的每个元素通过函数 f 转换为新的元素，并将生成的 RDD 的每个集合中的元素合并为一个集合，内部创建 FlatMappedRDD(this，sc.clean(f))        图 2表示RDD的一个分区 ，进 行 flatMap函 数 操 作，flatMap 中 传 入 的 函 数 为 f:T->U， T和 U 可以是任意的数据类型。将分区中的数据通过用户自定义函数 f 转换为新的数据。**外部大方框可以认为是一个 RDD 分区**，**小方框代表一个集合**。 V1、 V2、 V3 在一个集合作为 RDD 的一个数据项，可能存储为数组或其他容器，转换为V’1、 V’2、 V’3 后，**将原来的数组或容器结合拆散**，拆散的数据形成为 RDD 中的数据项

  ![](pic\flatmap算子示意图.png)
  
* mapPartitions：mapPartitions 函数获取到**每个分区的迭代器**，在函数中通过这个分区整体的迭代器对整个分区的元 素进行操作。内部实现是生 成MapPartitionsRDD。图中方框表示一个RDD分区

  ![](pic\mapPartition算子示意图.png)
  
* glom：将每个分区形成一个数组，内部实现是返回的GlommedRDD

  ![](pic\glom算子示意图.png)

#### 输入分区与输出分区多对一型

* union：**需要保证两个RDD元素的数据类型相同**，返回的RDD数据类型和被合并的RDD数据元素类型相同，**并不进行去重操作**，保存所有元素，想去重可以使用distinct()，**使用++符号相当于union函数操作**左侧大方框代表两个 RDD，大方框内的小方框代表 RDD 的分区。右侧大方框代表合并后的 RDD，大方框内的小方框代表分区。含有V1、V2、U1、U2、U3、U4的RDD和含有V1、V8、U5、U6、U7、U8的RDD合并所有元素形成一个RDD。V1、V1、V2、V8形成一个分区，U1、U2、U3、U4、U5、U6、U7、U8形成一个分区。

  ![](pic\笛卡尔运算原理.png)

  ![](pic\union算子示意图.png)

* cartesian:对两个RDD内中的所有元素进行笛卡尔积操作，操作后，内部实现返回CartesianRDD。图中左侧两个方框代表两个RDD，大方框内的小方框代表RDD分区，右侧大方框代表合并后的RDD，方框内的小方框代表分区

  ![](pic\cartesian算子示意图.png)

#### 输入分区与输出分区多对多型

* groupby：将元素通过函数生成相应的key，数据就转化成key-value格式，之后将key相同的元素分为一组，函数实现如下：

  (1)将用户函数预处理`val cleanF = sc.clean(f)`

  (2)对数据map进行函数操作，最后在进行groupbykey分组操作`this.map(t => (cleanF(t), t)).groupByKey(p)`

  ![](pic\groupby算子示意图.png)

#### 输出分区为输入分区子集型

* filter：过滤符合条件的记录数，true保留，false过滤掉。

  ![](pic\sample算子示意图.png)
  
* distinct：distinct将RDD中的元素进行去重操作，

  ![](pic\distinct算子示意图.png)

* subtract：subtract相当于集合的差操作，RDD1去除RDD1和RDD2交集中的所有元素

  ![](pic\subtract算子示意图.png)

* sample：随机抽样算子，根据传进去的小数按比例进行又放回或者无放回的抽样。

  ![](pic\sample算子示意图.png)

* takeSample:按照采样个数进行采样，返回结果不再是RDD，而是相当于对采样后的数据进行collect()，返回结果的集合为单机的数组

  ![](pic\takeSample算子示意图.png)

#### Cache型

* Cache算子：cache 将 RDD 元素从磁盘缓存到内存。 相当于 persist(MEMORY_ONLY) 函数的功能。

  ![](pic\cache算子的示意图.png)

* persist算子

### Key-Value数据类型的Transformation算子

#### 输入分区与输出分区一对一

* mapValues算子：针对(Key, Value)型数据中的Value进行Map操作，**而不对Key进行处理**，a=>a+2 代表对 (V1,1) 这样的 Key Value 数据对，数据只对 Value 中的 1 进行加 2 操作，返回结果为 3

  ![](pic\mapValue算子示意图.png)

#### 对单个RDD或两个RDD聚集

* combineByKey：相当于将元素为 (Int， Int) 的 RDD 转变为了 (Int， Seq[Int]) 类型元素的 RDD。图 16中的方框代表 RDD 分区。如图，通过 combineByKey， 将 (V1,2)， (V1,1)数据合并为（ V1,Seq(2,1)）。

  ![](pic\combineByKey算子示意图.png)

* reduceByKey：将相同的Key根据相应的逻辑进行处理，比如说将两个值合并成一个值

  ![](pic\reduceByKey算子示意图.png)

* partitionBy：是对**RDD进行分区操作**

  ![](pic\partitionBy算子示意图.png)

* Cogroup：将两个RDD进行协同划分，对在两个RDD中的Key-value类型的元素，每个RDD相同key的元素分别聚合成一个集合，并且返回两个RDD中对应Key的元素集合的迭代。图中大方框代表RDD，小方框代表RDD中的分区，将RDD1中的数据和RDD2中的数据进行合并

  ![](pic\Cogroup算子示意图.png)

#### 连接

* join：join对两个需要连接的RDD进行cogroup函数操作，将**相同key的数据**能够放到同一个分区，在coGroup操作之后形成的新RDD对每个key下的元素进行**笛卡尔积的操作**，返回的结果再展平，对应key下的所有元组形成一个集合。最后返回RDD

  ![](pic\join算子示意图.png)

* leftOutJoin和rightOutJoin：LeftOutJoin（左外连接）和RightOutJoin（右外连接）相当于在join的基础上**先判断一侧的RDD元素是否为空**，如果为空，则填充为空。 如果不为空，则将数据进行连接运算，并返回结果。

* sortByKey/sortBy：作用在K,V格式的RDD上，对key进行升序或者降序排序。

* 特点：将RDD类型转化为RDD类型

### Action行动算子

#### 概念

Action类算子也是一类算子（函数）叫做行动算子，如foreach,collect，count等。Transformations类算子是延迟执行，Action类算子是触发执行。一个application应用程序中有几个Action类算子执行，就有几个job运行。

#### 无输出

* foreach：循环遍历数据集中的每个元素，运行相应的逻辑。

  ![](pic\foreach算子示意图.png)

#### HDFS

* saveAsTextFile：函数将数据输出，存储到HDFS的指定目录中
* saveAsObjectFile：将分区中的每10个元素组成一个Array，然后将Array序列化，映射成(Null，BytesWritable(Y)，就是将RDD的每个分区存储为HDFS上的一个Block

#### Scala集合和数据类型

* collect：将计算结果回收到Driver端。，已经过时不推荐使用

* collectAsMap：collectAsMap对(K，V)型的RDD数据**返回一个单机HashMap**。 对于重复K的RDD元素，后面的元素覆盖前面的元素。

  ![](pic\collectAsMap算子示意图.png)

* reduceByKeyLocally：实现的是先reduce再collectAsMap的功能，先对RDD的整体进行reduce操作，然后收集所有结果返回一个HashMap

* count：返回数据集中的元素数。会在结果计算完成后回收到Driver端。

* take(n)：返回一个包含数据集前n个元素的集合。

* first：first=take(1),返回数据集中的第一个元素。

* reduce：先对两个元素<K,V>进行reduce函数操作，然后将结果和迭代器取出的下一个元素<K, V>进行reduce函数操作，知道迭代器遍历完所有的元素，得到最后结果。在RDD中，先对每个分区中的所有元素<K，V>的集合分别进行reduceLeft。 每个分区形成的结果相当于一个元素<K，V>，再对这个结果集合进行reduceleft操作。

### Spark中的Scala算子实践

* map算子:拿一条数据，出一条数据

  ```scala
  import org.apache.spark.{SparkConf, SparkContext}
  
  object Operate {
    def main(args: Array[String]) = {
      val conf = new SparkConf()
      conf.setMaster("local")
      conf.setAppName("test")
      val sc = new SparkContext(conf)
      sc.setLogLevel("Error")
      val lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data.txt")
       lines.map(one=>{
        one + "#"
      }).foreach(println)
    }
  }
  ```

* flatmap算子：一对多

  ```scala
  import org.apache.spark.{SparkConf, SparkContext}
  
  object Operate {
    def main(args: Array[String]) = {
      val conf = new SparkConf()
      conf.setMaster("local")
      conf.setAppName("test")
      val sc = new SparkContext(conf)
      sc.setLogLevel("Error")
      val lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data.txt")
      lines.flatMap(one=>{one.split(" ")}).foreach(println)
    }
  }
  ```

* filter算子

  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}
  
  object Operate {
    def main(args: Array[String]) = {
      val conf = new SparkConf()
      conf.setMaster("local")
      conf.setAppName("test")
      val sc = new SparkContext(conf)
      sc.setLogLevel("Error")
      val lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data.txt")
      val rdd1 = lines.flatMap(one=>{one.split(" ")})
      rdd1.filter(one=>{
        "hello".equals(one)
      }).foreach(println)
    }
  }
  ```

* reduceByKey:

  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}
  
  object Operate {
    def main(args: Array[String]) = {
      val conf = new SparkConf()
      conf.setMaster("local")
      conf.setAppName("test")
      val sc = new SparkContext(conf)
      sc.setLogLevel("Error")
      val lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data.txt")
      val words = lines.flatMap(one=>{one.split(" ")})
  
      val pairWords = words.map(one=>{(one, 1)})
  //    //val reduceResult =  pairWords.reduceByKey((v1:Int, v2:Int)=>{v1 + v2})
      val reduceResult :RDD[(String, Int)] = pairWords.reduceByKey((v1:Int, v2:Int) => (v1 + v2))
      reduceResult.foreach(println)
      }
  }
  ```

### Spark中的Java算子

map和flatmap的区别

![](pic\map和flatmap的区别.png)

* filter算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          JavaRDD<String> result = lines.filter(new Function<String, Boolean>() {
              public Boolean call(String line) throws Exception {
                  return "hello spark".equals(line);
              }
          });
          long count = result.count();
          System.out.println(count);
          result.foreach(new VoidFunction<String>() {
              public void call(String s) throws Exception {
                  System.out.println(s);
              }
          });
  		sc.stop();
      }
  }
  ```

* map算子：

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          JavaRDD<String> map = lines.map(new Function<String, String>() {
              public String call(String line) throws Exception {
                  return line + "*";
              }
          });
          map.foreach(new VoidFunction<String>() {
              public void call(String s) throws Exception {
                  System.out.println(s);
              }
          });
          sc.stop();
      }
  }
  ```

* mapToPair算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          JavaPairRDD<String, String> result = lines.mapToPair(new PairFunction<String, String, String>() {
              //第一个String对应call中的String，后面两个String分别对应k，v格式的String
              public Tuple2<String, String> call(String s) throws Exception {
                  return new Tuple2<String, String>(s, s + "#");
              }
          });
          result.foreach(new VoidFunction<Tuple2<String, String>>() {
              public void call(Tuple2<String, String> stringStringTuple2) throws Exception {
                  System.out.println(stringStringTuple2);
              }
          });
          sc.stop();
      }
  }
  ```

  ```scheme
  (hello java,hello java#)
  (hello spark,hello spark#)
  (hello python,hello python#)
  (hello c#,hello c##)
  ```

* flatmap算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          lines.flatMap(new FlatMapFunction<String, String>() {
              public Iterator<String> call(String s) throws Exception {
                  List<String> list = Arrays.asList(s.split(" "));
                  return list.iterator();
              }
          }).foreach(new VoidFunction<String>() {
              public void call(String s) throws Exception {
                  System.out.println(s   );
              }
          });
        }
  }
  ```

  ```
  hello
  java
  hello
  spark
  hello
  python
  hello
  c#
  hello
  spark
  hello
  spark
  hello
  spark
  hello
  spark
  hello
  spark
  ```

* reduceByKey

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          lines.flatMap(new FlatMapFunction<String, String>() {
              public Iterator<String> call(String s) throws Exception {
                  List<String> list = Arrays.asList(s.split(" "));
                  return list.iterator();
              }
          }).mapToPair(new PairFunction<String, String, Integer>() {
              public Tuple2<String, Integer> call(String word) throws Exception {
                  return new Tuple2<String, Integer>(word, 1);
              }
          }).reduceByKey(new Function2<Integer, Integer, Integer>() {
              // 前两个对kv中的参数，第三个对最后一个的参数
              public Integer call(Integer v1, Integer v2) throws Exception {
                  return v1 + v2;
              }
          }).foreach(new VoidFunction<Tuple2<String, Integer>>() {
              public void call(Tuple2<String, Integer> tp) throws Exception {
                  System.out.println(tp);
              }
          });
       }
   }
  ```

  ```
  (c#,1)
  (spark,43)
  (python,13)
  (hello,66)
  (java,9)
  ```

* 按单词出现次数进行降序排序

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          JavaPairRDD<String, Integer> reduceRDD = lines.flatMap(new FlatMapFunction<String, String>() {
              public Iterator<String> call(String s) throws Exception {
                  List<String> list = Arrays.asList(s.split(" "));
                  return list.iterator();
              }
          }).mapToPair(new PairFunction<String, String, Integer>() {
              public Tuple2<String, Integer> call(String word) throws Exception {
                  return new Tuple2<String, Integer>(word, 1);
              }
          }).reduceByKey(new Function2<Integer, Integer, Integer>() {
              // 前两个对kv中的参数，第三个对最后一个的参数
              public Integer call(Integer v1, Integer v2) throws Exception {
                  return v1 + v2;
              }
          });
          reduceRDD.mapToPair(new PairFunction<Tuple2<String, Integer>, Integer, String>() {
              public Tuple2<Integer, String> call(Tuple2<String, Integer> stringIntegerTuple2) throws Exception {
                  return stringIntegerTuple2.swap();
              }
          }).sortByKey(false).mapToPair(new PairFunction<Tuple2<Integer, String>, String, Integer>() {
              public Tuple2<String, Integer> call(Tuple2<Integer, String> integerStringTuple2) throws Exception {
                  return integerStringTuple2.swap();
              }
          }).foreach(new VoidFunction<Tuple2<String, Integer>>() {
              public void call(Tuple2<String, Integer> tp) throws Exception {
                  System.out.println(tp);
              }
          });
       }
  }
  ```

  ```
  (hello,66)
  (spark,43)
  (python,13)
  (java,9)
  (c#,1)
  ```

* collect算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          List<String> collect = lines.collect();
          for (String one : collect){
              System.out.println(one);
          }
      }
  }
  ```

  ```
  hello java
  hello spark
  hello python
  hello c#
  hello spark
  hello spark
  hello spark
  hello spark
  hello spark
  hello spark
  hello spark
  ...
  ```

* count算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          List<String> collect = lines.collect();
          long count = lines.count();
          System.out.println(count);
      }
  }
  ```

* take算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          List<String> take = lines.take(3);
          for (String one : take){
              System.out.println(one);
          }
      }
  }
  ```

* sample 算子

  ```java
  public class JavaOperate {
      public static void main(String[] args) {
          SparkConf conf = new SparkConf();
          conf.setMaster("local");
          conf.setAppName("test");
          JavaSparkContext sc = new JavaSparkContext(conf);
          JavaRDD<String> lines = sc.textFile("D:\\BigDataProj\\Spark\\SparkWC\\data2.txt");
          // 也可以大于1， 尽量不要大于1
          JavaRDD<String> sample = lines.sample(true, 0.1); 
          sample.foreach(new VoidFunction<String>() {
              public void call(String s) throws Exception {
                  System.out.println(s);
              }
          });
      }
  }
  ```

### Spark中的持久化算子

持久化的单位是partition

#### cache

* 默认将数据存储在内存中

* `cache() = persist() = persist(StorageLevel.MEMORY_ONLY)`

  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}
  
  object CacheTest {
    def main(args: Array[String]): Unit = {
      val conf = new SparkConf().setMaster("local").setAppName("cache")
      val sc = new SparkContext(conf)
      var rdd:RDD[String] = sc.textFile("./data/persistData.txt")
      rdd = rdd.cache()
      var startTime1: Long = System.currentTimeMillis()
      // 由于cache是懒执行，所有第一次count时，是从磁盘中找数据
      val result1 = rdd.count()
      var endTime1: Long = System.currentTimeMillis()
      println(s"在磁盘中读=$result1,time=${endTime1 - startTime1}ms")
  
      var startTime2: Long = System.currentTimeMillis()
      // 第二次找数据时，数据已经通过cache()放入内存中，直接从内存中读取数据
      val result2 = rdd.count()
      var endTime2: Long = System.currentTimeMillis()
      println(s"在内存中读=$result2,time=${endTime2 - startTime2}ms")
  
      sc.stop()
    }
  }
  ```

#### persist

* **可以手动指定持久化级别**
* MEMORY_ONLY
* MEMORY_ONLY_SER ：序列化
* MEMORY_AND_DISK
* MEMORY_AND_DISK_SER
* "_2"是由副本
* 尽量少使用DISK_ONLY级别

#### cache和persist的注意

* cache，persist，checkpoint都是懒执行，最小持久化单位是partition
* cache和persist之后可以直接赋值给一个值，下次直接使用找个值就是使用持久化的数据
* 如果采用第二种方式，后面不能紧跟action算子
* cache和persist的数据，当application执行完成之后会自动清除

#### checkpoint

* 将数据直接持久化到指定的目录，当lineage(计算逻辑)非常复杂，可以尝试使用checkpoint，checkpoint还可以切断RDD的关系，相当于在多个连续，连接的RDD之间添加一个checkpoint，这些checkpoint中的数据在计算完成之后会重新进行计算，然后重新扔回到checkpoint节点中

* 特殊场景使用checkpoint，**对RDD使用checkpoint要慎重**，因为要放进磁盘

* checkpoint要指定目录，可以将数据持久化到指定目录中，**当application执行完成之后，这个目录中的数据不会被清除** 

* checkpoint执行流程
  * 当sparkjob执行完成之后，spark会**从后往前**回溯，找到checkpointRDD做标记
  * 当回溯完成之后，Spark框架会重新启动一个job，计算标记的RDD的数据，放入指定的checkpoint中
  * 数据计算完成之后，放入目录之后，会切断RDD之间的依赖关系，当SparkApplication执行完成之后，数据目录中的数据不会被清除
  * 优化：对哪个RDD进行checkpoint，最好先cache一下，这样回溯完成之后再计算这个checkpointRDD数据的时候可以直接在内存中拿到放到指定的目录中
  
  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.storage.StorageLevel
  import org.apache.spark.{SparkConf, SparkContext}
  object CacheTest {
    def main(args: Array[String]): Unit = {
      val conf = new SparkConf().setMaster("local").setAppName("cache")
      val sc = new SparkContext(conf)
      sc.setLogLevel("Error")
      sc.setCheckpointDir("./data/ck")
      //var rdd: RDD[String] = sc.textFile("D:\\persistData.txt")
      var rdd: RDD[String] = sc.textFile("./data/data2.txt")
      rdd.cache()
      rdd.checkpoint()
      rdd.count()
       sc.stop()
    }
   }
  ```
  
  











