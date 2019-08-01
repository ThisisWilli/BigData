# 搭建YARN并进行单词统计

## 节点分布

|        | NN-1 | NN-2 | DN   | ZK   | ZKFC | JNN  | RS   | NM   |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Node01 | *    |      |      |      | *    | *    |      |      |
| Node02 |      | *    | *    | *    | *    | *    |      | *    |
| Node03 |      |      | *    | *    |      | *    | *    | *    |
| Node04 |      |      | *    | *    |      |      | *    | *    |

## 配置单节点Yarn

* `[root@node01 hadoop]# mv mapred-site.xml.template mapred-site.xml`

* `[root@node01 hadoop]# vi mapred-site.xml `

  ```XML
  <configuration>
      <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
      </property>
  </configuration>
  ```

* `[root@node01 hadoop]# vi yarn-site.xml `

  ```xml
  <configuration>
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>
      <property>
          <name>yarn.resourcemanager.ha.enabled</name>
          <value>true</value>
      </property>
      <property>
          <name>yarn.resourcemanager.cluster-id</name>
          <value>cluster1</value>
      </property>
      <property>
          <name>yarn.resourcemanager.ha.rm-ids</name>
          <value>rm1,rm2</value>
      </property>
      <property>
          <name>yarn.resourcemanager.hostname.rm1</name>
          <value>node03</value>
      </property>
      <property>
          <name>yarn.resourcemanager.hostname.rm2</name>
          <value>node04</value>
      </property>
      <property>
          <name>yarn.resourcemanager.zk-address</name>
          <value>node02:2181,node03:2181,node04:2181</value>
      </property>
  </configuration>
  ```

* `[root@node01 hadoop]# scp mapred-site.xml yarn-site.xml node02:`pwd`

* `[root@node01 hadoop]# scp mapred-site.xml yarn-site.xml node03:`pwd`

* `[root@node01 hadoop]# scp mapred-site.xml yarn-site.xml node04:`pwd`

* node03,node04之间配置免密钥

* `[root@node03 .ssh]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa`

* `[root@node03 .ssh]# cat id_dsa.pub >> authorized_keys`

* `[root@node03 .ssh]# ssh node03`

* `[root@node03 ~]# exit`

* `[root@node03 .ssh]# scp id_dsa.pub node04:`pwd`/node03.pub`

* `[root@node04 .ssh]# cat node03.pub >> authorized_keys`

* `[root@node04 .ssh]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa`

* `[root@node04 .ssh]# ssh localhost`

* `[root@node04 ~]# exit`

* `[root@node04 .ssh]# scp id_dsa.pub node03:`pwd`/node04.pub`

* `[root@node03 .ssh]# cat node04.pub >> authorized_keys`

## 启动相关服务

* 4个节点全部启动zkServer.sh start
* `[root@node01 hadoop]# start-dfs.sh`
* 启动yarn`[root@node01 hadoop]# start-yarn.sh`
* 启动resourcemanager`[root@node03 .ssh]# yarn-daemon.sh start resourcemanager`，`[root@node04 .ssh]# yarn-daemon.sh start resourcemanager`
* `node03:8088` 查看resourceManager

## 测试

* `[root@node01 mapreduce]# cd /opt/sxt/hadoop-2.6.5/share/hadoop/mapreduce`

* `[root@node01 mapreduce]# hadoop jar hadoop-mapreduce-examples-2.6.5.jar wordcout /user/root/test.txt /wordcount`

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/resourcemanager%E4%B8%AD%E6%9F%A5%E7%9C%8B.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/resourcemanager中查看.PNG)

* `[root@node01 mapreduce]# hdfs dfs -cat /wordcount/part-r-00000`

## 关闭服务

* 关闭resourcemanager `[root@node03 .ssh]# yarn-daemon.sh stop resourcemanager`

  ``[root@node04 .ssh]# yarn-daemon.sh stop resourcemanager``

* `[root@node01 mapreduce]# stop-all.sh`

## 获取配置文件，添加到项目中

* `[root@node01 ~]# cd /opt/sxt/hadoop-2.6.5/etc/hadoop`，将4个xml文件放入idea项目中的resources中

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E8%8E%B7%E5%8F%96%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/获取配置文件.PNG)
## 启动集群

* 启动zookeeper，所有node
* 每个node`zkServer.sh start`
* `[root@node01 hadoop]# start-all.sh`
* 启动两个resourceManager `[root@node03 ~]# yarn-daemon.sh start resourcemanager `
* `[root@node04 ~]# yarn-daemon.sh start resourcemanager`

## 编写wordcount

* 在项目中创建一个包并创建3个类

```java
package com.sxt.mr.wc;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * \* Project: hadoop
 * \* Package: com.sxt.mr.wc
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-01 09:51:12
 * \* Description:
 * \
 */
public class MyWC {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(MyWC.class);
        job.setJobName("myjob");

        //读取文件的路径
        Path inPath = new Path("/user/root/test.txt");
        FileInputFormat.addInputPath(job, inPath);
        Path outPath = new Path("/output/wordcount");
        //设置文件输出路径
        if (outPath.getFileSystem(conf).exists(outPath)){
            outPath.getFileSystem(conf).delete(outPath, true);
        }
        FileOutputFormat.setOutputPath(job, outPath);

        job.setMapperClass(MyMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setReducerClass(MyReducer.class);

        job.waitForCompletion(true);
    }
}
```

```java
package com.sxt.mr.wc;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.StringTokenizer;

/**
 * \* Project: hadoop
 * \* Package: com.sxt.mr.wc
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-01 10:11:53
 * \* Description:
 * \
 */
// 自定义类， Text等同与String， IntWritable代表出现次数Mapper <Object, Text, Text, IntWritable>
public class MyMapper extends Mapper <Object, Text, Text, IntWritable>{
    //Object->Text split   Text->IntWritable  map
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    public void map(Object key, Text value, Context context) throws IOException, InterruptedException{
        StringTokenizer itr = new StringTokenizer(value.toString());
        // 遍历split完的每行的value
        while (itr.hasMoreTokens()){
            word.set(itr.nextToken());
            context.write(word, one);
        }
    }
}
```

```java
package com.sxt.mr.wc;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * \* Project: hadoop
 * \* Package: com.sxt.mr.wc
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-01 10:29:03
 * \* Description:
 * \
 */
//
public class MyReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    //迭代计算
    //同一组key进行一次迭代计算
    private IntWritable result = new IntWritable();
    // shuffle完成之后， key完全相同才能调用reduce
    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException{
        int sum = 0;
        for(IntWritable val : values){
            sum += val.get();
        }
        result.set(sum);
        context.write(key, result);
    }
}
```

* 将项目打包成jar包，打包之前先检查jdk版本，node中为jdk7，idea中用的是jdk11，直接打包运行会有不兼容问题，要在pom.xml文件中指定jdk的编译版本

  ```XML
  	<properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <maven.compiler.target>1.7</maven.compiler.target>
          <maven.compiler.source>1.7</maven.compiler.source>
      </properties>
  ```

* File->Project Structure->Artifacts->+->选择JAR

  ![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E6%89%93%E5%8C%85%E6%88%90jar%E6%96%87%E4%BB%B6.PNG](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/打包成jar文件.PNG)

* Build->Build Artifacts选择build，在工程的out文件夹中可以看到生成的jar包

## 运行wordcount

* 将生成的jar包上传的主namenode中

* `[root@node01 software]# hadoop jar wc.jar  com.sxt.mr.wc.MyWC`

  

  

