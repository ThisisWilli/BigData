# Spark小文件合并

## Yarn中的运行组件

* ResourceManager(RM)：由Scheduler（资源调度器）和ApplicationsManager（ASM：应用管理器）2个组件组成。RM和每个NodeManager (NM)构成一个资源估算框架，管理协调分配集群中的资源，对集群中所有的资源拥有最终最高级别的仲裁权。
* ApplicationMaster(AM):主要与RM协商获取应用程序所需资源，实际资源都在NodeManager中，所以AM要与NM合作，在NM中运行任务，AM和mapreduce task都运行在Container中，**Container由RM调度，并由NM管理，监控所有task的运行情况**
* NodeManger:用来启动和监控本地计算机资源单位Container的启动情况，是每个节点上的资源和任务管理器，是RS在每台机器上的代理，负责容器的管理，定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态，并且接受并处理来自AM的Container启动/停止等请求。
* Container:Container是yarn资源的抽象，它封装了某个节点上的多维度资源（内存，cpu，磁盘，网络等），当AM向RM申请资源时，RM为AM返回的资源便是用 Container表示的。yarn会为每个任务分配一个Container，且该任务只能使用该Container描述的资源，它是一个动态资源划分单位，是根据应用程序的需求动态生成的。（目前yarn只支持cpu和内存2种资源）

## 分区

合理分区能让任务的task数量随着数据量的增大而增大，提高任务的并发度

### Hadoop中数据分片

* block size默认大小为128M，可以通过修改hdfs-site.xml文件中的dfs.blocksize对应的值，修改之前要提前关闭集群

## 方案

### 方案最终效果

​		启动spark-submit任务，根据传入的三个参数，实现根据hive表的文件格式反序列化read到spark、repartition重分区、合并小文件并压缩保存到临时目录、原始目录和临时目录数据条数校验、替换原目录里的文件等操作。三个参数如下：

* 1、HDFS文件系统路径开头（为了区分HA和非HA，所以要传入这个参数，比如hdfs://spark01:9000/）
* 2、hive表分区路径（比如：/data/cclog/t_h5_log_event/dt=20190101）
* 3、blocksize （比如120M，合并文件的阈值）

### 方案的具体实现

* 1、每天凌晨有个定时任务，关联hive元数据库的hive.SDS和hive.TBLS， 得到所有表和分区的文件存储格式，比如ORC、Parquet、Text等，然后保存到一个hive_format_type的匹配表中
     这个操作不涉及到实际业务表数据，只是hive database自身的元数据的查询操作。

  * hive.TBLS：该表中存储Hive表、视图、索引表的基本信息。

    | **元数据表字段**       | **说明**          | **示例数据**                                                 |
    | ---------------------- | ----------------- | ------------------------------------------------------------ |
    | **TBL_ID**             | 表ID              | 1                                                            |
    | **CREATE_TIME**        | 创建时间          | 1436317071                                                   |
    | **DB_ID**              | 数据库ID          | 2，对应DBS中的DB_ID                                          |
    | **LAST_ACCESS_TIME**   | 上次访问时间      | 1436317071                                                   |
    | **OWNER**              | 所有者            | liuxiaowen                                                   |
    | **RETENTION**          | 保留字段          | 0                                                            |
    | **SD_ID**              | 序列化配置信息    | 86，对应SDS表中的SD_ID                                       |
    | **TBL_NAME**           | 表名              | lxw1234                                                      |
    | **TBL_TYPE**           | 表类型            | MANAGED_TABLE、EXTERNAL_TABLE、INDEX_TABLE、VIRTUAL_VIEW     |
    | **VIEW_EXPANDED_TEXT** | 视图的详细HQL语句 | select `lxw1234`.`pt`, `lxw1234`.`pcid` from `liuxiaowen`.`lxw1234` |
    | **VIEW_ORIGINAL_TEXT** | 视图的原始HQL语句 | select * from lxw1234                                        |

  * hive.SDS

    ​		该表保存文件存储的基本信息，如INPUT_FORMAT、OUTPUT_FORMAT、是否压缩等。TBLS表中的SD_ID与该表关联，可以获取Hive表的存储信息。

    | **元数据表字段**              | **说明**         | **示例数据**                                               |
    | ----------------------------- | ---------------- | ---------------------------------------------------------- |
    | **SD_ID**                     | 存储信息ID       | 1                                                          |
    | **CD_ID**                     | 字段信息ID       | 21，对应CDS表                                              |
    | **INPUT_FORMAT**              | 文件输入格式     | org.apache.hadoop.mapred.TextInputFormat                   |
    | **IS_COMPRESSED**             | 是否压缩         | 0                                                          |
    | **IS_STOREDASSUBDIRECTORIES** | 是否以子目录存储 | 0                                                          |
    | **LOCATION**                  | HDFS路径         | hdfs://namenode/hivedata/warehouse/ut.db/t_lxw             |
    | **NUM_BUCKETS**               | 分桶数量         | 5                                                          |
    | **OUTPUT_FORMAT**             | 文件输出格式     | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat |
    | **SERDE_ID**                  | 序列化类ID       | 3，对应SERDES表                                            |

* 2、小文件合并主体功能

  * 1、根据传入的表分区路径和blocksize，在合并前优先判断该目录需不需要合并，思路是：
          如果设置的碎片阈值是120M，那么只会将该表分区内小于该阈值的文件进行合并，同时如果碎片文件数量小于一定阈值，默认设置的是5个，将不会触发合并，程序退出，这里主要考虑的是合并任务存在一定性能开销，因此允许系统中允许存在一定量的小文件。
  * 2、根据传入的第二个参数hive表分区路径，首先去匹配表hive_format_type里拿到该表路径对应的文件格式类型
  * 3、通过指定文件格式类型执行spark read操作，将分区下符合小文件阈值的文件全部读到DataFrame，可以避免读到大文件，同时可以通过count算子得到原始小文件的总条数和read耗时，此操作不影响业务
  * 4、根据hive表分区路径和第三个参数blocksize，将可以得到该目录的HDFS目录总空间并且算出合并后的partition个数，即合并后的文件个数=目录总大小/120M 取整+1
  * 5、DataFrame调用spark repartition，并指定合并的文件个数，将数据重写并压缩到合并临时目录，此操作不影响业务
  * 6、对合并后的临时目录重新执行spark read，再次得到新的条数和耗时，将两次的条数对比，如果相同则判定校验通过，并将耗时相差所得记录到mysql表，用于后期的grafana展示对应分区目录的读取提升
  * 7、判定校验通过后，将此前被标记的小文件移动到备份目录，并将合并目录下合并文件移动到原目录下，此操作类似hbase大合并完成后的动作，是瞬间完成的，对已经读到原来小文件数据的container不会有影响
          对读到一半小文件数据的container会报小文件找不到错误，待container失败后，重试即可读到新的合并文件了。可以只合并半年前的数据以将影响降到最低。文件合并最好在晚上7点后，这样影响最小，对于flink checkpoint 目录文件不做合并
  * 8、备份数据目录保留3-7天（可调），将会有定时任务自动扫描删除超时的表分区目录

  