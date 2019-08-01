# MapReduce

## MapReduce架构图

![](pic\MapReduce架构图.png)

## 分布式节点运算流转

![](pic\MapTask和ReduceTask.png)

* 缓存区存满之后一次性向外发出
* 在缓存区中先按照分区进行排序，几个reduce就有几个分区
* 相同的key调一个reduce方法
* combine
* split，默认与块的大小一致（也可比块少），逻辑上的片决定了map片的数量
* 进行（k，v）映射，大的k,v向缓存行中去写

## 计算框架

![](pic\计算框架MRwordcount单词统计.png)

* Map：
   * 读懂数据
   * 映射为KV模型
   * 并行分布式
   * 计算向数据移动
* Reduce：
   * 数据全量/分量加工（partition/group）全量指在reduce端可以处理一组或多组相同数据
   * Reduce中可以包含不同的key
   * 相同的Key汇聚到一个Reduce中
   * 相同的Key调用一次reduce方法
     * 排序实现key的汇聚

* K,V使用自定义数据类型，k,v必须为封装类
  * 作为参数传递，节省开发成本，提高程序自由度
  * Writable序列化：使能分布式程序数据交互
  * Comparable比较器：实现具体排序（字典序，数值序等）

## Hadoop1.X与mr1.X

* 运行架构：体现计算向数据移动

  ![](pic\Hadoop1与mr1.png)

MRv1角色：

* JobTracker

  * 核心，主，单点

  * 调度所有的作业

  * 监控整个集群的资源负载
* TaskTracker

  * 从，自身节点资源管理，按照jobtracker指令在所在节点开启maptask或者reducetask

  * 和JobTracker心跳，汇报资源，获取Task
* Client

  * 作业为单位

  * 规划作业计算分布

  * 提交作业资源到HDFS

  * 最终提交作业到JobTracker
* 弊端(MRv1)：
  * JobTracker：负载过重，单点故障
  * 资源管理与计算调度强耦合，其他计算框架需要重复实现资源管理(同时起一个storm和hadoop会造成冲突)
  * 不同框架对资源不能全局管理 

## Hadoop YARN (MRv2)

![](pic\hadoopyarnMRv2.png)

* YARN：解耦资源与计算
  * ResourceManager
    * 主，核心
    * 集群节点资源管理
  * NodeManager(一直向ResourceManager汇报)
    * 与RM汇报资源
    * 管理Container生命周期
    * 计算框架中的角色都以Container表示，一切皆container
  * Container：[封装了节点的多维度资源，如节点NM，CPU,MEM,I/O大小，启动命令]
    * 默认NodeManager启动线程监控Container大小，超出申请资源额度，kill
    * 支持Linux内核的Cgroup
* MR ：
  * MR-ApplicationMaster-Container
    * 作业为单位，避免单点故障，负载到不同的节点
    * 创建Task需要和RM申请资源（Container  /MR 1024MB）
  * Task-Container
* Client：
  * RM-Client：请求资源创建AM
  * AM-Client：与AM交互

* yarn指resource manage和nodemanage
* zookeeper为协调框架，Yarn为资源管控框架

## YARN

* YARN：Yet Another Resource Negotiator；
* Hadoop 2.0新引入的资源管理系统，直接从MRv1演化而来的；
  * 核心思想：将MRv1中JobTracker的**资源管理和任务调度两个功能分开**，分别由ResourceManager和ApplicationMaster进程实现
  * ResourceManager：负责整个集群的资源管理和调度
  * ApplicationMaster：负责应用程序相关的事务，比如任务调度、任务监控和容错等
* YARN的引入，使得多个计算框架可运行在一个集群中
  * 每个应用程序对应一个ApplicationMaster
  * 目前多个计算框架可以运行在YARN上，比如MapReduce、Spark、Storm等

## MapReduce On YARN

* 将MapReduce作业直接运行在YARN上，而不是由JobTracker和TaskTracker构建的MRv1系统
* 基本功能模块
  * YARN：负责资源管理和调度
  * MRAppMaster：负责任务切分、任务调度、任务监控和容错等
  * MapTask/ReduceTask：任务驱动引擎，与MRv1一致
* 每个MapRaduce作业对应一个MRAppMaster
  * MRAppMaster任务调度
  * YARN将资源分配给MRAppMaster
  * MRAppMaster进一步将资源分配给内部的任务
* MRAppMaster容错
  * 失败后，由YARN重新启动
  * 任务失败后，MRAppMaster重新申请资源