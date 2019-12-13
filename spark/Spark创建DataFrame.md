# Spark2.3中JavaAPI创建DataFrame的方式

## 通过反射的方式创建

### 注意

* **该方法不推荐使用**
* 自定义类要可序列化
* 自定义类的访问级别为Public
* RDD转成DataFrame后会根据映射将字段按Ascii排序
* 将DataFrame转换成RDD时获取字段两种方式,一种是df.getInt(0)下标获取（不推荐使用），另一种是df.getAs(“列名”)获取（推荐使用）

### 实现

* 数据如下所示

  ```
  Bob male 21
  Jack female 22
  Allen male 19
  ```

* 实现

  ```java
  import bean.Person;
  import org.apache.spark.SparkConf;
  import org.apache.spark.SparkContext;
  import org.apache.spark.api.java.JavaRDD;
  import org.apache.spark.api.java.JavaSparkContext;
  import org.apache.spark.api.java.function.Function;
  import org.apache.spark.api.java.function.VoidFunction;
  import org.apache.spark.rdd.RDD;
  import org.apache.spark.sql.Dataset;
  import org.apache.spark.sql.Row;
  import org.apache.spark.sql.SparkSession;
  import scala.Function1;
  
  /**
   * 通过反射的方式将非json格式的数据转化为RDD，其中SparkContext和SparkSession只能出现一个
   */
  public class readNonJsonRDDCreateDF {
      public static void main(String[] args) {
          SparkSession spark = SparkSession
                  .builder()
                  .appName("readNonJsonRDDCreateDF")
                  .master("local")
                  .getOrCreate();
          // JavaSparkContext sc = new JavaSparkContext();
          final Dataset<String> stringDataset = spark.read().textFile("data/person");
          //JavaRDD<String> stringRDD = sc.textFile("data/person", 1);
          JavaRDD<Person> mapRDD = stringDataset.toJavaRDD().map(new Function<String, Person>() {
              public Person call(String s) throws Exception {
                  Person person = new Person();
                  person.setName(s.split(" ")[0]);
                  person.setSex(s.split(" ")[1]);
                  person.setAge(s.split(" ")[2]);
                  return person;
              }
          });
          mapRDD.map(new Function<Person, String>() {
              public String call(Person person) throws Exception {
                  return person.toString();
              }
          }).foreach(new VoidFunction<String>() {
              public void call(String s) throws Exception {
                  System.out.println(s);
              }
          });
          // 利用反射机制生成df，即将RDD转换为Dataset
          Dataset<Row> df = spark.createDataFrame(mapRDD, Person.class);
          df.show();
  
          // 将DataSet转化为JavaRDD，转化为RDD再转化为Person含有Person类型的RDD
          JavaRDD<Row> transRDD = df.javaRDD();
          transRDD.map(new Function<Row, Person>() {
              public Person call(Row row) throws Exception {
                  Person person = new Person();
                  person.setName((String)row.getAs("name"));
                  person.setAge((String)row.getAs("age"));
                  person.setSex((String)row.getAs("sex"));
                  return person;
              }
          }).foreach(new VoidFunction<Person>() {
              public void call(Person person) throws Exception {
                  System.out.println(person.toString());
              }
          });
      }
  }
  ```

* 输出为

  ```
  Person{name='Bob', sex='male', age='21'}
  Person{name='Jack', sex='female', age='22'}
  Person{name='Allen', sex='male', age='19'}
  +---+-----+------+
  |age| name|   sex|
  +---+-----+------+
  | 21|  Bob|  male|
  | 22| Jack|female|
  | 19|Allen|  male|
  +---+-----+------+
  Person{name='Bob', sex='male', age='21'}
  Person{name='Jack', sex='female', age='22'}
  Person{name='Allen', sex='male', age='19'}
  ```

  

## 通过动态创建Schema将非json格式的RDD转换成DataFrame

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

import java.util.Arrays;
import java.util.List;


/**
 * \* project: SparkStudy
 * \* package: sql
 * \* author: Willi Wei
 * \* date: 2019-12-04 15:02:11
 * \* description:通过动态创建Schema的方式创建dataframe
 * \
 */

public class javaCreateSchema2DF {
    public static void main(String[] args) {
        SparkSession spark = SparkSession
                .builder()
                .master("local")
                .appName("javaCreateSchema2DF")
                .getOrCreate();
        // 读取文件并将原先读取的DataSet类型文件转化为JavaRDD类型
        JavaRDD<String> lineRDD = spark.read().textFile("data/person").toJavaRDD();
        JavaRDD<Row> rowRDD = lineRDD.map(new Function<String, Row>() {
            public Row call(String s) throws Exception {
                return RowFactory.create(
                        String.valueOf(s.split(" ")[0]),
                        String.valueOf(s.split(" ")[1]),
                        String.valueOf(s.split(" ")[2])
                );
            }
        });
        // 动态创建DataFrame中的元数据，元数据可以是自定义的字符串也可以来自外部数据库
        List <StructField> asList = Arrays.asList(
                DataTypes.createStructField("id", DataTypes.StringType, true),
                DataTypes.createStructField("sex", DataTypes.StringType, true),
                DataTypes.createStructField("age", DataTypes.StringType, true)
        );
        StructType schema = DataTypes.createStructType(asList);
        Dataset<Row> dataFrame = spark.createDataFrame(rowRDD, schema);
        dataFrame.show();
    }
}
```

输出为

```
+-----+------+---+
|   id|   sex|age|
+-----+------+---+
|  Bob|  male| 21|
| Jack|female| 22|
|Allen|  male| 19|
+-----+------+---+
```






