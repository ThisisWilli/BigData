# flink API学习

使用DataSet API来实现批处理，DataStream API来实现流处理

## flink程序的组成部分

* 1、获取执行环境，
* 2、加载/创建初始数据，
* 3、指定对数据的转换操作，
* 4、指定计算结果存放的位置，
* 5、触发程序执行

## idea中搭建flink程序

* 用mvn命令下载模版

  ```shell
  mvn archetype:generate  -DarchetypeGroupId=org.apache.flink  -DarchetypeArtifactId=flink-quickstart-java   -DarchetypeVersion=1.9.1
  ```

  ```
  flink run -p 1 -c com.willi.BatchJob ./flinkStudy.jar
  ```

  

