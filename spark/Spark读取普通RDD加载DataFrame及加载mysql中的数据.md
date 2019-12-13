# Spark读取普通RDD加载DataFrame及加载mysql中的数据

## 将RDD转化为DataFrame的方法

### 反射的方式

* 首先将RDD转换成自定义类型的RDD;2.rdd.toDF(),scala中的方法

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E5%B0%86RDD%E8%BD%AC%E5%8C%96%E4%B8%BADF.png )

* Spark1.6中JavaAPI的转换

  ![]( https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark1.6%E4%B8%ADJavaAPI%E7%9A%84RDD2DF_1.png )

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark1.6%E4%B8%ADJavaAPI%E7%9A%84RDD2DF_2.png)

### 动态创建Schema的方式

* 1.创建row类型的RDD
* 2.使用spark.createDataFrame(rowRDD, structType)映射成DataFrame

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E5%8A%A8%E6%80%81%E5%88%9B%E5%BB%BASchema.png )

![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/%E5%8A%A8%E6%80%81%E5%88%9B%E5%BB%BASchema_Java.png )

## 读取mysql数据的四种方式

* 1.

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E8%AF%BB%E5%8F%96mysql%E6%95%B0%E6%8D%AE%E7%AC%AC%E4%B8%80%E7%A7%8D%E6%96%B9%E5%BC%8F.png )

* 2.

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E8%AF%BB%E5%8F%96mysql%E6%95%B0%E6%8D%AE%E7%AC%AC%E4%BA%8C%E7%A7%8D%E6%96%B9%E5%BC%8F.png )

* 3.

  ![](https://willipic.oss-cn-hangzhou.aliyuncs.com/Spark/Spark%E8%AF%BB%E5%8F%96mysql%E6%95%B0%E6%8D%AE%E7%AC%AC%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F.png )
  
  ​	

