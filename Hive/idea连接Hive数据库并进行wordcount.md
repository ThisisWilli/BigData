# idea连接Hive数据库

## 集群中的准备工作

* 在node01中开启mysql服务`[root@node01 ~]# service mysqld start`
* 在node03中开启hive的元数据服务`[root@node03 ~]# hive --service metastore`
* 在node04中开启hiveserver2服务`[root@node04 ~]# hiveserver2`

## 在idea中配置hive数据库

* 首先确认hive版本，我使用的是hive2.1，那么就应该准备2版本的jar包，idea中自带的是hive3的连接jar包，与hive2不兼容，连接会报错，jar包如下图所示

  ![](pic/连接hive2所需jar包.PNG)

* 在idea的右边栏中找到database选项

  ![](pic\右边栏的database选项.PNG)

* 选择添加Hive

  ![](pic\在idea中添加hive.png)

* 现在drivers栏中选择Apache Hive配置驱动，将之前准备好的jar包导入

  ![](pic\配置Hive的驱动.PNG)

* 配置完之后再去配置data source中的Hive，配置完之后可测试连接，如果出现报错，先检查相关服务有没有全部开启，在检查hive版本与连接jar包是否匹配

  ![](pic\配置Hivedatasource.png)

* 连接成功之后，可查看hive中的数据表

  ![](pic\Hive连接成功示意图.png)

## 上传数据并建表

### 上传数据

数据如下所示

```
hello world hi
hi hell hadoop
hive hbase spark
hello hi
```

上传数据至hdfs`[root@node04 data]# hdfs dfs -put wc /usr/`

### 建表

先有数据后有表，所以建外部表

```sql
create external table world_count
(
ling string 
)
location '/usr/';
```

创建结果表

```
create table wc_result
(
word string,
ct int 
);
```

## 进行wordcount

在idea中的hive console中输入如下sql语句，先将world_count中的数据行转成列，再进行count

```sql
from (select explode(split(ling, ' ')) word from world_count) t
insert into wc_result
select word, count(word) group by word;
```

最后结果如下图所示

![](pic\wc结果.PNG)



