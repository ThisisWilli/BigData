# SparkSQL中的UDAF函数

用户自定义聚合函数

## 简介图

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/UDAF.jpg )

## 准备工作

* 创建MyUDAF类

  ```scala
  class MyUDAF extends UserDefinedAggregateFunction{
    // 输入数据的类型
    override def inputSchema: StructType = {
      DataTypes.createStructType(Array(DataTypes.createStructField("uuuu", StringType, true)))
    }
  
    // 聚合操作，所处理的数据的类型
    override def bufferSchema: StructType = {
      DataTypes.createStructType(Array(DataTypes.createStructField("xxxs", IntegerType, true)))
    }
  
    // 最终函数返回值的类型
    override def dataType: DataType = {
      DataTypes.IntegerType
    }
  
    // 多次运行 相同的输入总是相同的输出，确保一致性
    override def deterministic: Boolean = {
      true
    }
  
    //
    /**
     * 每个分组数据执行的初始化值，重要
     * 两个部分的初始化
     * 1.map端每个RDD分区内，按照groupby的字段，在RDD每个分区内按照groupby的字段分组，内有个初始化的值
     * 2.在reduce端 给每个group by的分组做初始值
     * @param buffer
     */
    override def initialize(buffer: MutableAggregationBuffer): Unit = {
      buffer(0) = 0
    }
  
    //每个组，有新的值进来的时候，进行分组对应的聚合值的计算，重要，RDD分区内部合并
    override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      buffer(0) = buffer.getAs[Int](0) + 1
    }
  
    // 最后merger的时候，在各个节点上的聚合值，要进行merge，也就是合并
    override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      // 计算每个分区中的值
      buffer1(0) = buffer1.getAs[Int](0) + buffer2.getAs[Int](0)
    }
  
    // 最后返回一个最终的聚合值要和dataType的类型一一对应
    override def evaluate(buffer: Row): Any = {
      buffer.getAs[Int](0)
    }
  }
  ```

## 实现UDAF

* scala实现

  ```scala
  object UDAF {
    def main(args: Array[String]): Unit = {
      val spark = SparkSession.builder().master("local").appName("UDAF").getOrCreate()
      val nameList = List[String]("zhangsan", "lisi", "wangwu", "zhangsan", "lisi", "wangwu")
      import spark.implicits._
      val frame = nameList.toDF("name")
      frame.createOrReplaceTempView("students")
  
      /**
       * 创建UDAF函数
       */
      spark.udf.register("NAMECOUNT", new MyUDAF())
      spark.sql("select name, NAMECOUNT(name) as count from students group by name").show()
    }
  }
  ```

  输出为

  ```
  +--------+-----+
  |    name|count|
  +--------+-----+
  |  wangwu|    2|
  |zhangsan|    2|
  |    lisi|    2|
  +--------+-----+
  ```

* Java实现

  ```java
  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.JavaRDD;
  import org.apache.spark.api.java.JavaSparkContext;
  import org.apache.spark.api.java.function.Function;
  import org.apache.spark.api.java.function.VoidFunction;
  import org.apache.spark.sql.*;
  import org.apache.spark.sql.expressions.MutableAggregationBuffer;
  import org.apache.spark.sql.expressions.UserDefinedAggregateFunction;
  import org.apache.spark.sql.types.DataType;
  import org.apache.spark.sql.types.DataTypes;
  import org.apache.spark.sql.types.StructField;
  import org.apache.spark.sql.types.StructType;
  import java.sql.Array;
  import java.util.ArrayList;
  import java.util.Arrays;
  import java.util.List;
  
  /**
   * \* project: SparkStudy
   * \* package: sql
   * \* author: Willi Wei
   * \* date: 2019-12-06 11:20:29
   * \* description:用javaAPI中用UDAF读取数据
   * \
   */
  
  /**
   * 1.首先通过sc.parallelize读取list数组，或通过textfile读取其他数据，获取rdd
   * 2.将rdd通过map算子转化成row类型的javaRDD
   * 3.通过DataTypes.createStructField("name", DataTypes.StringType, true)设置dataframe中的元数据类型，并创建DataTypes.createStructType(asList)
   * 4.最后通过spark.createDataFrame(rowRDD, schema);生成Dataset<Row> dataFrame
   */
  public class javaUDAF {
  
      public static void main(String[] args) {
          SparkSession spark = SparkSession
                  .builder()
                  .master("local")
                  .appName("javaUDAF")
                  .getOrCreate();
          JavaSparkContext sc = new JavaSparkContext(spark.sparkContext());
          JavaRDD<String> parallelize = sc.parallelize(
                  Arrays.asList("zhangsan","lisi","wangwu","zhangsan","zhangsan","lisi"));
          JavaRDD<Row> rowRDD = parallelize.map(new Function<String, Row>() {
  
              /**
               *
               */
              private static final long serialVersionUID = 1L;
              public Row call(String s) throws Exception {
                  return RowFactory.create(s);
              }
          });
  
          List<StructField> fields = Arrays.asList(DataTypes.createStructField("name", DataTypes.StringType, true));
          StructType schema = DataTypes.createStructType(fields);
          Dataset<Row> dataFrame = spark.createDataFrame(rowRDD, schema);
          dataFrame.registerTempTable("user");
          /**
           * 注册一个UDAF函数,实现统计相同值得个数
           * 注意：这里可以自定义一个类继承UserDefinedAggregateFunction类也是可以的
           */
          spark.udf().register("StringCount",new UserDefinedAggregateFunction() {
  
              /**
               *
               */
              private static final long serialVersionUID = 1L;
  
              /**
               * 初始化一个内部的自己定义的值,在Aggregate之前每组数据的初始化结果
               */
              @Override
              public void initialize(MutableAggregationBuffer buffer) {
                  buffer.update(0, 0);
              }
  
              /**
               * 更新 可以认为一个一个地将组内的字段值传递进来 实现拼接的逻辑
               * buffer.getInt(0)获取的是上一次聚合后的值
               * 相当于map端的combiner，combiner就是对每一个map task的处理结果进行一次小聚合
               * 大聚和发生在reduce端.
               * 这里即是:在进行聚合的时候，每当有新的值进来，对分组后的聚合如何进行计算
               */
              @Override
              public void update(MutableAggregationBuffer buffer, Row arg1) {
                  buffer.update(0, buffer.getInt(0)+1);
  
              }
              /**
               * 合并 update操作，可能是针对一个分组内的部分数据，在某个节点上发生的 但是可能一个分组内的数据，会分布在多个节点上处理
               * 此时就要用merge操作，将各个节点上分布式拼接好的串，合并起来
               * buffer1.getInt(0) : 大聚合的时候 上一次聚合后的值
               * buffer2.getInt(0) : 这次计算传入进来的update的结果
               * 这里即是：最后在分布式节点完成后需要进行全局级别的Merge操作
               * 也可以是一个节点里面的多个executor合并
               */
              @Override
              public void merge(MutableAggregationBuffer buffer1, Row buffer2) {
                  buffer1.update(0, buffer1.getInt(0) + buffer2.getInt(0));
              }
              /**
               * 在进行聚合操作的时候所要处理的数据的结果的类型
               */
              @Override
              public StructType bufferSchema() {
                  return DataTypes.createStructType(Arrays.asList(DataTypes.createStructField("bffer111", DataTypes.IntegerType, true)));
              }
              /**
               * 最后返回一个和DataType的类型要一致的类型，返回UDAF最后的计算结果
               */
              @Override
              public Object evaluate(Row row) {
                  return row.getInt(0);
              }
              /**
               * 指定UDAF函数计算后返回的结果类型
               */
              @Override
              public DataType dataType() {
                  return DataTypes.IntegerType;
              }
              /**
               * 指定输入字段的字段及类型
               */
              @Override
              public StructType inputSchema() {
                  return DataTypes.createStructType(Arrays.asList(DataTypes.createStructField("nameeee", DataTypes.StringType, true)));
              }
              /**
               * 确保一致性 一般用true,用以标记针对给定的一组输入，UDAF是否总是生成相同的结果。
               */
              @Override
              public boolean deterministic() {
                  return true;
              }
  
          });
  
          spark.sql("select name ,StringCount(name) as strCount from user group by name").show();
  
  
          sc.stop();
      }
  }
  ```

  