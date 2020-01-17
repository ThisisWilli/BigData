# Hive总结

## Hive原理架构

###  Hive简介

#### 数据库和数据仓库的区别

* 数据库用来支撑业务系统的交互式查询，hive用来做复杂的**离线业务查询**
* 数据库中的数据是维持更新的，hive会把历史数据多保留下来，会形成一个时间拉链数据( 拉链表是针对数据仓库设计中表存储数据的方式而定义的，顾名思义，所谓拉链，就是记录历史。记录一个事物从开始，一直到当前状态的所有变化的信息。)
* 数据库存储的是单一系统的数据，hive是将多种数据源的数据规整到一起

#### hive的本质是mapreduce

* 延时性高，不能做到很快的响应
* 是将HQL语句转换为mapreduce执行

### Hive的结构

#### 架构图

![](pic\Hive架构.png)

### Hive的角色服务

#### Driver

* 是一个hive进程提供对外访问服务
* 分类
  * hive：结构命令行的sql提交
  * hiveserver2：beeline或者jdbc的方式

#### Client

* CLI
* JDBC：通过roc的thrift方式进行访问
* Web UI：弃用

#### metastore

* 元数据：表示表的相关信息

### Hive搭建

搭建是按照元数据的存储和管理进行搭建的

#### 搭建方式

* 使用hive自带的内存数据库derby(不用)
* 使用单价的MySQL数据库，通过网络来访问元数据(使用较多)
* 使用远程元数据服务的方式，实现hive和关系型数据库的解耦(使用较多)

#### 基本属性

* hdfs path：hive表存储的hdfs的目录
* mysql属性
  * drivername
  * url
  * username
  * password
* hive.metastore.uris:通过9083端口访问元数据服务

#### 访问方式分类

* 1.服务端直接执行命令自己启动元数据服务
* 2.先开启一个元数据服务，服务端通过9083端口进行连接

#### 注意

* 企业推荐使用远程服务的模式访问hive
  * 1.先手动启动hive的元数据服务：hive --service metastore
  * 2.hive/hiveserver2都是通过9083端口进行元数据访问
  * 3.hive方式由开发人员使用
  * 4.hiveserver2只提供查询业务

## Hive DDL(Data Definition Language)

### 创建数据库

* `create database databaseName`

### 创建表

* 最原始的方法

  ```sql
  create table tablename(
  col dataType...
  )
  row format delimited 
  fields terminated by "某个符号"
  collection items terminated by "某个符号"
  map keys terminated by | serde(正则)
  ```

* 数据和表结构都用的情况

  ```sql
  create table tablename as select_statement 
  ```

* 只有表结构

  ```sql
  create table tablename like tablename 
  ```

### 分区

* 目的：提高数据的检索效率
* 展现形式：**在hdfs路径中包含了多级目录**
* 创建分区：在创建表的时候要制定`partitioned by(col type)`
* 分区类型
  * 静态分区：在分区表导入数据的时候，分区列的值为人为规定
  * 动态分区：通过表中某一个字段值将数据存放到指定目录
* 添加分区：`alter table tablename add partition(col=value)`(只适用于静态分区)动态分区自己家
* 删除分区：`alter table tablename drop partition(col-value)`(只适用于静态分区)
* 修复分区：当在hdfs中已经存在多级目录的文件的时候，创建外部表的时候需要修复分区(其实是向关系型数据库中添加元数据)`msck repair table tablename`

### 分桶

* 目的：抽样查询，事务支持
* 展示形式：将一个数据文件拆分成桶的个数文件(随机打散)
* 分桶原则：取其中某一列的值，将值取hash值，再将hash值%桶的个数
* 实现方式：创建表的时候指定`clustered by tablename(col) into number buckets`
* 抽样查询：`tablesample(bucket x out of y)`
  * x：从哪个桶开始取数据
  * y：桶的个数的倍数或者因子

### 视图

* Hive中也支持视图：`create view view_name as select_statement`
* 特点
  * 不支持物化视图
  * hive视图不能执行insert或者load操作
  * 支持迭代视图

### 索引

* 目的：提交检索效率
* 创建视图：`create index index_name on tablename(col) as '指定索引器' in the tablename(存储索引数据)`
* hive中的视图需要自己创建：`alter index index_name on tablename rebuild`

### Hive函数

#### Hive本身自带了很多内嵌函数

* 字符函数
* 数值函数
* 日期函数
* 复杂类型函数
* 条件函数
* 聚合函数
* UDTF函数

#### 函数的分类

* udf(一进一出)
* udaf(多进一出)
* udtf(一进多出)

#### 自定义函数

* 1.编写java代码继承UDF类
* 2.实现evaluate方法，所有实现的核心逻辑写到此方法中
* 3.将写好的代码打包成jar包
* 4.将jar包上传到本地linux或者hdfs
* 5.如果是本地linux，在Hive客户端执行`add jar path`
  * `create temporary function func_name as 'package+class'`
* 6.如果时hdfs，直接创建函数
  * `create function func_name as 'package+class' using jar 在hdfs中的路径`

### 内部表和外部表

#### 创建

* 内部表不需要指定数据存储的路径，直接将数据存储在默认的目录中
* 外部表需要使用external关键字指定，需要使用location指定存储数据的位置

#### 删除

* 内部表中的数据和元数据都是由hive来管理的，删除的时候全部删除
* 外部表的数据由hdfs来管理，元数据由hive管理，删除的时候只删除元数据，数据不会删除

## DML(Data Manipulation Language)

### 增

* `load data local inpath overwrite/into table tablename(partition)`
* `from ... insert overwrite tablename select...`
* `insert into tablename values()`
* `insert into local directory dic.. select -statement`
* 注意3，4几乎不用，1，2使用较多

### 删，改

* hive支持删除和修改数据，但是需要事务的支持
* 开启事务之后有很多限制
  * 1.不支持rollback，commit，begin
  * 必须是orc(按列存储文件)文件格式
  * 表必须被分桶
  * 事务默认不开启，需要手动开启

## Hive的运行

### Hive参数的设置

#### 参数的分类

* 1.hiveconf等同于设置在hive-site.xml文件中的属性
* 2.System
* 3.env
* 4.hivevar

#### 设置方式

* 1.在hive-site.xml中设置(全局生效)
* 2.启动命令行的时候添加(`--hivbeconf k=v`)
* 3.进入命令行之后`set k = v`
* 4.在当前用户的家目录下创建.hiverc的文件

### Hive的运行方式

#### cli

* 可以提交sql语句
* 与hdfs交互
* 与linux交互但是命令前要加！

#### 脚本运行方式

* `hive -e 'sql'`
* `hive -e 'sql' > aa,txt`
* `hive -S -e 'sql'`
* `hive -f file`
* `hive -i file`
* 在hive的命令行中执行`source file`

### 权限管理

#### 模式

* 1.基于元数据的权限管理
* 2.基于sql标准的权限管理(推荐)
* 3.基于第三方组建的权限管理
* 4.默认的管理方式

#### 角色

* 用户：使用者
* 角色：一组权限的集合

#### 默认角色

* public
* admin

#### 权限分配

* grant
* revoke

## Hive的优化









