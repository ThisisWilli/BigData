# Spark shuffle两种Manager

## 概念

* 1.6中有hashshufflemanager和1.2中就有的sortshufflemanager
* 2.3之后就只有sortshuffleManager

## shuffle类型

### 最初的HashShuffleManager

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E6%9C%80%E5%88%9D%E7%9A%84HashShuffleManager.png )

#### 特点

* Executor中的task串行执行，因为一次只能执行一个task
* 中间的任务是shuffle
* 每个buffer大小为32k，是在内存中的
* 产生磁盘小文件的数量为M*R，m代表maptask，R代表reducetask

#### 缺点

* 网络连接多，容易导致网络波动，进而导致task拉取数据失败
* OOM，读写文件以及缓存过多
* 小文件过多，低效的IO操作

### 优化后的HashShuffleManager

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E4%BC%98%E5%8C%96%E5%90%8E%E7%9A%84HashShuffleManager.png )

#### 特点

* 假如一个core中有1000个task，那么1000个task公用一个buffer内存，减少了磁盘小文件的数量
* 产生的磁盘小文件的数量为C*R，core为核
* 虽然相对第一中方法小文件数量减少了很多，但总体上来说，还是很多

### SortShuffle

#### 普通机制

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SortShuffle%E6%99%AE%E9%80%9A%E6%9C%BA%E5%88%B6.png )

* 上下两个task为map和reduce task
* 先申请一定大小的内存，随着数据量的增大，如果executor中内存足够，则再申请一定大小的内存，如果达到一定的阈值后开始溢写文件
* 一个maptask最终形成了两个磁盘小文件，数量很少

#### bypasss机制

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/SorShuffle%20bypass%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6.png )

* 少了排序的过程
* sortbykey， repartition不需要排序