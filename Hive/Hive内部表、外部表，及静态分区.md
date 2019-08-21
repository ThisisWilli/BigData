# Hive内部表、外部表，及静态分区

## 创建内部表

- 创建一张新表

  ```sql
  create table psn2
  (
  id int,
  name string,
  likes array<string>,
  address map<string, string>
  );
  ```

- 创建data2，注意内部存储分隔符`1^A小明1^Alol^Bbook^Bmovie^Abeijing^Cshangxuetang^Bshanghai^Cpudong`

- 在node04中插入数据

  ```
  hive> load data local inpath '/root/data/data2' into table psn2;
  Loading data to table default.psn2
  Table default.psn2 stats: [numFiles=1, totalSize=62]
  OK
  Time taken: 0.981 seconds
  hive> select * from psn2
      > ;
  OK
  1	小明1	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
  Time taken: 0.069 seconds, Fetched: 1 row(s)
  ```

- 在node04中创建新表psn3

  ```sql
  create table psn3
  (
  id int,
  name string,
  likes array<string>,
  address map<string, string>
  )
  row format delimited
  fields terminated by '\001'
  collection items terminated by '\002'
  map keys terminated by '\003';
  ```

- 和上面操作一样在node04中输入插入数据的命令，显示插入成功，说明'\001''\002''\003'(一般不推荐这么写)，与^A,^B,^C等价

  ## 创建外部表

- `[root@node04 data]# hdfs dfs -mkdir /usr`

- `[root@node04 data]# hdfs dfs -put data /usr` 上传文件到hdfs

- 创建外部表

  ```sql
  create external table psn4
  (
  id int,
  name string,
  likes array<string>,
  address map<string, string>
  )
  row format delimited
  fields terminated by ','
  collection items terminated by '-'
  map keys terminated by ':'
  location '/usr/';
  ```

## 内部表和外部表的区别，应用场景？

* 内部表数据由Hive自身管理，外部表数据由HDFS管理； 

* 内部表数据存储的位置是hive.metastore.warehouse.dir（默认：/user/hive/warehouse），外部表数据的存储位置由自己制定； 

* 删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS上的文件并不会被删除； 

* 对内部表的修改会将修改直接同步给元数据，而对外部表的表结构和分区进行修改，则需要修复（MSCK REPAIR TABLE table_name;）

* 创建表的时候，内部表直接存储在默认的hdfs路径，外部表需要自己指定路径

* 删除表的时候，内部表会将数据和元数据全部删除，外部表哦只删除元数据，数据不删除

  创建一张新表

```sql
create table psn
(
id int,
name string,
likes array<string>,
address map<string, string>
)
row format delimited
fields terminated by ','
collection items terminated by '-'
map keys terminated by ':';
```

将根目录中的install日志插入到表中

```
hive> load data local inpath '/root/install.log' into table psn;
Loading data to table default.psn
Table default.psn stats: [numFiles=1, totalSize=8815]
OK
Time taken: 0.265 seconds
```

- 注意：hive：读时检查（实现解耦，提高数据记载的效率）关系型数据库：写时检查

## Hive分区

### 建立单分区

- 创建一张带有分区的表格

  ```sql
  create table psn5
  (
  id int,
  name string,
  likes array<string>,
  address  map<string, string>
  )
  partitioned by(age int)
  row format delimited
  fields terminated by ','
  collection items terminated by '-'
  map keys terminated by ':';
  ```

- 在node04中向表中插入数据`hive> load data local inpath '/root/data/data' into table psn5 partition(age=10);`并在web端查看分区标志

  ![](F:/Java/BigData/Hive/pic/查看分区数据库.PNG)

### 建立双分区

创建带有双分区的表格

```sql
create table psn6
(
id int,
name string,
likes array<string>,
address  map<string, string>
)
partitioned by(age int,sex string)
row format delimited
fields terminated by ','
collection items terminated by '-'
map keys terminated by ':';
```

导入数据`hive> load data local inpath '/root/data/data' into table psn6 partition(age=20, sex='man');`并在web查看

![](F:/Java/BigData/Hive/pic/查看双分区数据库.PNG)

### 添加和删除分区

`hive> alter table psn6 add partition(sex='girl', age=30);`

`hive> alter table psn6 drop partition(sex='man');`

```
create external table psn7
(
id int,
name string,
likes array<string>,
address  map<string, string>
)
partitioned by(age int)
row format delimited
fields terminated by ','
collection items terminated by '-'
map keys terminated by ':';
```

分区属不属于元数据

