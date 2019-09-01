# HBase复习

## HBase介绍

### HBase是数据库

* 不是内存数据库
* 不是列式数据库，KV格式的面向列存储的分布式数据库

### 特点

* 高可靠性
* 可伸缩性
* 面向列
* 实时读写
* 高性能

### 数据规模

* 十亿级别的行
* 百万级别的列

### 使用场景

* 非结构化或者半结构化的**松散数据**
* 高并发客户端请求的数据
* 要求实时性比较高的数据

## HBase原理结构

### HBase架构

#### 架构图

![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hbase/pic/HBase%E6%9E%B6%E6%9E%84.png](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hbase/pic/HBase架构.png)

#### 角色

##### client

* 提交hbase的读写请求，**并维护客户端的缓存**

##### master

* 管理整个集群的regionserver
* 分配region到哪一台regionserver
* 维护regionserver的负载均衡
* 负责为切分的region分配新的位置
* master应该是不存储数据的

##### zookeeper

* 保证每个集群任何时候都有唯一一台处理active的master
* 存储region元数据的所在节点信息(**所有region的寻址路口**)
* 存储table的相关信息
* 实时监控regionserver的上线下线信息，并且实时通知master

##### regionserver

###### hbase集群的从节点

* 负责存储client的读写请求的数据，负责存储region
* 但某一个region变大之后，regionserver负责等分成两个

###### 其他组件

* region
  * **相当于表的概念**
  * 当一个表创建完成之后，写数据的时候会先往一个region写数据
  * 当一个region的数据变大达到某一个阈值之后，会切分成两个
* store
  * 相当于列族：是实际存储数据的最小逻辑单元
  * **memstore：位于内存的空间**
  * storefile：位于磁盘的文件
  * 注意：memstore和storefile共同组成了LSM树结构

### 数据模型

#### 示意图

#### 角色

##### rowkey

*  唯一标识一行数据：相当于mysql数据的主键
* **rowkey默认是字典序**
* rowkey的最大存储长度是64K，一般建议10-100字节

##### column family

* 一组列的集合
* 必须作为表的schema预先给出
* 列族是权限控制，存储，属性设置的最小单元

##### qualifier

* 表示某一字段值，不需要余先给出
* 可以在插入值得时候，同时给列复制

##### timestamp

* 时间戳，起版本号的作用
* 一般使用系统的默认时间戳，不建议自定义

##### Cell

* 存储数据的最小单元
* 保存多个数据值
* 数据格式都是KV格式的数据
  * K：rowkey+column family+qualifier+timestamp
  * V: value
* cell中存储的数据都是字节数字类型

### HBase的读写流程

#### 读流程

* client发送请求给zk，从zk中获取到元数据存储的所在regionserver的节点
* 访问元数据所在的regionserver节点，访问表的元数据信息，找到表的存放位置
* 访问表所在的regionserver节点，找到对应的region，找到对应的store，从memstore中读取数据，如果读不到，从blockcache中读取数据
* 如果没有，去storefile中读取数据，并且将读取到的数据缓存到blockcache中
* 将结果**返回**给客户端

#### 写流程

* client发送请求给zk。从zk中获取到元数据存储的所在regionserver节点
* 访问元数据所在的regionserver节点，访问表的元数据信息，找到表的存放位置
* 访问表所在的regionserver几点，首先向WAL中进行写日志操作，完成之后，找到对应的region，向store的memstore写入数据
* 当面memstore中达到阈值(64M)之后，会进行**溢写**操作
* 存储到storefile，落地到磁盘，当磁盘的小文件过多的时候，会进行minor合并

### 存储结构

#### WAL

* 流程
  * 客户端发送请求之后，会先进行WAL写日志操作，预先存储到内存中
  * logAsync会监控内存空间的数据是否有修改，如果有的话，直接将修改操作落地到hdfs
  * logroller会将一段时间内的日志文件关闭，重新开启一个新的日志文件

#### memestore和storefile

* 使用了LSM树结构
  * 位于内存的小的存储结构
  * 位于磁盘的大的存储结构

## HBase API

### HBaseAdmin

* create table
* disable table
* delete table

### HTable

* get
* put
* scan
* delete

### FilterList

* SingleColumnValueFilter
* PrefixFilter
* 比较过滤器
* 专用过滤器
* 附加过滤器

## HBase CLL

* status
* who am i
* version
* create
* drop
* disable
* enable
* put
* get
* delete
* scan
* list
* flush
* split

## HBase-mapreduce

* MR从HBASE获取数据
* MR将数据结果存入HBASE
* 注意

## HBase优化

### 表设计

* 预分区
* rowkey的设计原则
  * 越短越好
  * hash
  * 取反
* 不要设置过多的列族，1-2个
* setMaxVersion
* SetTTL
* inMemory
* 合并和切分
  * 合并
    * minor：小范围合并(3-10个文件)，自动合并
    * major：将所有的storefile文件合并成一个：建议关闭自动合并，推荐手动合并
  * 切分：将一个region等分为两个region，分布在不同的regionserver上

### 写表操作

* 多客户端并发写，创建多个HTable对象
* 属性设置
  * 关闭autoFlush
  * 设置客户端写缓存
  * WAL Flag(不推荐关闭)
* 多线程写

### 读表操作

* 多客户端并发读：创建多个HTable对象
* 属性设置
  * 1、scan设置抓取数据的条数
  * 2、关闭resultScanner
  * 指定查询的column和family，不要将一整行的所有数据读取进来
* 多线程读
* blockCache
  * 一个regionserver共享一个blockcache
  * blockcache默认占用0.2的内存空间，对于注重读响应的系统，可以适当调大值
  * blockcache默认采用LRU的淘汰机制，默认淘汰最老的一批数据

### HTable和HTablePool

### HBase索引

* 跟ES做整合
* 步骤
  * 通过程序将Hbase中的数据读取存入到ES中
  * ES会创建索引，返回的值为rowkey
  * 当客户端请求进来之后，先去ES中查询获取rowkey的列表
  * 再根据rowkey去hbase中获取对应的数据









