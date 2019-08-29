# idea连接HBase

## 准备工作

* 启动集群

* 新建maven项目，配置相关依赖

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>HbaseOperation</groupId>
      <artifactId>HbaseOperation</artifactId>
      <version>1.0-SNAPSHOT</version>
      <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <hbase.version>0.98.12</hbase.version>
      </properties>
      <dependencies>
          <!--hbase-->
          <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
          <dependency>
              <groupId>org.apache.hbase</groupId>
              <artifactId>hbase-client</artifactId>
              <version>0.98.12-hadoop2</version>
          </dependency>
          <dependency>
              <groupId>org.apache.hbase</groupId>
              <artifactId>hbase-common</artifactId>
              <version>0.98.12-hadoop2</version>
          </dependency>
          <!--要使用HBase的MapReduce API-->
          <dependency>
              <groupId>org.apache.hbase</groupId>
              <artifactId>hbase-server</artifactId>
              <version>0.98.12-hadoop2</version>
          </dependency>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>5.1.24</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/junit/junit -->
          <!-- https://mvnrepository.com/artifact/junit/junit -->
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.8</version>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.11</version>
              <scope>compile</scope>
          </dependency>
  
  
      </dependencies>
  </project>
  ```

## 开始测试

* 编写相关测试类，全部代码如下：

  ```java
  package com.bd.hbase;
  
  import org.apache.hadoop.conf.Configuration;
  import org.apache.hadoop.hbase.*;
  import org.apache.hadoop.hbase.client.*;
  import org.apache.hadoop.hbase.util.Bytes;
  import org.junit.After;
  import org.junit.Before;
  import org.junit.Test;
  
  import java.io.IOException;
  import java.io.InterruptedIOException;
  
  /**
   * \* Project: HbaseOperation
   * \* Package: com.bd.hbase
   * \* Author: Hoodie_Willi
   * \* Date: 2019-08-29 10:45:30
   * \* Description:
   * \
   */
  public class HBaseDemo {
  
      //表管理类
      HBaseAdmin admin = null;
      //数据管理类
      HTable table = null;
      //表名
      String tm = "phone";
  
      /**
       * 完成初始化
       * @throws IOException
       */
      @Before
      public void init() throws IOException {
          Configuration conf = new Configuration();
          conf.set("hbase.zookeeper.quorum", "node02, node03, node04");
          admin = new HBaseAdmin(conf);
          table = new HTable(conf, tm.getBytes());
      }
  
      /**
       * 创建表
       * @throws IOException
       */
      @Test
      public void createTable() throws IOException {
          //表的描述类
          HTableDescriptor desc = new HTableDescriptor(TableName.valueOf(tm));
          //列族
          HColumnDescriptor family = new HColumnDescriptor("cf".getBytes());
          desc.addFamily(family);
          if (admin.tableExists(tm)){
              admin.disableTable(tm);
              admin.deleteTable(tm);
          }
          admin.createTable(desc);
      }
  
      @Test
      public void insert() throws InterruptedIOException, RetriesExhaustedWithDetailsException {
          Put put = new Put("1111".getBytes());
          put.add("cf".getBytes(), "name".getBytes(), "zhangsan".getBytes());
          put.add("cf".getBytes(), "age".getBytes(), "zhangsan".getBytes());
          put.add("cf".getBytes(), "sex".getBytes(), "zhangsan".getBytes());
          table.put(put);
      }
  
      @Test
      public void get() throws IOException {
          //要注意io，避免io过大，增加过滤操作
          Get get = new Get("1111".getBytes());
          //添加要获取的列和列族，减少网络的io，相当于在服务器端做了过滤
          get.addColumn("cf".getBytes(), "name".getBytes());
          get.addColumn("cf".getBytes(), "age".getBytes());
          get.addColumn("cf".getBytes(), "sex".getBytes());
  
          Result result = table.get(get);
          Cell cell1 = result.getColumnLatestCell("cf".getBytes(), "name".getBytes());
          Cell cell2 = result.getColumnLatestCell("cf".getBytes(), "age".getBytes());
          Cell cell3 = result.getColumnLatestCell("cf".getBytes(), "sex".getBytes());
          System.out.println(Bytes.toString(CellUtil.cloneValue(cell1)));
          System.out.println(Bytes.toString(CellUtil.cloneValue(cell2)));
          System.out.println(Bytes.toString(CellUtil.cloneValue(cell3)));
      }
  
      @Test
      public void scan() throws IOException {
          Scan scan = new Scan();
          //scan.setStartRow(star);
          //scan.setStopRow();
          ResultScanner rss = table.getScanner(scan);
          for (Result result : rss) {
              Cell cell1 = result.getColumnLatestCell("cf".getBytes(), "name".getBytes());
              Cell cell2 = result.getColumnLatestCell("cf".getBytes(), "age".getBytes());
              Cell cell3 = result.getColumnLatestCell("cf".getBytes(), "sex".getBytes());
              System.out.println(Bytes.toString(CellUtil.cloneValue(cell1)));
              System.out.println(Bytes.toString(CellUtil.cloneValue(cell2)));
              System.out.println(Bytes.toString(CellUtil.cloneValue(cell3)));
          }
      }
  
      /**
       *
       * @throws IOException
       */
      @After
      public void destory() throws IOException {
          if (admin != null){
              admin.close();
          }
      }
  }
  ```

### 准备配置文件

不需要将节点中的配置文件放入本地，只需编写函数

```java
@Before
    public void init() throws IOException {
        Configuration conf = new Configuration();
        conf.set("hbase.zookeeper.quorum", "node02, node03, node04");
        admin = new HBaseAdmin(conf);
        table = new HTable(conf, tm.getBytes());
    }
```

### 创建一张新表

```java
String tm = "phone";
    /**
     * 创建表
     * @throws IOException
     */
    @Test
    public void createTable() throws IOException {
        //表的描述类
        HTableDescriptor desc = new HTableDescriptor(TableName.valueOf(tm));
        //列族
        HColumnDescriptor family = new HColumnDescriptor("cf".getBytes());
        desc.addFamily(family);
        if (admin.tableExists(tm)){
            //先disable再删除
            admin.disableTable(tm);
            admin.deleteTable(tm);
        }
        admin.createTable(desc);
    }

```

在hbase shell中查看表，可以看到插入成功

```
hbase(main):001:0> list
TABLE                                                                                                                                                 
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/hbase/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/sxt/hadoop-2.6.5/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
phone                                                                                                                                                 
psn                                                                                                                                                   
tbl                                                                                                                                             
3 row(s) in 1.9200 seconds
```

### 向表中插入数据

```java
    @Test
    public void insert() throws InterruptedIOException, RetriesExhaustedWithDetailsException {
        //put ’<table name>’,’row_key’,’<colfamily:colname>’,’<value>’
        Put put = new Put("1111".getBytes());
        put.add("cf".getBytes(), "name".getBytes(), "zhangsan".getBytes());
        put.add("cf".getBytes(), "age".getBytes(), "zhangsan".getBytes());
        put.add("cf".getBytes(), "sex".getBytes(), "zhangsan".getBytes());
        table.put(put);
    }
```

在hbase shell中查看，插入成功

```
hbase(main):002:0> scan 'phone'
ROW                                    COLUMN+CELL                                                                                                    
 1111                                  column=cf:age, timestamp=1567048238877, value=zhangsan                                                         
 1111                                  column=cf:name, timestamp=1567048238877, value=zhangsan                                                        
 1111                                  column=cf:sex, timestamp=1567048238877, value=zhangsan                                                         
1 row(s) in 0.1400 seconds
```

### 查看数据

* get方法

  ```java
  @Test
      public void get() throws IOException {
          //要注意io，避免io过大，增加过滤操作
          Get get = new Get("1111".getBytes());
          //添加要获取的列和列族，减少网络的io，相当于在服务器端做了过滤
          get.addColumn("cf".getBytes(), "name".getBytes());
          get.addColumn("cf".getBytes(), "age".getBytes());
          get.addColumn("cf".getBytes(), "sex".getBytes());
  
          Result result = table.get(get);
          Cell cell1 = result.getColumnLatestCell("cf".getBytes(), "name".getBytes());
          Cell cell2 = result.getColumnLatestCell("cf".getBytes(), "age".getBytes());
          Cell cell3 = result.getColumnLatestCell("cf".getBytes(), "sex".getBytes());
          System.out.println(Bytes.toString(CellUtil.cloneValue(cell1)));
          System.out.println(Bytes.toString(CellUtil.cloneValue(cell2)));
          System.out.println(Bytes.toString(CellUtil.cloneValue(cell3)));
      }
  ```

  查询成功

  ![](pic\get方法查询成功.PNG)

* scan方法

  现在hbase shell中插入一条新数据

  ```
  hbase(main):003:0> put 'phone', '2222', 'cf:name', 'lisi'
  0 row(s) in 0.1600 seconds
  
  hbase(main):004:0> put 'phone', '2222', 'cf:age', '13'
  0 row(s) in 0.0050 seconds
  
  hbase(main):005:0> put 'phone', '2222', 'cf:sex', 'man'
  0 row(s) in 0.0310 seconds
  
  ```

  ```java
   @Test
      public void scan() throws IOException {
          Scan scan = new Scan();
          //scan.setStartRow(star);
          //scan.setStopRow();
          ResultScanner rss = table.getScanner(scan);
          for (Result result : rss) {
              Cell cell1 = result.getColumnLatestCell("cf".getBytes(), "name".getBytes());
              Cell cell2 = result.getColumnLatestCell("cf".getBytes(), "age".getBytes());
              Cell cell3 = result.getColumnLatestCell("cf".getBytes(), "sex".getBytes());
              System.out.println(Bytes.toString(CellUtil.cloneValue(cell1)));
              System.out.println(Bytes.toString(CellUtil.cloneValue(cell2)));
              System.out.println(Bytes.toString(CellUtil.cloneValue(cell3)));
          }
      }
  ```

  查询成功

  ![](pic\scan方法查询成功.PNG)



