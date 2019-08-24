# Hive总结

##  Hive是数据仓库

### 数据库和数据仓库的区别

* 用到数据仓库：分析，决策类影响

## Hive搭建

搭建是按照元数据的存储和管理进行搭建的

### 搭建方式

* 使用hive自带的内存数据库derby(不用)
* 使用单价的MySQL数据库，通过网络来访问元数据(使用较多)
* 使用远程元数据服务的方式，实现hive和关系型数据库的解耦(使用较多)

## Hive DDL

### 创建数据库

* `create database databaseName`

### 创建表

* 最原始的方法

  ```
  create table tablename(
  col dataType...
  )
  row format dilimited 
  fields terminated by "collection items terminated by"map keys terminated by | serde(正则)
  ```

* 数据和表结构都用的情况

  ```
  create table tablename as select_statement 
  ```

* 只有表结构

  ```sql
  create table tablename like tablename 
  ```

### 内部表和外部表

#### 创建

* 内部表不需要指定数据存储的路径，直接将数据存储在默认的目录中
* 外部表需要使用external关键字指定，需要使用location指定存储数据的位置

#### 删除

* 内部表中的数据和元数据都是由hive来管理的，删除的时候全部删除
* 外部表的数据由hdfs来管理，元数据由hive管理，删除的时候只删除元数据，数据不会删除

## DML

### 增

* `load data local inpath overwrite/into table tablename(partition)`
* `from ... insert overwrite tablename select...`
* `insert into table values()`
* `insert into local directory dic.. select -statement`

### 删，改

想要使用删除和修改必须要经过事务，需要配置事务，主要为以下几点限制

* rollback，commit不支持
* 必须是orc的文件格式
* 表必须被分桶
* 默认事务是不开启的

## Hive的分区

* 目的：方便提高检索的效率
* 展现形式：在hdfs目录上创建多级目录

### Hive分区的分类

#### 静态分区

* 静态分区的值是人为指定的

#### 动态分区

* 分区列的值是有记录的某一列值来决定的

### 添加分区(只适用于静态分区)

* `alter table tablename add partition(col=val)`

### 修复分区

* 分区是作为元数据存储在MySQL中的，当hdfs路径中包含多级目录，同时存在分区列时，可以创建外部表使用，但是分区的元数据没有在MySQL中存储，查不到数据
* `msck repair table tablename`

## Hive函数

### Hive本身自带了很多内嵌函数

* 字符函数

* 数值函数

* 日期函数

* 复杂类型函数

* 条件函数

### 函数的分类

* udf(一进一出)
* udaf(多进一出)
* udtf(一进多出)

### 自定义函数

* 编写java代码继承UDF类
* 实现evaluate方法，所有实现的核心逻辑写到此方法中
* 将写好的代码打包成jar包
* 将jar包上传到本地linux或者hdfs
* 如果是本地linux，在Hive客户端执行`add jar path`
  * `create temporary function func_name as 'package+class'`
* 如果时hdfs，直接创建函数
  * `create function func_name as 'package+class' using jar 在hdfs中的路径`





