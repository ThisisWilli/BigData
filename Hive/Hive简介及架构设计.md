# Hive简介及架构设计

## Hive简介

官网：https://hive.apache.org/

* 数据仓库，数据仓库管理各种数据库中的数据，数据仓库中的数据不允许删除，不允许修改
* 解释器(SQL->YARN->MR)，编译器，优化器等。
* Hive 运行时，元数据()存储在关系型数据库里面

### Hive的产生

非java编程者对hdfs的数据做mapreduce操作，不能做实时分析，只能做离线的

## Hive架构

![](pic\Hive架构.png)

* 用户接口主要有三个：CLI(command line interface)，Client 和 WUI。其中最常用的是CLI，Cli启动的时候，会同时启动一个Hive副本。Client是Hive的客户端，用户连接至Hive Server。在启动 Client模式的时候，需要指出Hive Server所在节点，并且在该节点启动Hive Server。 WUI是通过浏览器访问Hive。
* Hive将元数据存储在数据库中，如mysql、derby(Hive自带内存型数据库)。Hive中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等。    
* 解释器、编译器、优化器完成HQL查询语句从词法分析、语法分析、编译、优化以及查询计划的生成。生成的查询计划存储在HDFS中，并在随后有MapReduce调用执行。
* Hive的数据存储在HDFS中，大部分的查询、计算由MapReduce完成（包含*的查询，比如select * from tbl不会生成MapRedcue任务）。
* driver为一个jvm进程
* Thrift Server 基于RPC协议

### Hive执行过程简图

![](pic\Hive架构图2.png)

![](pic\Hive操作符.png)

### Hive词法分析工具

![](pic\ANTLR词法语法分析工具解析hql.png)

## 