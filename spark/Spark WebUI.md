# Spark WebUI

* 启动zookeeper，hdfs集群，以及所有节点，启动hdfs是为了让其读文件

  ```
  zkServer.sh start
  start-dfs.sh
  start-all.sh
  ```

* 进入spark-shell

* 在hdfs中创建文件`[root@node01 ~]# hdfs dfs -mkdir -p /spark/data`

* 先将文件上传到节点中的一个文件夹

* 再将文件上传到hdfs中`[root@node01 ~]# hdfs dfs -put /root/wc.txt /spark/data`

* 为了用spark-shell读取文件，所以需要启动standalone集群`[root@node01 sbin]# ./start-all.sh`，同时也要启动spark-shell

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark-webui%E5%90%AF%E5%8A%A8spark-shell%E6%88%90%E5%8A%9F.png )

* 在查看spark-shell运行的任务时，应该去node04:4040端口查看，**webui中没有显示spark-shell的运行应该是因为有些地方没有配置**

