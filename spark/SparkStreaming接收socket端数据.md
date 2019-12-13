# SparkStreaming接受socket端数据

## 准备工作

* 启动一台node，并在node上安装nc包`[root@node04 ~]# nc -lk 9999`

## 实现

* 创建测试文件，如果只用`conf.setMaster("local")`，那么只会有一个线程在接受数据，边没有办法去处理数据，所以必须用`conf.setMaster("local[2]")`

  ```scala
  object StreamingTest {
    def main(args: Array[String]): Unit = {
      val conf = new SparkConf()
      conf.setMaster("local[2]")
      conf.setAppName("streamingTest")
  
      val ssc = new StreamingContext(conf, Durations.seconds(5))
      ssc.sparkContext.setLogLevel("Error")
      val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node04", 9999)
      val words = lines.flatMap(one=>{one.split(" ")})
      val pairWords: DStream[(String, Int)] = words.map(one=>{(one, 1)})
      val result: DStream[(String, Int)] = pairWords.reduceByKey((v1:Int, v2:Int)=>{v1 + v2})
      result.print(100)
  
      ssc.start()
      ssc.awaitTermination()
  
      ssc.stop()
    }
  }
  
  ```

* 在node中发送信息

  ```
  [root@node04 ~]# nc -lk 9999
  hello java
  heelo python
  ```

* 在webui中查看 http://localhost:4040

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SparkWebui%E4%B8%AD%E6%9F%A5%E7%9C%8Bsparkstreaming%E4%BB%BB%E5%8A%A1.png )

