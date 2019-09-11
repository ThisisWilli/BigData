# SparkCore

## Spark RDD

### 简介

![](pic\Spark核心RDD.jpg)

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

### Transformation转换算子

#### 概念

Transformations类算子是一类算子（函数）叫做转换算子，如map,flatMap,reduceByKey等。Transformations算子是延迟执行，也叫**懒加载执行**。

#### Transformation类算子

* filter：过滤符合条件的记录数，true保留，false过滤掉。
* map：将一个RDD中的每个数据项，通过map中的函数映射变为一个新的元素。特点：输入一条，输出一条数据。
* flatMap：先map后flat。与map类似，每个输入项可以映射为0到多个输出项。
* sample：随机抽样算子，根据传进去的小数按比例进行又放回或者无放回的抽样。
* reduceByKey：将相同的Key根据相应的逻辑进行处理。
* sortByKey/sortBy：作用在K,V格式的RDD上，对key进行升序或者降序排序。
* 特点：将RDD类型转化为RDD类型

### Action行动算子

#### 概念

Action类算子也是一类算子（函数）叫做行动算子，如foreach,collect，count等。Transformations类算子是延迟执行，Action类算子是触发执行。一个application应用程序中有几个Action类算子执行，就有几个job运行。

#### Action类算子

* count：返回数据集中的元素数。会在结果计算完成后回收到Driver端。
* take(n)：返回一个包含数据集前n个元素的集合。
* first：first=take(1),返回数据集中的第一个元素。
* foreach：循环遍历数据集中的每个元素，运行相应的逻辑。
* collect：将计算结果回收到Driver端。
* 特点：将RDD类型的数据转化为Long，Seq类型

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

* 可以手动指定持久化级别









