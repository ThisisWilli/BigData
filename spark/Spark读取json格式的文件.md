# Spark读取json格式文件

## 准备工作

* 在pom.xml文件中引入依赖

  ```xml
          <dependency>
              <groupId>org.apache.spark</groupId>
              <artifactId>spark-sql_2.11</artifactId>
              <version>${spark.version}</version>
          </dependency>
  ```

## 使用scala和java读取json格式的文件

### 使用Scala读取json文件

```scala
object Test {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("test").master("local").getOrCreate()
    val df = spark.read.json("data/json")
    df.show()
    df.printSchema()
    val rows = df.take(10)
    rows.foreach(println)
  }
}
```

### 使用java读取json文件

```java
public class SQLTest {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local").setAppName("test");
        JavaSparkContext sc = new JavaSparkContext(conf);

        // 创建SQLContext
        SQLContext sqlContext = new SQLContext(sc);
        //SparkSession sparkSession = new SparkSession(sc);
        Dataset df = sqlContext.read().json("data/json");
        df.show();
        df.printSchema();

        df.select(df.col("name")).show();
        df.createOrReplaceTempView("t1");
        df.createOrReplaceGlobalTempView("t2");
        Dataset<Row> sql = sqlContext.sql("select * from t1");
        sql.show();

    }
}
```

### 读取复杂Json文件

* 嵌套json文件如下

  ```json
  {"name":"zhangsan","score":100, "infos":{"age":20, "gender":"man"}}
  {"name":"lisi","score":65, "infos":{"age":15, "gender":"female"}}
  {"name":"wangwu","score":100, "infos":{"age":20, "gender":"man"}}
  ```

* 读取文件

  ```scala
  object readNestJsonFile {
    def main(args: Array[String]): Unit = {
      val spark = SparkSession.builder().master("local").appName("readNestJsonFile").getOrCreate()
  
      // 读取嵌套的json文件
      val frame = spark.read.format("json").load("data/NetsJsonFile")
      frame.printSchema()
      frame.show()
  
      // 注册一张临时表
      frame.createOrReplaceTempView("infosView")
      spark.sql("select name, infos.age, score, infos.gender from infosView").show(100)
    }
  }
  ```

  输出为

  ```
  root
   |-- infos: struct (nullable = true)
   |    |-- age: long (nullable = true)
   |    |-- gender: string (nullable = true)
   |-- name: string (nullable = true)
   |-- score: long (nullable = true)
   +---------+--------+-----+
  |    infos|    name|score|
  +---------+--------+-----+
  |[20, man]|zhangsan|  100|
  |[20, man]|    lisi|  100|
  |[20, man]|zhangsan|  100|
  +---------+--------+-----+
  ```

  ### 读取嵌套的json文件

  * 文件如下

    ```json
    {"name":"zhangsan","age":21, "scores":[{"yuwen":98, "shuxue":90, "yingyue": 100}, {"yuwen":98, "shuxue":78, "yingyue": 100}]}
    {"name":"lisi","age":11, "scores":[{"yuwen":70, "shuxue":90, "yingyue": 60}, {"yuwen":55, "shuxue":78, "yingyue": 100}]}
    {"name":"wangwu","age":33, "scores":[{"yuwen":98, "shuxue":90, "yingyue": 100}, {"yuwen":98, "shuxue":78, "yingyue": 100}]}
    ```

  * 读取，读取之后再在select函数中编写sql语句进行读取

    ```scala
    object readJsonArray {
      def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder().master("local").appName("readNestJsonFile").getOrCreate()
    
        // 读取嵌套的json文件
        val frame = spark.read.format("json").load("data/jsonArrayFIle")
        // 不折叠显示
        frame.show(false)
        frame.printSchema()
    
        // 导入explode
        import org.apache.spark.sql.functions._
        // 导入这个可以使用$符
        import spark.implicits._
        // explode负责将json格式的数组展开，数组中每个json对象都是一条数据
        val transDF = frame.select($"name", $"age", explode($"scores")).toDF("name", "age", "allScores")
        transDF.show(100, false)
        transDF.printSchema()
      }
    }
    ```

  * 输出为

    ```
    +---+--------+------------------------------+
    |age|name    |scores                        |
    +---+--------+------------------------------+
    |21 |zhangsan|[[90, 100, 98], [78, 100, 98]]|
    |11 |lisi    |[[90, 60, 70], [78, 100, 55]] |
    |33 |wangwu  |[[90, 100, 98], [78, 100, 98]]|
    +---+--------+------------------------------+
    
    root
     |-- age: long (nullable = true)
     |-- name: string (nullable = true)
     |-- scores: array (nullable = true)
     |    |-- element: struct (containsNull = true)
     |    |    |-- shuxue: long (nullable = true)
     |    |    |-- yingyue: long (nullable = true)
     |    |    |-- yuwen: long (nullable = true)
     +--------+---+-------------+
    |name    |age|allScores    |
    +--------+---+-------------+
    |zhangsan|21 |[90, 100, 98]|
    |zhangsan|21 |[78, 100, 98]|
    |lisi    |11 |[90, 60, 70] |
    |lisi    |11 |[78, 100, 55]|
    |wangwu  |33 |[90, 100, 98]|
    |wangwu  |33 |[78, 100, 98]|
    +--------+---+-------------+
    
    root
     |-- name: string (nullable = true)
     |-- age: long (nullable = true)
     |-- allScores: struct (nullable = true)
     |    |-- shuxue: long (nullable = true)
     |    |-- yingyue: long (nullable = true)
     |    |-- yuwen: long (nullable = true)
    
    ```

    

### SparkSQL 1.6与2.0之后版本的区别

* 1、Spark1.6中要创建SQLContext(SparkContext)，Spark2.0+使用的SparkSession

* 2、得到DataFrame之后注册临时表不一样，Spark1.6中是`df.registerTempTable("t1");`，Spark2.0+为`df.createOrReplaceTempView("t1");`，`df.createOrReplaceGlobalTempView("t2");`

  