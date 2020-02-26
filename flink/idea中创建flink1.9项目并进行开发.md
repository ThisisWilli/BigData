# idea中创建flink1.9项目并编写代码提交到本地运行

## 前言

flink在idea中开发真的有点麻烦。。

## 开发环境

* macos15.3
* jdk1.8
* flink1.9
* scala2.12

## 准备工作

* 在[flink官网](https://flink.apache.org/downloads.html)下载flink1.9或更新版本，到本地解压

* flink的conf目录下的master和slave文件分别配置flink的master节点和slave节点，由于本次为单机模式，master节点ip和端口使用默认的localhost:8081，slave节点不配置

* 配置flink的环境变量，在用户目录下`vim .bash_profile`，添加如下

  ```
  export FLINK_HOME=flink的安装路径
  export PATH=$PATH:$FLINK_HOME/bin
  ```

* 配置完记得`source .bash_profile`

* 在终端输入start-cluster.sh，再进入localhost:8081页面查看是否有flink的dashboard

## 创建maven项目

* 找一个空的文件夹，进入终端，mvn 拉取flink1.9.1的flink-java模版，填写你项目的groupId和ArtifactsId，注意尽量不要选择1.9.1以上的版本，因为阿里云的maven镜像好像没有高版本的flink-java模版，下载起来会很慢

  ```
  mvn archetype:generate  -DarchetypeGroupId=org.apache.flink  -DarchetypeArtifactId=flink-quickstart-java   -DarchetypeVersion=1.9.1
  ```

* 成功之后打开idea，import刚刚用maven编译的项目，不要用open

* idea导入项目后，先检查scala的版本，**要保证pom.xml中的scala版本与系统安装的scala版本一致，否则打包执行程序后可能会报错，这点非常重要**

  ```xml
  <properties>
  		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  		<flink.version>1.9.0</flink.version>
  		<java.version>1.8</java.version>
  		<scala.binary.version>2.12</scala.binary.version>
  		<maven.compiler.source>${java.version}</maven.compiler.source>
  		<maven.compiler.target>${java.version}</maven.compiler.target>
  	</properties>
  ```

## 编写代码

* 在编写代码前，还要给module添加依赖，右键module，在dependenies中加入flink安装目录下的lib和opt两个文件夹

* 编写测试代码

  ```java
  public class StreamTest {
      public static void main(String[] args) throws Exception {
          // set up the streaming execution environment
          final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
          DataStream<String> source = env.readTextFile("rtms-flink/data/data1");
          source.map(line->{
              return line + "=======";
          }).print();
          // execute program
          env.execute("Flink Streaming Java API Skeleton");
      }
  
  ```

  执行成功

  ```
  3> hello flink=======
  20:47:31,359 INFO  org.apache.flink.runtime.taskmanager.Task                     - Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (3/4) (e66045df516d4c018ba1a6146a5484e7) switched from RUNNING to FINISHED.
  20:47:31,359 INFO  org.apache.flink.runtime.taskmanager.Task                     - Freeing task resources for Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (3/4) (e66045df516d4c018ba1a6146a5484e7).
  20:47:31,360 INFO  org.apache.flink.runtime.taskmanager.Task                     - Ensuring all FileSystem streams are closed for task Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (3/4) (e66045df516d4c018ba1a6146a5484e7) [FINISHED]
  20:47:31,360 INFO  org.apache.flink.runtime.taskexecutor.TaskExecutor            - Un-registering task and sending final execution state FINISHED to JobManager for task Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out e66045df516d4c018ba1a6146a5484e7.
  20:47:31,361 INFO  org.apache.flink.runtime.executiongraph.ExecutionGraph        - Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (3/4) (e66045df516d4c018ba1a6146a5484e7) switched from RUNNING to FINISHED.
  20:47:31,368 INFO  org.apache.flink.runtime.taskmanager.Task                     - Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (4/4) (97b4fdf44415e8c457f0b03f9266f868) switched from RUNNING to FINISHED.
  20:47:31,368 INFO  org.apache.flink.runtime.taskmanager.Task                     - Freeing task resources for Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (4/4) (97b4fdf44415e8c457f0b03f9266f868).
  20:47:31,368 INFO  org.apache.flink.runtime.taskmanager.Task                     - Ensuring all FileSystem streams are closed for task Split Reader: Custom File Source -> Map -> Sink: Print to Std. Out (4/4) (97b4fdf44415e8c457f0b03f9266f868) [FINISHED]
  2> hello spark=======
  ```

* flink执行打包的jar包

  ```
  flink run -p 1 -c com.willi.GetKafkaData ./target/rtms-flink-0.0.1.jar
  ```

  

