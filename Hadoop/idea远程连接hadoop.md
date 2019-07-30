# idea远程连接hadoop进行文件操作

部署完高可用集群之后，尝试idea远程连接hadoop进行操作

## 配置windows中的hadoop环境

* 下载hadoop2.6.5到windows中并放在一个纯英文目录下

* 配置环境变量，先系统变量中创建HADOOP_HOME

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E5%88%9B%E5%BB%BAHADOOP_HOME.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/创建HADOOP_HOME.PNG)

* 创建HADOOP_USER_NAME，名称为集群中的登录名称

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E5%88%9B%E5%BB%BAHADOOP_USER_NAME.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/创建HADOOP_USER_NAME.PNG)

* 在系统变量的Path中添加%HADOOP_HOME%/bin

* 将hadoop.dll添加到C:\Windows\System32文件夹下

* 在命令行中输入hdfs和hadoop，检测是否安装成功

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/hadoop%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/hadoop是否安装成功.PNG)

## 在idea中配置hadoop

* 首先下载插件(再次感谢作者)https://github.com/fangyuzhong2016/HadoopIntellijPlugin，直接clone下来即可，注意项目中所要求的配置信息

* idea中创建maven项目

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E5%88%9B%E5%BB%BAmaven%E5%B7%A5%E7%A8%8B.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/创建maven工程.PNG)

* 工程创建完成之后，File->Settings-Plugin->点击右上角的设置->Install plugin from disk

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E5%AE%89%E8%A3%85plugins.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/安装plugins.PNG)

* 重启idea，菜单栏中会出现hadoop选项

## 连接hadoop，进行文件操作

* 首先开启集群，先`zkServer.sh start`开启zookeeper，再`start-dfs.sh`开启全部节点，在配置节点前首先进入node01:50070查看node01和node02的状态。

* 点击idea上方菜单栏中的hadoop进入设置

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E8%AE%BE%E7%BD%AEhadoop%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/设置hadoop文件系统.PNG)

* 确定之后左边会出现hadoop

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/hadoop%E4%BE%A7%E8%BE%B9%E6%A0%8F.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/hadoop侧边栏.PNG)

* 进行文件操作，代码具体如下，可用junit进行调试，并在hadoop插件中查看hdfs文件系统

  ```java
  package com.sxt.hdfs.test;
  
  import org.apache.hadoop.conf.Configuration;
  import org.apache.hadoop.conf.Configured;
  import org.apache.hadoop.fs.*;
  import org.apache.hadoop.io.IOUtils;
  import org.junit.After;
  import org.junit.Before;
  import org.junit.Test;
  
  import java.io.BufferedInputStream;
  import java.io.FileInputStream;
  import java.io.IOException;
  import java.io.InputStream;
  /**
   * \* Project: hadoop
   * \* Package: com.sxt.hdfs.test
   * \* Author: Hoodie_Willi
   * \* Date: 2019-07-29 20:36:22
   * \* Description:
   * \
   */
  public class test {
      Configuration conf = null;
      FileSystem fs = null;
      @Before
      public void conn() throws IOException {
          conf = new Configuration();
          fs = FileSystem.get(conf);
          //System.out.println("success");
      }
      @Test
      public void mkdir() throws IOException {
          Path path = new Path("/mytemp");
          if (fs.exists(path)){
              fs.delete(path);
          }
          fs.mkdirs(path);
      }
      @Test
      public void uploadFile() throws IOException{
          //文件上传路径
          Path path = new Path("/mytemp/haha.txt");
          FSDataOutputStream fdos = fs.create(path);
          //获取磁盘文件
          InputStream is = new BufferedInputStream(new FileInputStream("D:\\IdeaProject\\hadoop\\src\\files\\hello.txt"));
          IOUtils.copyBytes(is, fdos, conf, true);
      }
      @Test
      public void readFile() throws IOException{
          Path f = new Path("/user/root/test.txt");
          FileStatus file = fs.getFileStatus(f);
  //        BlockLocation[] blks = fs.getFileBlockLocations(file,0, file.getLen());
  //        for (BlockLocation blk : blks){
  //            System.out.println(blk);
  //        }
          //读取文件
          FSDataInputStream fdis = fs.open(f); // fileinputstream
          fdis.seek(1048576);
          System.out.println((char) fdis.readByte());
          System.out.println((char) fdis.readByte());
          System.out.println((char) fdis.readByte());
          System.out.println((char) fdis.readByte());
          System.out.println((char) fdis.readByte());
      }
      //@After
      public void close() throws IOException{
          fs.close();
      }
  }
  ```

  

