# macos下编译Spark2.3.4源码

## 配置环境

* 下载Scala2.12

* 在github中下载Spark源码，clone或者直接下载都可以

  ![](pic/选择想要下载的Spark源码.png)

* 下载完之后配置java，spark，maven，scala环境变量如下`vim ~/.bash_profile`

  ![](pic/配置mac中的环境变量.png)

  配置完成之后记得`source .bash_profile`

## 开始编译

* 进入Spark的目录`mvn -DskipTests clean package`，如编译成功，则如下所示

  ![](pic/mvn编译Spark成功.png)

  

