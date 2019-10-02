# Spark-Submit提交参数

## Options

### master

* MASTER_URL, 可以是spark://host:port, mesos://host:port, yarn,  yarn-cluster,yarn-client, local

### deploy-mode

* DEPLOY_MODE, Driver程序运行的地方，client或者cluster,默认是client。

### class

* CLASS_NAME， 主类名称，含包名

### jars

* 逗号分割的本地JARS，Driver和executor依赖的第三方包

### files

* 用逗号隔开的文件列表,会放置在每个executor工作目录中

### conf

* spark的配置属性

### driver-memory

* Driver参加程序使用内存大小(例如：1000M， 5G)，默认1024M

### executor-memory

* 每个executor内存大小（如：1000M，2G），默认1G

## Spark standalone with cluster deploy mode only:

### driver-cores

* driver程序的使用core个数，默认为1，仅限于Spark Standalone模式

## Spark standalone or Mesos with cluster deploy mode only

### supervise

* 失败后是否重启Driver，仅限于Spark  alone或者Mesos模式

## Spark standalone and Mesos only:

### total-executor-cores

* executor使用的总核数，仅限于SparkStandalone、Spark on Mesos模式

## Spark standalone and YARN only

### executor-cores

* 每个executor使用的core数，Spark on Yarn默认为1，standalone默认为worker上所有可用的core。

## YARN-only

### driver-cores

* driver使用的core,仅在cluster模式下，默认为1。

### queue 

* QUEUE_NAME  指定资源队列的名称,默认：default

### num-executors

* 一共启动的executor数量，默认是2个。