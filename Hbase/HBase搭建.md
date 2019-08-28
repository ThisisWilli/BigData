# HBase搭建

## standalone模式搭建

* HBase自带zk，要在没有安装zookeeper的节点安装HBase，所以选择node01

* 上传源码包到node01

* 解压源码包`[root@node01 ~]# tar -zxvf tar -zxvf hbase-0.98.12-hadoop2-bin.tar.gz`

* 安装完之后更改名称`[root@node01 ~]# mv hbase-0.98.12-hadoop2 hbase`

* 配置环境变量`[root@node01 hbase]# vi /etc/profile`

  ```
  export JAVA_HOME=/usr/java/jdk1.7.0_67
  export HADOOP_HOME=/opt/sxt/hadoop-2.6.5
  export HBASE_HOME=/root/hbase
  export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin
  ```

* 执行更改`source /etc/profile`

* 更改HBase配置文件`[root@node01 conf]# vi hbase-env.sh`，查找时可按`!ls /usr/java`更改java的环境变量`export JAVA_HOME=/usr/java/jdk1.7.0_67`

* 配置`[root@node01 conf]# vi hbase-site.xml`

  ```xml
  <configuration>
  <property>     
          <name>hbase.rootdir</name>     
          <value>file:///home/testuser/hbase</value>   
  </property>   
  <property>     
          <name>hbase.zookeeper.property.dataDir</name>     
          <value>/home/testuser/zookeeper</value>   
  </property> 
  </configuration>
  ```

* 切换到home目录`[root@node01 conf]# cd /home/`并启动hbase`[root@node01 home]# start-hbase.sh `

* 启动完成之后再web页面进行查看`node01:60010`

  ![](pic\web页面查看hbase.PNG)

* 进入hbase shell进行查看

  ```
  [root@node01 home]# hbase shell
  2019-08-28 07:27:14,594 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
  HBase Shell; enter 'help<RETURN>' for list of supported commands.
  Type "exit<RETURN>" to leave the HBase Shell
  Version 0.98.12-hadoop2, rbbf8bf4362e858bd9bd0511c9289c31b09b7a927, Thu Apr  9 19:43:38 PDT 2015
  hbase(main):001:0>
  ```

* 在hbase shell中查看命名空间`hbase(main):004:0> list_namespace`

  ```
  hbase(main):004:0> list_namespace
  NAMESPACE                                                                                                                                             
  SLF4J: Class path contains multiple SLF4J bindings.
  SLF4J: Found binding in [jar:file:/root/hbase/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: Found binding in [jar:file:/opt/sxt/hadoop-2.6.5/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
  default                                                                                                                                               
  hbase                                                                                                                                       
  2 row(s) in 0.9910 seconds
  ```

* 并在home目录下查看文件夹

  ```
  [root@node01 home]# ls
  testuser
  [root@node01 home]# cd testuser/
  [root@node01 testuser]# ls
  hbase  zookeeper
  [root@node01 testuser]# cd hbase/
  [root@node01 hbase]# ls
  data  hbase.id  hbase.version  oldWALs  WALs
  [root@node01 hbase]# cd data/
  [root@node01 data]# ls
  default  hbase
  ```

  default是默认的一个库，没有指定namespace那么所有表都会存到这个命名空间中

* 尝试在hbase中建表`hbase(main):006:0> create 'tbl','cf'`

* 创建完成之后，再到home目录下查看是否又tbl这个数据

  ```
  [root@node01 data]# cd default/
  [root@node01 default]# ls
  tbl
  [root@node01 default]# cd tbl/
  [root@node01 tbl]# ls
  c7b5004d9577d853368f1bc61e174f9d
  ```

  乱码为md5加密生成，就是**region**

* 插入数据`hbase(main):009:0> put 'tbl','2','cf:name','zhangsanfeng'`

* 查看数据

  ```
  hbase(main):012:0> get 'tbl', '2'
  COLUMN                                 CELL                                                                                                           
   cf:name                               timestamp=1566949549780, value=zhangsanfeng                                                                    
  1 row(s) in 0.0310 seconds
  
  hbase(main):013:0> scan 'tbl'
  ROW                                    COLUMN+CELL                                                                                                    
   2                                     column=cf:name, timestamp=1566949549780, value=zhangsanfeng                                                    
  1 row(s) in 0.0130 seconds
  ```

* 将表插入列族`hbase(main):014:0> flush 'tbl'`，再在cf文件夹下进行查看

  ```
  [root@node01 cf]# hbase hfile -p -m -f ede2b19eee994f13a4bd9b123b11fbea 
  SLF4J: Class path contains multiple SLF4J bindings.
  SLF4J: Found binding in [jar:file:/root/hbase/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: Found binding in [jar:file:/opt/sxt/hadoop-2.6.5/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
  2019-08-28 07:54:39,546 INFO  [main] Configuration.deprecation: fs.default.name is deprecated. Instead, use fs.defaultFS
  2019-08-28 07:54:39,618 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
  2019-08-28 07:54:39,861 INFO  [main] util.ChecksumType: Checksum using org.apache.hadoop.util.PureJavaCrc32
  2019-08-28 07:54:39,862 INFO  [main] util.ChecksumType: Checksum can use org.apache.hadoop.util.PureJavaCrc32C
  K: 2/cf:name/1566949549780/Put/vlen=12/mvcc=0 V: zhangsanfeng
  ```

* 再插入一条数据`hbase(main):015:0> put 'tbl','1','cf:age','12'`

* 再插入一条数据`hbase(main):018:0> put 'tbl','2','cf:name','lisi'`

* `hbase(main):020:0> flush 'tbl'` 再在cf中查看数据

  ```
  [root@node01 cf]# ls
  82573473ad2c495581586af2afbf8276
  [root@node01 cf]# hbase hfile -p -f 82573473ad2c495581586af2afbf8276 
  SLF4J: Class path contains multiple SLF4J bindings.
  SLF4J: Found binding in [jar:file:/root/hbase/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: Found binding in [jar:file:/opt/sxt/hadoop-2.6.5/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
  2019-08-28 08:10:08,081 INFO  [main] Configuration.deprecation: fs.default.name is deprecated. Instead, use fs.defaultFS
  2019-08-28 08:10:08,155 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
  2019-08-28 08:10:08,379 INFO  [main] util.ChecksumType: Checksum using org.apache.hadoop.util.PureJavaCrc32
  2019-08-28 08:10:08,384 INFO  [main] util.ChecksumType: Checksum can use org.apache.hadoop.util.PureJavaCrc32C
  K: 1/cf:age/1566950591866/Put/vlen=2/mvcc=0 V: 12
  K: 2/cf:name/1566950760739/Put/vlen=4/mvcc=0 V: lisi
  Scanned kv count -> 2
  ```

  只有一个文件，并且zhangsan也被覆盖，说明文件个数达到3个之后，会自动合成为一个文件

* 查看meta数据

  ```
  hbase(main):028:0> list_namespace_tables 'hbase'
  TABLE                                                                                              meta                                                                                               namespace                                                                                         2 row(s) in 0.0110 seconds
  hbase(main):029:0> scan 'hbase:meta'
  ROW                                    COLUMN+CELL                                                                                                    
   hbase:namespace,,1566948178564.922609 column=info:regioninfo, timestamp=1566948178663, value={ENCODED => 9226092a46cc0cc0b75d0e9274cee3bc, NAME => 'h
   2a46cc0cc0b75d0e9274cee3bc.           base:namespace,,1566948178564.9226092a46cc0cc0b75d0e9274cee3bc.', STARTKEY => '', ENDKEY => ''}                
   hbase:namespace,,1566948178564.922609 column=info:seqnumDuringOpen, timestamp=1566948178735, value=\x00\x00\x00\x00\x00\x00\x00\x01                  
   2a46cc0cc0b75d0e9274cee3bc.                                                                                                                          
   hbase:namespace,,1566948178564.922609 column=info:server, timestamp=1566948178735, value=node01:54078                                                
   2a46cc0cc0b75d0e9274cee3bc.                                                                                                                          
   hbase:namespace,,1566948178564.922609 column=info:serverstartcode, timestamp=1566948178735, value=1566948170792                                      
   2a46cc0cc0b75d0e9274cee3bc.                                                                                                                          
  1 row(s) in 0.0140 seconds
  
  hbase(main):030:0> create 'hbase:test','cf'
  0 row(s) in 0.1390 seconds
  
  => Hbase::Table - hbase:test
  hbase(main):031:0> list_namespace_tables 'hbase'
  TABLE                                                                                                                                                 
  meta                                                                                                                                                  
  namespace                                                                                                                                             
  test                                                                                                                                                  
  3 row(s) in 0.0060 seconds
  ```

* 注意上表中只有一个row，key

## HBase完全分布式搭建

### 准备工作

* 网络
* hosts文件
* ssh免密钥
  * `[root@node02 ~]# ssh-keygen`
  * `[root@node02 ~]# ssh-copy-id -i .ssh/id_rsa.pub node03`
  * hdfs-site.xml文件中如果使用了dsa则必须按照hadoop中的规定进行配置
* 时间，各个机器或者节点时间必须一致，以下两个命令都是4台机器同时启动
  * `yum install ntpdate`
  * `ntpdate ntp1.aliyun.com` 
* jdk版本

### 解压配置

* 更改node01中hbase的配置，让HBase不使用自带的zookeeper，而是使用我们搭建的zookeeper

  * `[root@node01 conf]# vi hbase-env.sh`
* ``export HBASE_MANAGES_ZK=false`

* 更改hbase-site.xml `[root@node01 conf]# vi hbase-site.xml`

  * 需要更改hdfs的名称以及配置了zookeeper的节点名称

  ```
  <configuration>
  	<property>
  		<name>hbase.rootdir</name>
  		<value>hdfs://mycluster/hbase</value>
  	</property>
  	<property>
  		<name>hbase.cluster.distributed</name>
  		<value>true</value>
  	</property>
  	<property>
  		<name>hbase.zookeeper.quorum</name>
  		<value>node02,node03,node04</value>
  	</property>
  </configuration>
  ```

* 更改regionservers，使用node02，node03，node04`[root@node01 conf]# vi regionservers`

  ```
  node02
  node03
  node04
  ```

* 编辑备机`[root@node01 conf]# vi backup-masters`，从哪一台启动哪一台就是主机

* 将hdfs-site.xml拷贝到当前的conf目录下`[root@node01 conf]# cp /opt/sxt/hadoop-2.6.5/etc/hadoop/hdfs-site.xml  ./`

### 启动

* 将hbase分发到各个节点上

  ```
  [root@node01 ~]# scp -r hbase node02:/root/
  [root@node01 ~]# scp -r hbase node03:/root/
  [root@node01 ~]# scp -r hbase node04:/root/
  ```

* 启动hbase

  ```
  [root@node01 conf]# start-hbase.sh
  starting master, logging to /root/hbase/logs/hbase-root-master-node01.out
  node03: starting regionserver, logging to /root/hbase/bin/../logs/hbase-root-regionserver-node03.out
  node02: starting regionserver, logging to /root/hbase/bin/../logs/hbase-root-regionserver-node02.out
  node04: starting regionserver, logging to /root/hbase/bin/../logs/hbase-root-regionserver-node04.out
  node04: starting master, logging to /root/hbase/bin/../logs/hbase-root-master-node04.out
  ```

  单节点启动命令`hbase-daemon.sh start master`

  ![](pic\全分布是web页面查看.PNG)

  ### 查看数据

* 在安装zookeeper的节点上启动zkCli.sh

  ```
  [zk: localhost:2181(CONNECTED) 0] ls /
  [hbase, hadoop-ha, yarn-leader-election, zookeeper, hiverserver2]
  ```

* 在hbase shell中创建一张新表之后，再到zookeeper客户端进行查看，zk中存储这数据表的信息，以及regionserver的信息

  ```
  [zk: localhost:2181(CONNECTED) 6] ls /hbase/table
  [psn, hbase:meta, hbase:namespace
  [zk: localhost:2181(CONNECTED) 7] ls /hbase/rs
  [node03,60020,1566982522325, node02,60020,1566982522294, node04,60020,1566982522914]
  ```

* 查看元数据信息

  ```
  [zk: localhost:2181(CONNECTED) 10] get /hbase/meta-region-server
  �regionserver:60020 
                      �mE$PBUF
   
  node02�������- 
  cZxid = 0x2300000043
  ctime = Wed Aug 28 18:50:21 CST 2019
  mZxid = 0x2300000043
  mtime = Wed Aug 28 18:50:21 CST 2019
  pZxid = 0x2300000043
  cversion = 0
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 60
  numChildren = 0
  [zk: localhost:2181(CONNECTED) 11]
  ```

  



