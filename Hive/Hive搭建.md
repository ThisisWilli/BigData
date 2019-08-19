# Hive搭建

## 搭建模式



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

* `[root@node01 ~]# mysql -uroot -p`

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

  ![](pic\在resourceManage节点中查看hive任务.PNG)

* 查看表中数据`hive> select * from tbl;`

  ```
  hive> select * from tbl;
  OK
  1	1
  Time taken: 0.097 seconds, Fetched: 1 row(s)
  ```

  

