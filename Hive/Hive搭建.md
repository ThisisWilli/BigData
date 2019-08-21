# Hive搭建

## 搭建模式

### 模式一：单用户模式安装

通过网络连接到一个数据库中，是最经常使用到的模式

![](pic\Hive搭建模式一.png)

### 模式二：远程非服务器模式

用于非Java客户端访问元数据库，在服务器端启动MetaStoreServer，客户端利用Thrift协议通过MetaStoreServer访问元数据库

![](pic\Hive搭建模式二.png)

## 单用户模式安装

### 安装mysql服务并更改权限

* 先在node01中安装mysql数据库`[root@node01 ~]# yum install mysql-server`

* 启动mysql服务`[root@node01 ~]# service mysqld start`

* `[root@node01 ~]# mysql`

* `mysql> show databases;`

* `mysql> use mysql`

* `mysql> show tables;`

* 讲所有权限赋给所有库和所有表`mysql> select host, user, passord from user;`

* `mysql> grant all privileges on *.* to 'root'@'%' identified by '123' with grant option;`

* `mysql> select host, user, password from user;`再次查看

  ```
  +-----------+------+-------------------------------------------+
  | host      | user | password                                  |
  +-----------+------+-------------------------------------------+
  | localhost | root |                                           |
  | node01    | root |                                           |
  | 127.0.0.1 | root |                                           |
  | localhost |      |                                           |
  | node01    |      |                                           |
  | %         | root | *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
  +-----------+------+-------------------------------------------+
  6 rows in set (0.00 sec)
  ```

* `mysql> delete from user where host!='%';`

* `mysql> quit;`

* `mysql> flush privileges;`

* `[root@node01 ~]# mysql -uroot -p`，并输入密码

* `mysql> show databases;`

  ```
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | test               |
  +--------------------+
  ```

### 安装Hive

在node02上安装Hive

* `[root@node02 ~]# tar -zxvf apache-hive-1.2.1-bin.tar.gz`

* `[root@node02 ~]# mv apache-hive-1.2.1-bin hive`

* `[root@node02 hive]# vi /etc/profile`

  ```
  export HIVE_HOME=/root/hive
  export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$HIVE_HOME/bin
  ```

* `[root@node02 hive]# . /etc/profile`

### 配置文件

* `[root@node02 conf]# mv hive-default.xml.template hive-site.xml`

  ```xml
  <configuration>
    <property>  
          <name>hive.metastore.warehouse.dir</name>
          <value>/user/hive_remote/warehouse</value>
    </property>
     
    <property> 
          <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://node01/hive_remote?createDatabaseIfNotExist=true</value>
    </property>
     
    <property>
          <name>javax.jdo.option.ConnectionDriverName</name>
          <value>com.mysql.jdbc.Driver</value>
    </property>
    
    <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>root</value>
    </property>
    
    <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>123</value>
    </property>
  </configuration>
  
  ```

* `[root@node02 ~]# cp mysql-connector-java-5.1.32-bin.jar /root/hive/lib/`

* 启动hive`[root@node02 ~]# hive`

* 在Hive中创建一张表`hive> create table tbl(id int, age int);`

* 在表中插入数据`hive> insert into tbl values(1, 1);`

* 在resourcemanager节点中查看mr任务

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hive/pic/%E5%9C%A8resourceManage%E8%8A%82%E7%82%B9%E4%B8%AD%E6%9F%A5%E7%9C%8Bhive%E4%BB%BB%E5%8A%A1.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hive/pic/在resourceManage节点中查看hive任务.PNG)

* 查看表中数据`hive> select * from tbl;`

  ```
  hive> select * from tbl;
  OK
  1	1
  Time taken: 0.097 seconds, Fetched: 1 row(s)
  ```


### 可能会出现的问题

执行hive命令时可能会提示下列错误

```
Logging initialized using configuration in jar:file:/home/hive/lib/hive-common-1.2.1.jar!/hive-log4j.properties
[ERROR] Terminal initialization failed; falling back to unsupported
java.lang.IncompatibleClassChangeError: Found class jline.Terminal, but interface was expected
	at jline.TerminalFactory.create(TerminalFactory.java:101)
	at jline.TerminalFactory.get(TerminalFactory.java:158)
	at jline.console.ConsoleReader.<init>(ConsoleReader.java:229)
	at jline.console.ConsoleReader.<init>(ConsoleReader.java:221)
	at jline.console.ConsoleReader.<init>(ConsoleReader.java:209)
	at org.apache.hadoop.hive.cli.CliDriver.setupConsoleReader(CliDriver.java:787)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:721)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:681)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:212)

Exception in thread "main" java.lang.IncompatibleClassChangeError: Found class jline.Terminal, but interface was expected
	at jline.console.ConsoleReader.<init>(ConsoleReader.java:230)
	at jline.console.ConsoleReader.<init>(ConsoleReader.java:221)
	at jline.console.ConsoleReader.<init>(ConsoleReader.java:209)
	at org.apache.hadoop.hive.cli.CliDriver.setupConsoleReader(CliDriver.java:787)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:721)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:681)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:212)
```

原因：hadoop文件夹下的jline包版本太低，将hive文件夹下的jline包拷贝到hadoop文件夹下即可

`cp /hive/apache-hive-1.2.1-bin/lib/jline-2.12.jar /hadoop-2.6.5/share/hadoop/yarn/lib`

## 多平台搭建

* 将node02 上的hive拷贝到node03，node04中`[root@node02 ~]# scp -r hive/ node03:/root/`，`[root@node02 ~]# scp -r hive/ node04:/root/`

* 在node03和node04中配置环境变量`[root@node03 ~]# vi /etc/pro`

  ```
  export JAVA_HOME=/usr/java/jdk1.7.0_67
  export HADOOP_HOME=/opt/sxt/hadoop-2.6.5
  export ZOOKEEPER_HOME=/opt/sxt/zookeeper-3.4.6
  export HIVE_HOME=/root/hive
  export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$HIVE_HOME/bin
  ```

  `[root@node03 ~]# source /etc/profile`

* 更改配置`[root@node03 conf]# vi hive-site.xml`，更改两处

  ```xml
  <configuration>
    <property>
          <name>hive.metastore.warehouse.dir</name>
          <value>/user/hive/warehouse</value>       
    </property>
     
    <property>
          <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://node01/hive?createDatabaseIfNotExist=true</value>       
    </property>
  
    <property>
          <name>javax.jdo.option.ConnectionDriverName</name>
          <value>com.mysql.jdbc.Driver</value>
    </property>
  
    <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>root</value>
    </property> 
  
    <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>123</value>
    </property>
  </configuration>
  ```

* 在node04中更改配置`[root@node04 conf]# vi hive-site.xml`

  ```xml
  <configuration>
    <property>
          <name>hive.metastore.warehouse.dir</name>
          <value>/user/hive/warehouse</value>       
    </property> 
    <property>
          <name>hive.metastore.uris</name>
          <value>thrift://node03:9083</value>
  
  </configuration>
  
  ```

* 在node03中启动hive服务端，及元数据进行测试`[root@node03 conf]# hive --service metastore`

* 在node04中启动hive服务`[root@node04 conf]# hive`，会报jline包过的错误

  ```
  [root@node04 sxt]# cd hadoop-2.6.5/share/hadoop/yarn/lib
  [root@node04 lib]# rm -rf jline-0.9.94.jar 
  [root@node04 lib]# cp /root/hive/lib/jline-2.12.jar    ./
  ```

* 在node01的mysql服务端新建一张表格

* 在node04中新建一张表格进行测试`hive> create table psn(id int);`可在node01:50070处进行查看

* 在node04中对新建的表插入一条数据，并且转换成mapreduce操作

  ```
  hive> insert into psn values(1);
  Query ID = root_20190820130047_149f0a17-5089-47c6-b7d5-2052472e4466
  Total jobs = 3
  Launching Job 1 out of 3
  Number of reduce tasks is set to 0 since there's no reduce operator
  Starting Job = job_1566275397658_0001, Tracking URL = http://node03:8088/proxy/application_1566275397658_0001/
  Kill Command = /opt/sxt/hadoop-2.6.5/bin/hadoop job  -kill job_1566275397658_0001
  Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
  2019-08-20 13:01:17,975 Stage-1 map = 0%,  reduce = 0%
  2019-08-20 13:01:35,447 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.04 sec
  MapReduce Total cumulative CPU time: 1 seconds 40 msec
  Ended Job = job_1566275397658_0001
  Stage-4 is selected by condition resolver.
  Stage-3 is filtered out by condition resolver.
  Stage-5 is filtered out by condition resolver.
  Moving data to: hdfs://mycluster/user/hive/warehouse/psn/.hive-staging_hive_2019-08-20_13-00-47_029_6853042225447250554-1/-ext-10000
  Loading data to table default.psn
  Table default.psn stats: [numFiles=1, numRows=1, totalSize=2, rawDataSize=1]
  MapReduce Jobs Launched: 
  Stage-Stage-1: Map: 1   Cumulative CPU: 1.04 sec   HDFS Read: 3306 HDFS Write: 69 SUCCESS
  Total MapReduce CPU Time Spent: 1 seconds 40 msec
  OK
  Time taken: 49.828 seconds
  ```

* 在node02中查看之前hive中的数据

  ```
  [root@node02 ~]# hdfs dfs -cat /user/hive_remote/warehouse/tbl/*
  11
  [root@node02 ~]# hdfs dfs -get /user/hive_remote/warehouse/tbl/*
  [root@node02 ~]# ll
  total 156256
  -rw-r--r--  1 root root        4 Aug 20 13:05 000000_0
  -rw-------. 1 root root      900 May 21 05:28 anaconda-ks.cfg
  -rw-r--r--  1 root root 92834839 Aug 19 05:52 apache-hive-1.2.1-bin.tar.gz
  drwxr-xr-x  9 root root     4096 Aug 19 05:58 hive
  -rw-r--r--. 1 root root     8815 May 21 05:28 install.log
  -rw-r--r--. 1 root root     3384 May 21 05:28 install.log.syslog
  -rw-r--r--  1 root root   969020 Aug 19 05:52 mysql-connector-java-5.1.32-bin.jar
  drwxr-xr-x  2 root root     4096 Jul 29 23:35 software
  -rw-r--r--  1 root root 66152921 Aug 16 11:50 WeatherTemp.jar
  -rw-r--r--  1 root root    10060 Aug 20 12:29 zookeeper.out
  [root@node02 ~]# cat 000000_0 
  11
  [root@node02 ~]# cat -A 000000_0 
  1^A1$
  ```

* https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL DDL语法地址

* 尝试建表

  ```sql
  create table psn
  (
  id int,
  name string,
  likes array<string>,
  address map<string, string>
  )
  row format delimited
  fields terminated by ','
  collection items terminated by '-'
  map keys terminated by ':';
  ```

* 在node04中输入`hive> desc formatted psn`进行查看

* 插入数据，将数据按文件路径插入表中,在node04中`hive> load data local inpath '/root/data/data' into table psn;`

* 在node04中查看数据

  ```
  hive> select * from psn;
  OK
  1	小明1	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  2	小明2	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  3	小明3	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  4	小明4	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  5	小明5	["lol","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  6	小明6	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  7	小明7	["lol","book"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  8	小明8	["lol","book"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  9	小明9	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  Time taken: 0.07 seconds, Fetched: 9 row(s)
  ```

