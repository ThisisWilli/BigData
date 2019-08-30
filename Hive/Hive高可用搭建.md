# Hive高可用搭建搭建

## 节点配置

|        | NN-1 | NN-2 | DN   | ZK   | ZKFC | JNN  | RS   | NM   | Hiverserver2 | beeline |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------------ | ------- |
| Node01 | *    |      |      |      | *    | *    |      |      |              |         |
| Node02 |      | *    | *    | *    | *    | *    |      | *    | *            |         |
| Node03 |      |      | *    | *    |      | *    | *    | *    | *            |         |
| Node04 |      |      | *    | *    |      |      | *    | *    |              | *       |

## 配置高可用

* 先配置两台hiveserver2

* 在node02中配置hive-site.xml

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
    <property>
          <name>hive.server2.support.dynamic.service.discovery</name>
          <value>true</value>
    </property>
    <property>
          <name>hive.server2.zookeeper.namespace</name>
          <value>hiveserver2</value>
    </property>
    <property>
          <name>hive.zookeeper.quorum</name>
          <value>node02:2181,node03:2181,node04:2181</value>
    </property>
    <property>
          <name>hive.zookeeper.client.port</name>
          <value>2181</value>
    </property>
    <property>
          <name>hive.server2.thrift.bind.host</name>
          <value>node02</value>
    </property>
    <property>
          <name>hive.server2.thrift.port</name>
          <value>10001</value>
   </property>
  </configuration>
  ```

* 在node03中`[root@node03 conf]# vi hive-site.xml`

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
    <property>
          <name>hive.server2.support.dynamic.service.discovery</name>
          <value>true</value>
    </property>
    <property>
          <name>hive.server2.zookeeper.namespace</name>
          <value>hiveserver2_zk</value>
    </property>
    <property>
          <name>hive.zookeeper.quorum</name>
          <value>node02:2181,node03:2181,node04:2181</value>
    </property>
    <property>
    <name>hive.zookeeper.client.port</name>
          <value>2181</value>
    </property>
    <property>
          <name>hive.server2.thrift.bind.host</name>
          <value>node03</value>
    </property>
    <property>
          <name>hive.server2.thrift.port</name>
          <value>10001</value> 
    </property>
  
  </configuration>
  
  ```

* 启动zookeeper的服务端`[root@node03 conf]# zkCli.sh`

* 创建新的节点,注意要和上面的配置文件中的相同`[zk: localhost:2181(CONNECTED) 5] create /hiverserver2 hiverserver2`

