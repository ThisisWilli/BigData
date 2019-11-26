# Spark中配置Master-HA

## 两种模式

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Master-HA.png )

## 配置Master-HA

可在 https://spark.apache.org/docs/2.3.1/spark-standalone.html 根据文档进行配置

* 在node01的spark文件中进行配置`[root@node01 conf]# vim spark-env.sh`

  ```
  export SPARK_MASTER_HOST=node01
  export SPARK_MASTER_PORT=7077
  export SPARK_WORKER_CORES=2
  export SPARK_WORKER_MEMORY=3g
  export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=node02:2181,node03:2181,node04:2181 -Dspark.deploy.zookeeper.dir=/MasterHA1125"
  export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
  ```

* 将配置完的发送到其他两个Worker节点`[root@node01 conf]# scp ./spark-env.sh node02:`pwd` [root@node01 conf]# scp ./spark-env.sh node03:`pwd``

* 将node02这个worker节点设置为standby的master节点`[root@node02 conf]# vim spark-env.sh`

  ```
  export SPARK_MASTER_HOST=node02
  ```

* 配置完成之后分别启动standalone集群`[root@node01 sbin]# ./start-all.sh `, `[root@node02 sbin]# ./start-master.sh`

* 进入web页面中进行查看

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Master-HA%E9%85%8D%E7%BD%AEnode01webui.png )

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Master-HA%E9%85%8D%E7%BD%AEnode02webui.png )

* 在客户端中提交任务进行测试`[root@node04 bin]# ./spark-submit --master spark://node01:7077,node02:7077 --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 1000`
* 杀死node01中的Master进程，过一段时间之后可见node02成功接管

## 验证

* `[root@node04 bin]# ./spark-submit --master spark://node01:7077,node02:7077 --executor-cores 1 --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 5000`

* 登录webui进行查看

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E8%A7%84%E5%AE%9Aexecutor-cores%E4%B8%AA%E6%95%B0.png )

* `[root@node04 bin]# ./spark-submit --master spark://node01:7077,node02:7077 --executor-cores 1 --total-executor-cores 2 --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-
  2.3.1.jar 5000`

* 登录webui进行查看

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E8%A7%84%E5%AE%9Atotal-executor.png  )