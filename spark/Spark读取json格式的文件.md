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

## 实践

* 创建测试类

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

  