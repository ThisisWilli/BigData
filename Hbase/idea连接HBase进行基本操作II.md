# idea连接HBase并对数据进行操作

## 创建一张表

* 先创建一张表

  ```java
  @Test
      public void createTable() throws IOException {
          //表的描述类
          HTableDescriptor desc = new HTableDescriptor(TableName.valueOf(tm));
          //列族
          HColumnDescriptor family = new HColumnDescriptor("cf".getBytes());
          desc.addFamily(family);
          if (admin.tableExists(tm)){
              // 删除之前要先disable
              admin.disableTable(tm);
              admin.deleteTable(tm);
          }
          admin.createTable(desc);
      }
  ```

## 向表内插入数据

* 先写好辅助函数

  ```java
      /**
       * 产生时间
       * @param string
       * @return
       */
      private String getDate(String string){
          //月-日-小时-分钟-秒
          return string + String.format("%02d%02d%02d%02d%02d", r.nextInt(12) + 1,
                  r.nextInt(31), r.nextInt(24), r.nextInt(60), r.nextInt(60));
      }
  
      /**
       * 产生手机号后9位
       */
      Random r = new Random();
      SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddhhmmss");
      private String getPhone(String phonePrefix){
          return phonePrefix + String.format("%08d", r.nextInt(99999999));
      }
  ```

* 准备插入数据，假设10个人每年产生1000条通话记录

  ```java
  /**
       * 10个用户，每个用户每年产生1000条通话记录
       * dnum:对方手机号
       * type:类型：0主叫，1被叫
       * length：长度
       * data：时间
       */
      @Test
      public void insert() throws ParseException, InterruptedIOException, RetriesExhaustedWithDetailsException {
          List<Put> puts = new ArrayList<Put>();
          for (int i = 0; i < 10; i++){
              String phoneNumber = getPhone("158");
              for (int j = 0; j< 1000; j++){
                  // 属性
                  String dnum = getPhone("177");
                  String length = String.valueOf(r.nextInt(99)); // 产生[0, n)的随机数
                  String type = String.valueOf(r.nextInt(2)); // [0, 2)
                  String date = getDate("2019");
                  //rowkey设计
                  String rowkey = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse(date).getTime());
                  Put put = new Put(rowkey.getBytes());
                  put.add("cf".getBytes(), "dnum".getBytes(), dnum.getBytes());
                  put.add("cf".getBytes(), "length".getBytes(), length.getBytes());
                  put.add("cf".getBytes(), "type".getBytes(), type.getBytes());
                  put.add("cf".getBytes(), "date".getBytes(), date.getBytes());
                  puts.add(put);
              }
          }
          // 全部插入
          table.put(puts);
      }
  ```

* 执行Test单元，在hbase shell中查看数据，显示插入成功

  ![](pic\查看一万条插入数据.PNG)

## 查看数据

### 用scan方法实现

* 用到过滤器加快查询速度

  ```java
  /**
       * 查询每一个用户，所有的主叫电话
       * 条件：
       * 1.电话号码
       * 2.type=0
       */
      @Test
      public void scan2() throws IOException {
          // MUST_PASS_ONE只要scan的数据行符合其中一个filter就可以返回结果(但是必须扫描所有的filter)，
          //另外一种MUST_PASS_ALL必须所有的filter匹配通过才能返回数据行(但是只要有一个filter匹配没通过就算失败，后续的filter停止匹配)
          // 因为要查询所有主叫号码，所以选择MUST_PASS_ALL
          FilterList filters = new FilterList(FilterList.Operator.MUST_PASS_ALL);
          SingleColumnValueFilter filter1 = new SingleColumnValueFilter("cf".getBytes(), "type".getBytes(),
                  CompareFilter.CompareOp.EQUAL, "0".getBytes());
          PrefixFilter filter2 = new PrefixFilter("15891411775".getBytes());
          filters.addFilter(filter1);
          filters.addFilter(filter2);
          Scan scan = new Scan();
          scan.setFilter(filters);
          ResultScanner scanner = table.getScanner(scan);
          for (Result result : scanner){
              System.out.print(Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "dnum".getBytes()))));
              System.out.print("--" + Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "type".getBytes()))));
              System.out.print("--" + Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "date".getBytes()))));
              System.out.println("--" + Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "length".getBytes()))));
          }
          scanner.close();
      }
  ```

  查询成功

  ![](pic\scan2查询成功.PNG)

### 用get方法实现查询

```java
/**
     * 查询某一个用户3月份的所有通话记录
     * 条件：
     *  1.某一个用户
     *  2.时间
     */
    @Test
    public void scan1() throws ParseException, IOException {
        String phoneNumber = "15891411775";
        String startRow = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse("20190401000000").getTime());
        String stopRow = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse("20190301000000").getTime());
        Scan scan = new Scan();
        scan.setStartRow(startRow.getBytes());
        scan.setStopRow(stopRow.getBytes());
        table.getScanner(scan);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            System.out.print(Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "dnum".getBytes()))));
            System.out.print("--" + Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "type".getBytes()))));
            System.out.print("--" + Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "date".getBytes()))));
            System.out.println("--" + Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "length".getBytes()))));
        }
    }
```

查询成功

![](pic\scan1查询成功.PNG)