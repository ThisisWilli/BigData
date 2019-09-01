# HBase protobuffer

## 为什么需要protobuffer

* 在节点中查看数据存放了几条`[root@node02 ~]# hbase hfile -p -f /hbase/data/default/phone/26967daae26389fa0625e28890e893d0/cf/d804f42d47a7487c872cdc5713706e3d`，部分数据如下

  ```
  K: 15891411775_9223370490502286807/cf:date/1567134466791/Put/vlen=14/mvcc=0 V: 20190101222129
  K: 15891411775_9223370490502286807/cf:dnum/1567134466791/Put/vlen=11/mvcc=0 V: 17728721897
  K: 15891411775_9223370490502286807/cf:length/1567134466791/Put/vlen=2/mvcc=0 V: 54
  K: 15891411775_9223370490502286807/cf:type/1567134466791/Put/vlen=1/mvcc=0 V: 0
  K: 15891411775_9223370490576485807/cf:date/1567134466791/Put/vlen=14/mvcc=0 V: 20190101014450
  K: 15891411775_9223370490576485807/cf:dnum/1567134466791/Put/vlen=11/mvcc=0 V: 17707560382
  K: 15891411775_9223370490576485807/cf:length/1567134466791/Put/vlen=1/mvcc=0 V: 0
  K: 15891411775_9223370490576485807/cf:type/1567134466791/Put/vlen=1/mvcc=0 V: 0
  K: 15891411775_9223370490591873807/cf:date/1567134466791/Put/vlen=14/mvcc=0 V: 20190100212822
  K: 15891411775_9223370490591873807/cf:dnum/1567134466791/Put/vlen=11/mvcc=0 V: 17707028673
  K: 15891411775_9223370490591873807/cf:length/1567134466791/Put/vlen=2/mvcc=0 V: 61
  K: 15891411775_9223370490591873807/cf:type/1567134466791/Put/vlen=1/mvcc=0 V: 0
  K: 15891411775_9223370490639998807/cf:date/1567134466791/Put/vlen=14/mvcc=0 V: 20190100080617
  K: 15891411775_9223370490639998807/cf:dnum/1567134466791/Put/vlen=11/mvcc=0 V: 17748223906
  K: 15891411775_9223370490639998807/cf:length/1567134466791/Put/vlen=2/mvcc=0 V: 78
  K: 15891411775_9223370490639998807/cf:type/1567134466791/Put/vlen=1/mvcc=0 V: 0
  K: 15891411775_9223370490666686807/cf:date/1567134466791/Put/vlen=14/mvcc=0 V: 20190100124129
  K: 15891411775_9223370490666686807/cf:dnum/1567134466791/Put/vlen=11/mvcc=0 V: 17714565023
  K: 15891411775_9223370490666686807/cf:length/1567134466791/Put/vlen=2/mvcc=0 V: 12
  K: 15891411775_9223370490666686807/cf:type/1567134466791/Put/vlen=1/mvcc=0 V: 1
  
  ```

  可见一个rowkey会存4条，为了节省空间，引入protobuffer，结构化存储数据

## 安装protobuffer

* `[root@node01 ~]# tar -zxvf protobuf-2.5.0.tar.gz`

* 安装依赖包`[root@node01 protobuf-2.5.0]# yum groupinstall "Development tools"`

* 编译`[root@node01 protobuf-2.5.0]# ./configure`

* 查看安装完之后的路径

  ```
  [root@node01 protobuf-2.5.0]# cd /usr/local/
  [root@node01 local]# ls
  bin  etc  games  include  lib  lib64  libexec  sbin  share  src
  [root@node01 local]# cd bin
  [root@node01 bin]# ls
  protoc
  ```


## 使用protobuffer

* 创建proto文件`[root@node01 ~]# vi phone.proto`,一定要用notepad++生成，不然会产生字符错误

  ```java
  package com.bd.hbase;
  message PhoneDetail
  {
          required string dnum = 1;
          required string length = 2;
          required string type = 3;
          required string date = 4;
  }
  ```

* 执行`[root@node01 ~]# /usr/local/bin/protoc phone.proto --java_out=/root/`

* 执行完成之后，查看执行目录，会产生一个com文件夹，一次搜寻，将其加入到项目的包中

* 删除之前的phone表`hbase(main):004:0> truncate 'phone'`

* 创建一张新的phone表

  ```java
  /**
       * 10个用户，每个用户1000条，每一条记录当作一个对象进行存储
       */
      @Test
      public void insert2() throws ParseException, InterruptedIOException, RetriesExhaustedWithDetailsException {
          List<Put> puts = new ArrayList<Put>();
          for (int i = 0; i < 10; i++){
              String phoneNumber = getPhone("158");
              for (int j = 0; j < 1000; j++){
                  String dnum = getPhone("177");
                  String length = String.valueOf(r.nextInt(99)); // 产生[0, n)的随机数
                  String type = String.valueOf(r.nextInt(2)); // [0, 2)
                  String date = getDate("2019");
  
                  // 保存到对象属性中
                  Phone.PhoneDetail.Builder phoneDetail = Phone.PhoneDetail.newBuilder();
                  phoneDetail.setDate(date);
                  phoneDetail.setLength(length);
                  phoneDetail.setType(type);
                  phoneDetail.setDnum(dnum);
  
                  //rowkey
                  String rowkey = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse(date).getTime());
                  Put put = new Put(rowkey.getBytes());
                  put.add("cf".getBytes(), "phoneDetail".getBytes(), phoneDetail.build().toByteArray());
                  puts.add(put);
  ;            }
          }
          table.put(puts);
      }
  ```

* 查看新表`hbase(main):004:0> scan 'phone'`

  ![](pic\查看proto格式存储的文件.PNG)

* 存储空间虽然变小但是只能通过rowkey查询数据

### 查询一条数据

* 新建方法

  ```java
  @Test
      public void get() throws IOException {
          Get get = new Get("15871044379_9223370490668637807".getBytes());
          table.get(get);
          Result result = table.get(get);
          Phone.PhoneDetail phoneDetail = Phone.PhoneDetail.parseFrom(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(),
                  "phoneDetail".getBytes())));
          System.out.println(phoneDetail);
      }
  ```

  结果如下

  ```
  log4j:WARN Please initialize the log4j system properly.
  log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
  dnum: "17770899722"
  length: "20"
  type: "0"
  date: "20190100000858"
  ```

### 3

* 修改原有proto文件

  ```
  package com.bd.hbase;
  message PhoneDetail
  {
          required string dnum = 1;
          required string length = 2;
          required string type = 3;
          required string date = 4;
  }
  message DayOfPhone
  {
          repeated PhoneDetail DayOfPhone = 1;
  }
  ```

* 重新执行文件`[root@node01 ~]# /usr/local/bin/protoc phone.proto --java_out=/root/`

* 移动到本地

* 删除原有表`hbase(main):004:0> truncate 'phone'`

* 再次插入数据

  ```java
  /**
       * 10个用户， 每天产生1000条记录，每天的所有数据放到一个rowkey中
       */
      @Test
      public void insert3() throws ParseException, InterruptedIOException, RetriesExhaustedWithDetailsException {
          List<Put> puts = new ArrayList<Put>();
          for (int i = 0; i < 10; i++){
              String phoneNumber = getPhone("158");
              String rowkey = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse("20190402000000").getTime());
              Phone.DayOfPhone.Builder dayOfPhone = Phone.DayOfPhone.newBuilder();
              for (int j = 0; j < 1000; j++){
                  String dnum = getPhone("177");
                  String length = String.valueOf(r.nextInt(99)); // 产生[0, n)的随机数
                  String type = String.valueOf(r.nextInt(2)); // [0, 2)
                  String date = getDate("20190402");
  
                  // 保存到对象属性中
                  Phone.PhoneDetail.Builder phoneDetail = Phone.PhoneDetail.newBuilder();
                  phoneDetail.setDate(date);
                  phoneDetail.setLength(length);
                  phoneDetail.setType(type);
                  phoneDetail.setDnum(dnum);
                  dayOfPhone.addDayOfPhone(phoneDetail);
  
                  //rowkey
                  Put put = new Put(rowkey.getBytes());
                  put.add("cf".getBytes(), "day".getBytes(), dayOfPhone.build().toByteArray());
                  puts.add(put);
              }
          }
          table.put(puts);
      }
  ```

* 查看数据

  ![](pic\查看10条数据.PNG)

* 找一个值准备取出

  ```java
  @Test
      public void get2() throws IOException {
          Get get = new Get("15899017841_9223370482720375807".getBytes());
          table.get(get);
          Result result = table.get(get);
          Phone.DayOfPhone parseFrom = Phone.DayOfPhone.parseFrom(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(),
                  "day".getBytes())));
          int count = 0;
          for (Phone.PhoneDetail pd : parseFrom.getDayOfPhoneList()) {
              System.out.println(pd);
              count++;
          }
          System.out.println(count);
      }
  ```

  执行成功

  ![](pic\查询整合成10条的1000条数据.PNG)

