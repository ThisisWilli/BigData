# Hadoop复习

## Hadoop简介

### 解决什么问题

* 海量网页的存储
* 如何快速检索

### 如何解决

* GFS-HDFS
* MAPREDUCE-MAPREDUCE
* BIGTABLE-HBASE

### Hadoop组键

### Hadoop生态系统

![](pic\Hadoop生态系统.png)

* DAG有向无环图
* 再加一个kafka

## Hadoop的特点

### 安全性

* 多副本
* 但一个副本坏了之后，会自动拷贝其他副本

### 扩展性

* 可以扩展多个节点

### 处理海量数据

* GB，TB，PB
* 百万级别文件数
* 10k+节点数

### Hadoop的设计思想

* 分布式存储
* 分而治之，分布式并行计算
* 计算移向数据，目的：计算本地化

## HDFS

### Hadoop1.x架构

#### 架构图

![](F:\Java\BigData\Hadoop\pic\HDFS架构图.png)

#### 角色

##### NameNode

* 块的元数据
* 接受客户端的读写请求
* 获取block块的位置

##### Datanode

* 存储block
* 存储block副本
* 处理客户端读写块的请求

##### SecondaryNamenode

###### SSN是NN的冷备份

* 主要进行镜像文件和操作日志的合并
  * fsimage：元数据镜像文件
  * edits log：数据的操作日志
* SSN合并流程图
* SSN合并流程详解
  * 1.NN会把fsimage和edits文件进行**持久化**存储到磁盘，当触发checkpoint条件时，进行合并工作，，SSN开始工作
  * 2.NN会创建一个新的edits.new的文件，来记录新的操作日志
  * 3.SNN会见fsimage和edits文件加载到内存中，在内存中进行合并
  * 4.合并完成之后会生成一个新的fsimage.ckpt的文件
  * 5.SNN会把合并好之后的fsimage文件复制到NN，替换掉原来的fsimage以及edits
  * 6.之后一直循环往复
* checkpoint触发条件
  * 两次检查点的时间差在60分钟
  * 记录操作日志的eidts文件大小超过64M，会强制合并
* 注意：一般情况下，不要把SNN当作NN的备份，但是可以说成是冷备份
  * 冷备份：只是缓存一部分NN的数据，当服务宕机的时候，不能切换
  * 热备份：当服务宕机之后，可以随时切换

#####  jobtracker

* 负责管理集群中的资源
* 负责分配要执行的任务

##### tasktracker

实际工作的任务节点

####  存在问题

* NN单点故障
* NN压力过大，内存受限，无法扩展
* JobTracker压力过大，无法扩展
* 无法兼容除了MR之外的其他框架(storm,spark)



