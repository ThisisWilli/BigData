# Hive综合运用

## Hive DML语句，元数据服务？

* 删除表psn中的数据`hive> truncate table psn;`,动态分区进行建表

* 在表中插入数据`hive> load data inpath '/usr/age=10/data' into table psn;`

  ```sql
  create table psn10
  (
  id int,
  name string
  )
  row format delimited
  fields terminated by ',';
  
  create table psn11
  (
  id int,
  likes array<string>
  )
  row format delimited
  fields terminated by ','
  collection items terminated by '-';
  
  FROM psn
  INSERT OVERWRITE TABLE psn10
  SELECT id, name
  insert into psn11
  select id, likes;
  ```

* 分别查询数据

  ```sql
  hive> select * from psn10;
  OK
  1	小明1
  2	小明2
  3	小明3
  4	小明4
  5	小明5
  6	小明6
  7	小明7
  8	小明8
  9	小明9
  NULL	NULL
  Time taken: 0.049 seconds, Fetched: 10 row(s)
  hive> select * from psn11;
  OK
  1	["lol","book","movie"]
  2	["lol","book","movie"]
  3	["lol","book","movie"]
  4	["lol","book","movie"]
  5	["lol","movie"]
  6	["lol","book","movie"]
  7	["lol","book"]
  8	["lol","book"]
  9	["lol","book","movie"]
  NULL	NULL
  Time taken: 0.046 seconds, Fetched: 10 row(s)
  
  ```

  ```
  insert overwrite local directory '/root/result'
  select * from psn;
  ```

  执行完mr任务之后,在node04中查看数据

  ```
  [root@node04 result]# cat -A 000000_0 
  1^AM-eM-0M-^OM-fM-^XM-^N1^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  2^AM-eM-0M-^OM-fM-^XM-^N2^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  3^AM-eM-0M-^OM-fM-^XM-^N3^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  4^AM-eM-0M-^OM-fM-^XM-^N4^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  5^AM-eM-0M-^OM-fM-^XM-^N5^Alol^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  6^AM-eM-0M-^OM-fM-^XM-^N6^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  7^AM-eM-0M-^OM-fM-^XM-^N7^Alol^Bbook^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  8^AM-eM-0M-^OM-fM-^XM-^N8^Alol^Bbook^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  9^AM-eM-0M-^OM-fM-^XM-^N9^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong$
  \N^A\N^A\N^A\N$
  ```

## HIVE SerDe

* SerDe 用于做序列化和反序列化。
* 构建在数据存储和执行引擎之间，对两者实现解耦。
* Hive通过ROW FORMAT DELIMITED以及SERDE进行内容的读写。

### Hive正则匹配

创建一张新的数据表,包含正则表达式,并上传数据

```
 CREATE TABLE logtbl (
    host STRING,
    identity STRING,
    t_user STRING,
    time STRING,
    request STRING,
    referer STRING,
    agent STRING)
  ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
  WITH SERDEPROPERTIES (
    "input.regex" = "([^ ]*) ([^ ]*) ([^ ]*) \\[(.*)\\] \"(.*)\" (-|[0-9]*) (-|[0-9]*)"
  )
  STORED AS TEXTFILE;
```

```
hive> load data local inpath '/root/data/localhost' into table logtbl;
Loading data to table default.logtbl
Table default.logtbl stats: [numFiles=1, totalSize=1759]
OK
Time taken: 0.263 seconds
hive> select * from lo
load       local      locate(    location   log(       log10      log2       lower(     
hive> select * from logtbl;
OK
192.168.57.4	-	-	29/Feb/2016:18:14:35 +0800	GET /bg-upper.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:35 +0800	GET /bg-nav.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:35 +0800	GET /asf-logo.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:35 +0800	GET /bg-button.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:35 +0800	GET /bg-middle.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET / HTTP/1.1	200	11217
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET / HTTP/1.1	200	11217
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /tomcat.css HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /tomcat.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /asf-logo.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /bg-middle.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /bg-button.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /bg-nav.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /bg-upper.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET / HTTP/1.1	200	11217
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /tomcat.css HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /tomcat.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET / HTTP/1.1	200	11217
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /tomcat.css HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /tomcat.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /bg-button.png HTTP/1.1	304	-
192.168.57.4	-	-	29/Feb/2016:18:14:36 +0800	GET /bg-upper.png HTTP/1.1	304	-
Time taken: 0.052 seconds, Fetched: 22 row(s)
```

## Hive Beeline

* Beeline 要与HiveServer2配合使用

* 服务端启动hiveserver2

* 客户的通过beeline两种方式连接到hive
  * 1、beeline -u jdbc:hive2://localhost:10000/default -n root
  * 2、beeline 
  * beeline> !connect jdbc:hive2://<host>:<port>/<db>;auth=noSasl root 123
  
* 默认 用户名、密码不验证

### 启动HiveServer2

hiveserver2提供了jdbc的一种访问方式，在公司内推荐使用hiveserver2，web形式访问

* `[root@node03 ~]# hiveserver2`

* `[root@node04 ~]# beeline`

* 在node04中连接数据库，密码验证可以随便写，但是不能不写

  ```
  beeline> !connect jdbc:hive2://node03:10000/default root 123
  Connecting to jdbc:hive2://node03:10000/default
  Connected to: Apache Hive (version 1.2.1)
  Driver: Hive JDBC (version 1.2.1)
  Transaction isolation: TRANSACTION_REPEATABLE_READ
  0: jdbc:hive2://node03:10000/default> 
  ```

  

  