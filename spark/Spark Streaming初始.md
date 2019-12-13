# Spark Streaming初始

## Spark Streaming和storm的区别

* 1.Storm是纯实时处理数据，Spark Streaming微批处理数据，可以通过控制时间间隔做到实时处理
* 2.Storm擅长处理简单的汇总型业务，Spark Streaming擅长处理复杂的业务。Storm相对于Spark Streaming来说轻量级，Spark Streaming中可以使用core或者sql或者机器学习
* 3.Storm的事务与SparkStreaming不同，SparkStreaming可以管理事务
* 4.Storm支持动态的资源调度，Spark也是支持的

## Spark Streaming接受数据原理

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SparkStreaming%E6%8E%A5%E6%94%B6%E6%95%B0%E6%8D%AE%E5%8E%9F%E7%90%86.jpg )