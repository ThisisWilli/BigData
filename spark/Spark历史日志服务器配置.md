# Spark历史日志服务器配置

## 配置存放历史日志的路径

* 在客户端中`[root@node04 spark-2.3.1]# cd conf/`

* `[root@node04 conf]# vim spark-defaults.conf`，进行配置，**注意hdfs的端口**，如果存放日志的端口配置错误，那么在使用spark-shell提交任务时，会有连接异常java.net.ConnectException

  ```
  spark.eventLog.enabled           true
  spark.eventLog.dir               hdfs://node01:8020/spark/log
  ```

* 可通过`[root@node01 ~]# netstat -ntlp`查看端口信息，找到接收Client连接的RPC端口，该端口用于获取文件系统metadata信息。 

  [hdfs默认端口一览]: https://blog.csdn.net/yeruby/article/details/49406073

* 在hdfs中创建文件夹`[root@node01 ~]# hdfs dfs -mkdir -p /spark/data`

* 在客户端中提交任务对前面的操作进行验证`./spark-shell --master spark://node01:7077 --name xyz`

* 在webui中进行查看

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E9%85%8D%E7%BD%AE%E5%8E%86%E5%8F%B2%E6%97%A5%E5%BF%97%E6%9C%8D%E5%8A%A1%E5%99%A8-%E6%9F%A5%E7%9C%8Bspark%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1.png )

* 在HDFS中验证是否有日志存储

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E9%85%8D%E7%BD%AE%E5%8E%86%E5%8F%B2%E6%9C%8D%E5%8A%A1%E5%99%A8-HDFS%E4%B8%AD%E6%9F%A5%E7%9C%8B%E5%8E%86%E5%8F%B2%E6%97%A5%E5%BF%97%E5%AD%98%E5%82%A8.png )

## 配置日志恢复参数

* 为了恢复存储在hdfs中的日志，还需配置一些属性`[root@node04 spark-2.3.1]# vim ./conf/spark-defaults.conf`

  ```
  spark.eventLog.enabled           true
  spark.eventLog.dir               hdfs://node01:8020/spark/log
  spark.history.fs.logDirectory    hdfs://node01:8020/spark/log
  ```

* 启动历史服务器`[root@node04 sbin]# ./start-history-server.sh `

* 通过` http://node04:18080/ `在webui中查看历史服务器

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E9%85%8D%E7%BD%AE%E5%8E%86%E5%8F%B2%E6%9C%8D%E5%8A%A1%E5%99%A8-%E5%9C%A8webui%E4%B8%AD%E6%9F%A5%E7%9C%8B%E5%8E%86%E5%8F%B2%E6%9C%8D%E5%8A%A1%E5%99%A8.png )

## 通过配置实现将日志压缩存储

* 配置是否将历史日志进行压缩`[root@node04 conf]# vim spark-defaults.conf`

  ```
  spark.eventLog.enabled           true
  spark.eventLog.dir               hdfs://node01:8020/spark/log
  spark.history.fs.logDirectory    hdfs://node01:8020/spark/log
  spark.eventLog.compress          true
  ```

* 在hdfs的webui中进行查看

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E9%85%8D%E7%BD%AE%E5%8E%86%E5%8F%B2%E6%9C%8D%E5%8A%A1%E5%99%A8-%E5%9C%A8webui%E4%B8%AD%E6%9F%A5%E7%9C%8B%E5%8E%8B%E7%BC%A9%E5%AD%98%E5%82%A8%E7%9A%84%E6%97%A5%E5%BF%97.png )



