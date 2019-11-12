# Spark简介

## Spark生态系统

![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E7%94%9F%E6%80%81%E7%B3%BB%E7%BB%9F.png )

* 底层的Mesos可以理解为Yarn，spark要基于一个资源调用框架才能运行
* Tachyon时基于内存的存储系统，HDFS是基于磁盘的存储系统
* 一般查数据是基于业务，数仓是基于数据分析

## Spark运行模式

* Local：多用于本地测试，如在eclipse，idea中写程序测试等。
* Standalone：Standalone是Spark自带的一个资源调度框架，它支持完全分布式，**就是脱离Yarn**
* Yarn:Hadoop生态圈里面的一个资源调度框架，Spark也是可以基于Yarn来计算的。
* Mesos:资源调度框架，国外用的多，不需要掌握
* 要基于Yarn来进行资源调度，必须实现AppalicationMaster接口，Spark实现了这个接口，所以可以基于Yarn。