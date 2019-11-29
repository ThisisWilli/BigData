# Spark Shuffle文件的寻址与内存管理

## Spark Shuffle文件寻址

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Shuffle%E6%96%87%E4%BB%B6%E5%AF%BB%E5%9D%80.jpg )

* BlockManagerMaster管理两个Worker中BlockManagerSlave
* 48M是可配置的
* oom问题出在绿线那里

## Spark 内存管理

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.jpg )

* 从上到下依次减少
* 虚线说明可以相互动态借用
* 把task整体运行的内存调大了