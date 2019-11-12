# Spark Standalone模式

## 准备工作

### 安装配置jdk8

* 检查jdk版本

* 上传jdk8和Spark2.3.1到node01

* `[root@node01 software]# tar -zxvf ./jdk-8u181-linux-x64.tar.gz `

* 删除jdk安装包`[root@node01 software]# rm -rf ./jdk-8u181-linux-x64.tar.gz`

* 在node01上配置jdk，注意PATH必须放到export jdk前面，不然会导致还是识别老版本的jdk

  ```
  # export JAVA_HOME=/usr/java/jdk1.7.0_67
  export JAVA_HOME=/root/software/jdk1.8.0_181
  export PATH=$JAVA_HOME/bin:$PATH
  export HADOOP_HOME=/opt/sxt/hadoop-2.6.5
  export HBASE_HOME=/root/hbase
  export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin
  ```

* 在其他节点上也配置jdk8

* 发送jdk8到其他节点上`[root@node01 software]# scp -r ./jdk1.8.0_181/ node02:`pwd``

* 每个node上都要`source /etc/profile`

* 直接解压的java需要覆盖软连接路径，软连接路径在`/usr/bin`下，因为spark默认会在usr/bin目录下寻找java

* 改变java的软连接`ln -sf /root/software/jdk1.8.0_181/bin/java  /usr/bin/java`

* 配置hadoop配置文件中的java`[root@node01 hadoop]# vim hadoop-env.sh`

* 每个节点都修改文件`export JAVA_HOME=/root/software/jdk1.8.0_181/bin/java`

* 勘误：应该是改成`export JAVA_HOME=/root/software/jdk1.8.0_181`**不然hadoop集群起不来**

### 安装Spark

* 解压`[root@node01 software]# tar -zxvf ./spark-2.3.1-bin-hadoop2.6.tgz `

* 改名`[root@node01 software]# mv ./spark-2.3.1-bin-hadoop2.6.tgz  ./spark-2.3.1`

* 删除安装包`[root@node01 software]# rm -rf ./spark-2.3.1-bin-hadoop2.6`

* 先配置spark的slave，先复制一份`[root@node01 conf]# cp slaves.template slaves`

* `[root@node01 conf]# vim slaves`添加node02，node03

* `[root@node01 conf]# cp spark-env.sh.template spark-env.sh`

* `[root@node01 conf]# vim spark-env.sh`，配置相关配置信息

  ```
  export SPARK_MASTER_HOST=node01
  export SPARK_MASTER_PORT=7077
  export SPARK_WORKER_CORES=2
  export SPARK_WORKER_MEMORY=3g
  ```

* 发送spark文件夹到node02，node03节点`[root@node01 sbin]# scp -r ./spark-2.3.1/ node03:`pwd```

### 安装Spark客户端

* `[root@node04 software]# mkdir spark`
* `[root@node01 software]# scp -r spark-2.3.1/ node04:`pwd``
* `[root@node04 spark-2.3.1]# cd conf/`
* `[root@node04 conf]# rm -f ./slaves`
* `[root@node04 conf]# rm -f ./spark-env.sh`

## 对Spark集群进行操作

### 运行Spark集群

* 启动spark集群，首先要进入`/root/software/spark-2.3.1/sbin`目录下，再`[root@node01 sbin]# ./start-all.sh`

* 进入web端页面查看`node01:8080`

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/sparkweb%E9%A1%B5%E9%9D%A2%E6%9F%A5%E7%9C%8B.png )

* 关闭集群`[root@node01 sbin]# ./stop-all.sh`

### Standalone模式提交任务

* 启动集群



