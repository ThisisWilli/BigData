# Spark任务执行

## Spark RDD的宽窄依赖

###  前期总结

* 每一个application都有自己独立的driver，每个application都有自己独立的executor
* stage是由一组并行的task组成的

###  概念图

![](pic\RDD的宽窄依赖.png)

![](pic\RDD款窄依赖示意图.png)
### 特点
* 窄依赖：父RDD与子RDD partition之间的关系是一对一，父RDD与子RDD之间的关系是多对一
* 宽依赖(shuffle)：父RDD与子RDD partition之间的关系是一对多，涉及到节点之间数据的传输

## Spark计算模式

### 概念图

![](pic\Spark计算模式.png)

### 特点

* Spark处理数据的模式:pipeline管道处理模式(在迭代过程中发挥威力):`f3(f2(f1(textFile)))`  `1+1+1=3`
* 一个task其实是处理一连串分区的数据
* 如果两个rdd之间是窄依赖,那么它们可以合称为一个分区,当遇到宽依赖时,则划分出一个stage
* stage中国的并行度由谁决定:
  * 由stage中finalRDD的partition个数决定
* pipeline中的数据何时落地:
  * 1)shuffle write时落地
  * 2)对RDD持久化时
* 如何提高Stage的并行度
  * `reduceByKey(xxx, numpartition)`
  * `join(xx, numpartition)`
  * `distinct`

