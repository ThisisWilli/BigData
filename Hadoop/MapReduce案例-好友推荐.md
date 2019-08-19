# MapReduce实例-好友推荐

## 背景

好友关系图如下图所示

![https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/%E5%A5%BD%E5%8F%8B%E6%8E%A8%E8%8D%90%E5%9B%BE.png](https://raw.githubusercontent.com/ThisisWilli/BigData/master/Hadoop/pic/好友推荐图.png)

转化为文字之后为，即为输入数据

```
tom hello hadoop cat
world hadoop hello hive
cat tom hive
mr hive hello
hive cat hadoop world hello mr
hadoop tom hive world
hello tom world hive mr
```



## 思路

* 推荐者与被推荐者一定有一个或多个相同的好友
* 全局去寻找好友列表中两两关系
* 去除直接好友
* 统计两两关系出现次数

## 调用API

* map：按好友列表输出两俩关系
* reduce：sum两两关系
* 再设计一个MR
*  生成详细报表

## 编写Mapper

```java
package com.sxt.FR;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.util.StringUtils;

import java.io.IOException;

/**
 * \* Project: FriendRecommended
 * \* Package: com.sxt.FR
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-17 13:27:40
 * \* Description:
 * \
 */

public class FMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
    Text tkey = new Text();
    IntWritable tval = new IntWritable();
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //world hello hadoop cat
        String[] words = StringUtils.split(value.toString(), ' ');

        for (int i = 1; i < words.length; i++){
            // 把一对封装在tkey中
            tkey.set(getFd(words[0], words[i]));
            // 如果是直接好友，则直接输出0
            tval.set(0);
            // 用数组的第一个元素与后面的所有元素一一匹配，输出他们他们的直接好友关系
            context.write(tkey, tval);

            for (int j = i + 1; j < words.length; j++){
                // 把一对儿封装在tkey中
                tkey.set(getFd(words[i], words[j]));
                // 如果是潜在间接好友， 则直接输出1
                tval.set(1);
                // 用数组的第一个元素与后面的所有元素一一匹配，输出他们他们的直接好友关系
                context.write(tkey, tval);
            }
        }
    }
    private String getFd(String a, String b){
        return a.compareTo(b) > 0 ? b+":"+a : a+":"+b;
    }
}
```

## 编写Reducer

```JAVA
package com.sxt.FR;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * \* Project: FriendRecommended
 * \* Package: com.sxt.FR
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-17 13:31:17
 * \* Description:
 * \
 */
public class FReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    IntWritable tval = new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        // 出现tom:hello 0代表两个人直接认识，不需要再去推荐
        // tom:hello 1
        // tom:hell0 1

        // 只考虑以下这种情况，两人间接认识的情况
        // hadoop:hive 1
        // hadoop:hive 1
        // hadoop:hive 1
        // hadoop:hive 1
        int num = 0;
        for (IntWritable val : values){
            if (val.get() == 0){
                return;
            }
            num++;
        }
        // 最终输出两个人之间的共同好友数
        tval.set(num);
        context.write(key, tval);
    }
}
```

## 编写主类

```java
package com.sxt.FR;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * \* Project: FriendRecommended
 * \* Package: com.sxt.FR
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-17 13:15:31
 * \* Description:
 * \
 */
public class MyFD {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(MyFD.class);
        job.setJobName("friend");

        Path inPath = new Path("/user/fd/input");
        FileInputFormat.addInputPath(job, inPath);
        Path outPath = new Path("/user/fd/output");
        if (outPath.getFileSystem(conf).exists(outPath)){
            outPath.getFileSystem(conf).delete(outPath, true);
        }
        FileOutputFormat.setOutputPath(job, outPath);

        job.setMapperClass(FMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setReducerClass(FReducer.class);
        job.waitForCompletion(true);
    }
}
```

## 结果

讲项目打包成jar包在放在namenode上运行

`[root@node01 software]# hadoop jar FriendRecommended.jar com.sxt.FR.MyFD`

查看输出数据为

```
cat:hadoop	2
cat:hello	2
cat:mr	1
cat:world	1
hadoop:hello	3
hadoop:mr	1
hive:tom	3
mr:tom	1
mr:world	2
tom:world	2
```



