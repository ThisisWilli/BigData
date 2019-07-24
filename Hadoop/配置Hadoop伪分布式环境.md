# 配置伪分布式环境

* 先安装Java

  * ``` rpm -i jdk-7u67-linux-x64.rpm  ```
  * 配置java环境变量
    * ``` vi + /etc/profile ``` 在末尾加上 ``` export JAVA_HOME=/usr/java/jdk1.7.0_67```和``` export PATH=$PATH:$JAVA_HOME/bin```
    * ```.   /etc/profile ```           
    * jps验证有无安装成功

* 配置密钥

  * 先使用``` ssh localhost ```连接本机
  * ``` cd ~ ``` , ``` ll -a ``` 查看.ssh文件
  * ``` ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa``` 
  * ``` cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys ```

* 安装hadoop，并配置hadoop环境变量

  * ``` tar xf hadoop-2.6.5.tar.gz -C /opt/sxt/ ```
  * ``` vi + /etc/profile ``` 在末尾加上 ``` export HADOOP_HOME=/opt/sxt/hadoop-2.6.5``` 和 ``` $HADOOP_HOME/bin:$HADOOP_HOME/sbin```
  * ``` source /etc/profile ```  

* 配置hadoop的环境

  * ``` cd /opt/sxt/hadoop-2.6.5/etc/hadoop```  修改文件夹中的hadoop-env.sh
  * ``` vi hadoop-env.sh,mapred-env.sh,mapred-env.sh``` 配置环境变量``` export JAVA_HOME=/usr/java/jdk1.7.0_67```

* 配置configuration

  * ``` root@node01 hadoop]# vi core-site.xml ``` 

  *  localhost改为node01

    ```xml
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost:9000</value>
        </property>
    </configuration>
    ```

  * 配置副本数，伪分布式只需1个副本

    ```xml
    <configuration>    
    	<property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
    </configuration>
    ```

    

* 配置从节点(datanode)

  * ``` vi slaves ``` 改为node01

* 配置seconddatanode

  * ```vi hdfs-site.xml```

  * 将配置文件改为 50090为端口号，namenode端口为9000

    ``` xml
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>node01:50090</value>
        </property>
    </configuration>
    
    ```

* 配置DataNode，NameNode临时文件的存放地址

  * ``` root@node01 hadoop]# vi core-site.xml ``` 

  * ```xml
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://node01:9000</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/var/sxt/hadoop/pseudo</value>
        </property>
    </configuration>
    ```

    

* 格式化NameNode

  * ``` [root@node01 hadoop]# hdfs namenode -format```

* 启动集群

  * ``` [root@node01 hadoop-2.6.5]# start-dfs.sh```

  * 若成功则显示如下

    ```
    The authenticity of host 'node01 (192.168.68.31)' can't be established.
    RSA key fingerprint is 8f:fe:ef:51:b9:60:03:55:cc:8c:6c:33:a0:b0:6c:41.
    Are you sure you want to continue connecting (yes/no)? yes
    node01: Warning: Permanently added 'node01,192.168.68.31' (RSA) to the list of known hosts.
    node01: starting namenode, logging to /opt/sxt/hadoop-2.6.5/logs/hadoop-root-namenode-node01.out
    node01: starting datanode, logging to /opt/sxt/hadoop-2.6.5/logs/hadoop-root-datanode-node01.out
    Starting secondary namenodes [node01]
    node01: starting secondarynamenode, logging to /opt/sxt/hadoop-2.6.5/logs/hadoop-root-secondarynamenode
    -node01.out
    ```

  * ``` [root@node01 hadoop-2.6.5]# jps``` 检验

    ``` 
    1709 DataNode
    1604 NameNode
    1843 SecondaryNameNode
    1945 Jps
    ```

* 通过可视化网页查看节点信息

  * ``` [root@node01 hadoop-2.6.5]# ss -nal``` 寻找第一个端口号
  * 在浏览器中输入```node01:50070```

* 在dfs文件中创建文件夹

  * ``` [root@node01 hadoop-2.6.5]# hdfs dfs -mkdir -p /user/root```
  * ``` [root@node01 hadoop-2.6.5]# hdfs dfs -ls /```
  * 登录node01:50070通过Utilities查看dfs系统

* 向dfs上传文件

  * ``` [root@node01 software]# hdfs dfs -put hadoop-2.6.5.tar.gz /user/root```
  * ``` [root@node01 software]# cd /var/sxt/hadoop/pseudo/dfs/data/``` 查看文件在本地的存储位置

* 关闭集群

  * ``` stop-dfs.sh```