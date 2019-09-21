# Spark提交任务的方式

## Spark基于Standalone-client提交任务

![](pic\Spark 基于Standalone-client 模式提交任务.png)

### 提交命令

* `[root@node04 bin]# ./spark-submit --master spark://node01:7077 --deploy-mode client --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar  100`
* `./spark-submit --master spark://node01:7077 --class ...jar`

### 基于Standalone-client提交任务的特点

* Spark基于Standalone-client模式提交人物，每个Spark application都有自己的独立的Driver，如果在客户端提交100个application，会有100个driver进程在客户端启动，Driver负责发送task，监控task执行，回收结果，很容易造成客户端网卡流量激增的问题，这种模式适用于程序测试，不适用于生产环境，在客户端可以看到task执行和结果
* **client提交任务在webui中不会看到有driver进程**

## Spark基于Standalone-cluster模式提交任务

![](pic\Spark 基于Standalone-cluster 模式提交任务.png)

### 提交命令

* `[root@node04 bin]# ./spark-submit --master spark://node01:7077 --deploy-mode cluster --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar  100`

### 基于Standalone-cluster提交任务的特点

* Spark基于Standalone-cluster模式提交任务，当在客户端提交多个application时，Driver时随机在某些Worker节点启动，客户端就没有网卡流量激增问题，将这种问题分散到集群中，在客户端看不到task执行和结果，要去webui中查看结果，这种模式适用于生产环境。
* **cluster提交任务在webui中会看到driver进程的产生，并可以查看执行日志**

## 基于Yarn-client模式来提交任务

![](pic\Spark 基于Yarn-client模式提交任务.png)

### 提交命令

* ` ./spark-submit --master yarn --class ..jar...`
* `./spark-submit --master yarn-client --class ..jar`
* `./spark-submit --master yarn --deploy-mode client --class ..jar`

### Spark基于Yarn-client模式提交任务的特点

* 当在driver提交多个application时，会有网卡流量激增问题，这种模式适用于程序测试，不适用于生产环境，当客户端可以看到task的执行和结果

### Spark基于Yarn-client模式提交人物产生的错误

初次在Yarn-client模式下提交时，第一次运行报错，主要为一下三部分

* ```
  client token: N/A
  	 diagnostics: Application application_1568870310822_0001 failed 2 times due to AM Container for appattempt_1568870310822_0001_000002 exited with
    exitCode: 1For more detailed output, check application tracking page:http://node03:8088/proxy/application_1568870310822_0001/Then, click on links to logs of each a
  ttempt.Diagnostics: Exception from container-launch.
  Container id: container_1568870310822_0001_02_000001
  Exit code: 1
  Stack trace: ExitCodeException exitCode=1: 
  	at org.apache.hadoop.util.Shell.runCommand(Shell.java:575)
  	at org.apache.hadoop.util.Shell.run(Shell.java:478)
  	at org.apache.hadoop.util.Shell$ShellCommandExecutor.execute(Shell.java:766)
  	at org.apache.hadoop.yarn.server.nodemanager.DefaultContainerExecutor.launchContainer(DefaultContainerExecutor.java:212)
  	at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:302)
  	at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:82)
  	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
  	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
  	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
  	at java.lang.Thread.run(Thread.java:745)
  ```

* ```
  2019-09-19 13:19:54 INFO  Client:54 - Deleted staging directory hdfs://mycluster/user/root/.sparkStaging/application_1568870310822_0001
  2019-09-19 13:19:54 ERROR SparkContext:91 - Error initializing SparkContext.
  org.apache.spark.SparkException: Yarn application has already ended! It might have been killed or unable to launch application master.
  	at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.waitForApplication(YarnClientSchedulerBackend.scala:89)
  	at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:63)
  	at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:164)
  	at org.apache.spark.SparkContext.<init>(SparkContext.scala:500)
  	at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2493)
  	at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:933)
  	at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:924)
  	at scala.Option.getOrElse(Option.scala:121)
  	at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:924)
  	at org.apache.spark.examples.SparkPi$.main(SparkPi.scala:31)
  	at org.apache.spark.examples.SparkPi.main(SparkPi.scala)
  	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  	at java.lang.reflect.Method.invoke(Method.java:498)
  	at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
  	at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:894)
  	at org.apache.spark.deploy.SparkSubmit$.doRunMain$1(SparkSubmit.scala:198)
  	at org.apache.spark.deploy.SparkSubmit$.submit(SparkSubmit.scala:228)
  	at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:137)
  	at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
  
  ```

* ```
  Exception in thread "main" org.apache.spark.SparkException: Yarn application has already ended! It might have been killed or unable to launch applicatio
  n master.	at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.waitForApplication(YarnClientSchedulerBackend.scala:89)
  	at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:63)
  	at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:164)
  	at org.apache.spark.SparkContext.<init>(SparkContext.scala:500)
  	at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2493)
  	at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:933)
  	at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:924)
  	at scala.Option.getOrElse(Option.scala:121)
  	at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:924)
  	at org.apache.spark.examples.SparkPi$.main(SparkPi.scala:31)
  	at org.apache.spark.examples.SparkPi.main(SparkPi.scala)
  	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  	at java.lang.reflect.Method.invoke(Method.java:498)
  	at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
  	at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:894)
  	at org.apache.spark.deploy.SparkSubmit$.doRunMain$1(SparkSubmit.scala:198)
  	at org.apache.spark.deploy.SparkSubmit$.submit(SparkSubmit.scala:228)
  	at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:137)
  	at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
  ```

#### 解决方法

* 在resourcemanager中查看日志，显示编译出来的class版本，与jdk版本不匹配，则检查hadoop-env.sh,yarn-env.sh,mapred-env.sh中jdk版本是否一致，不一致在需要在每个node中更改

* 再次更改之后记得重启集群！再次进行测试，仍然失败，在resourcemanager的webui上查看显示内存爆了

  ![](pic\Spark中client模式查找错误.png)

* 在yarn-site.xml中添加配置

  ```xml
  	<property>
          <name>yarn.nodemanager.pmem-check-enabled</name>
          <value>false</value>
      </property>
  
      <property>
          <name>yarn.nodemanager.vmem-check-enabled</name>
          <value>false</value>
      </property>
  
  ```

* 重启集群，再次提交任务，终于成功！

  ![](pic\yarn改错之后提交成功.png)

## Spark基于Yarn-cluster模式提交任务

![](pic\Spark 基于Yarn-cluster 模式提交任务.jpg)

### 提交命令

* `./spark-submit --master yarn --deploy-mode cluster --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar`
* `./spark-submit --master yarn-cluster --class ..jar`

### Spark基于Yarn-client模式提交任务的特点

* Spark基于Yarn-cluster模式提交任务，当有多个application提交时，每个application的Driver(AM)是分散到集群中的NM中启动，没有客户端的网卡流量激增问题，将这种问题分散到集群当中。在客户端看不到task的执行和结果，要去webui中查看，这种模式适用于生产环境。

  







