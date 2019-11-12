# Spark中广播变量和累加器

## 广播变量

### 示意图

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E5%B9%BF%E6%92%AD%E5%8F%98%E9%87%8F.png )

* 每个Executor中的blockManager负责管理broadCastList

* 广播变量的好处，不是每个task一份变量副本，而是变成每个节点的executor才一份副本。这样的话，

* task发送过来中如何Executor中的broadCastList中含有需要的变量，那么就不需要携带变量到Executor

* 不能将RDD广播出去(因为RDD是不存储数据的)，可以将RDD的结果广播出去，rdd.collect()

* 广播变量只能在Driver定义，在Driver端可以修改广播变量的值，在Executor中使用，不能在Executor中改变广播变量

* 如果executor端用到了Driver的变量，如果不使用广播变量在Executor有多少task就有多少Driver端的变量副本。

* 如果Executor端用到了Driver的变量，如果使用广播变量在每个Executor中只有一份Driver端的变量副本。

  ```scala
  object broadCastTest {
    def main(args: Array[String]): Unit = {
      val conf = new SparkConf()
      conf.setMaster("local").setAppName("test")
      val sc = new SparkContext(conf)
      sc.setLogLevel("Error")
      val list = List[String]("zhangsan", "lisi")
      val bcList: Broadcast[List[String]] = sc.broadcast(list)
      val nameRDD = sc.parallelize(List[String]("zhangsan", "lisi", "wangwu"))
      val result = nameRDD.filter(name => {
        // 拿回来之后的广播变量，还原了List
        val innerList: List[String] = bcList.value
        // !list.contains(name)
        // 这样使用的是广播变量，内存也能减少很多
        !innerList.contains(name)
      })
      result.foreach(println)
    }
  }
  ```

## 累加器

### 示意图

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E7%B4%AF%E8%AE%A1%E5%99%A8.png )

```scala
object AccumulatorTest {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local").setAppName("test")
    val sc = new SparkContext(conf)
    sc.setLogLevel("Error")
    val accumulator = sc.longAccumulator
    var i = 0
    val rdd1: RDD[String] = sc.textFile("data/data2.txt")
    val rdd2: RDD[String] = rdd1.map(one => {
//      i += 1
//      println(s"Executor i = $i")
      accumulator.add(1)
      // 两种方法打印都可以
      // println(s"Executor accumulator = ${accumulator}")
      println(s"Executor accumulator = ${accumulator.value}")
      one
    })
    rdd2.collect()
    // println(s"i = $i")
    println(s"accumulator = ${accumulator}")
  }
}
```



  