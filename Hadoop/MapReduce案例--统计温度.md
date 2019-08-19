# MapReduce案例--统计温度

## 需求

从数据中找出每月气温最高的两天

## 设置Mapper

### 自定义weather类

* 将日期转化为int去比较
* 实现WritableComparable接口，在Map/Reduce中，任何用作键来使用的类都应该实现WritableComparable接口
* 所谓的序列化，就是将结构化对象转化为字节流，以便在网络上传输或是写道磁盘进行永久存储。反序列化，就是将字节流转化为结构化对象。

```java
package com.sxt.weather;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * \* Project: WeatherTemp
 * \* Package: com.sxt.weather
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-15 14:53:31
 * \* Description:
 * \
 */
public class Weather implements WritableComparable<Weather> {
    private int year;
    private int month;
    private int day;
    private int temp;

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public int getMonth() {
        return month;
    }

    public void setMonth(int month) {
        this.month = month;
    }

    public int getDay() {
        return day;
    }

    public void setDay(int day) {
        this.day = day;
    }

    public int getTemp() {
        return temp;
    }

    public void setTemp(int temp) {
        this.temp = temp;
    }


    //重写writable中得方法
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeInt(this.getYear());
        dataOutput.writeInt(this.getMonth());
        dataOutput.writeInt(this.getDay());
        dataOutput.writeInt(this.getTemp());
    }

    public void readFields(DataInput dataInput) throws IOException {
        this.setYear(dataInput.readInt());
        this.setMonth(dataInput.readInt());
        this.setDay(dataInput.readInt());
        this.setTemp(dataInput.readInt());
    }

    public int compareTo(Weather o) {
        // 两个天气进行比较，根据数值类型选择比较器，compare返回0,1,-1
        int c1 = Integer.compare(this.getYear(), o.getYear());
        if (c1==0){
            int c2 = Integer.compare(this.getMonth(), o.getMonth());
            if (c2 == 0){
                return Integer.compare(this.getDay(), o.getDay());
            }
            return c2;
        }
        return c1;
    }


    @Override
    public String toString() {
        return year + '-' + month + "-" + day;
    }
}
```

### 实现Mapper类

```java
package com.sxt.weather;

import com.google.inject.internal.cglib.core.$ObjectSwitchCallback;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.util.StringUtils;
import sun.util.resources.cldr.bas.CalendarData_bas_CM;

import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * \* Project: WeatherTemp
 * \* Package: com.sxt.weather
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-15 15:14:09
 * \* Description:
 * \
 */
//        a.泛型一:KEYIN：LongWritable，对应的Mapper的输入key。输入key是每行的行首偏移量
//        b.泛型二: VALUEIN：Text，对应的Mapper的输入Value。输入value是每行的内容
//        c.泛型三:KEYOUT：对应的Mapper的输出key，根据业务来定义
//        d.泛型四:VALUEOUT：对应的Mapper的输出value，根据业务来定义
//        初学时，KEYIN和VALUEIN写死(LongWritable,Text)。KEYOUT和VALUEOUT不固定,根据业务来定
// 1949-10-01 14:21:02 34C
public class Tmapper extends Mapper <LongWritable, Text, Weather, IntWritable> {
    Weather tkey = new Weather(); // 处理后的年月日，温度
    IntWritable tval = new IntWritable(); // 封装温度
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //对一行数据进行处理1949-10-01 14:21:02 34C,获得时间温度
        String[] words = StringUtils.split(value.toString(), '\t'); // 定位制导符，切割时间和温度
        String pattern = "yyyy-MM-dd";
        SimpleDateFormat sdf = new SimpleDateFormat(pattern);
        try {
            Date date = sdf.parse(words[0]);
            Calendar cal = Calendar.getInstance();
            cal.setTime(date);
            // 处理日期
            tkey.setYear(cal.get(Calendar.YEAR));
            tkey.setMonth(cal.get(Calendar.MONTH) + 1);
            tkey.setDay(cal.get(Calendar.DAY_OF_MONTH));

            // 切割出温度
            int temp = Integer.parseInt(words[1].substring(0, words[1].lastIndexOf("c")));
            tkey.setTemp(temp);
            tval.set(temp);
            context.write(tkey, tval);
        } catch (ParseException e) {
            e.printStackTrace();
        }

    }
}
```

## 实现自定义排序比较器

保证相同时间的温度被分到一个reduce并且温度数据从大到小排列

```java
package com.sxt.weather;

import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

/**
 * \* Project: WeatherTemp
 * \* Package: com.sxt.weather
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-15 15:37:55
 * \* Description:实现天气年月正序，温度倒序
 * \
 */
public class TSortComparator extends WritableComparator {
    Weather t1 = null;
    Weather t2 = null;

    public TSortComparator() {
        super(Weather.class, true);
    }

    @Override
    /**
     * 保证相同的时间的数据被分到一个reduce
     */
    public int compare(WritableComparable a, WritableComparable b) {
        t1 = (Weather) a;
        t2 = (Weather) b;
        int c1 = Integer.compare(t1.getYear(), t2.getYear());
        if (c1 == 0){
            int c2 = Integer.compare(t1.getMonth(), t2.getMonth());
            if (c2 == 0){
                // 添加-保证高的温度在前，低的温度在后
                return -Integer.compare(t1.getTemp(), t2.getTemp());
            }
            return c2;
        }
        return c1;
    }
}
```

##  自定义分区器

```java
package com.sxt.weather;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Partitioner;

/**
 * \* Project: WeatherTemp
 * \* Package: com.sxt.weather
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-15 16:05:56
 * \* Description:
 * \
 */
public class TPartitioner extends Partitioner<Weather, IntWritable> {
    @Override
    /**
     * i与reducetask有直接关系
     */
    public int getPartition(Weather weather, IntWritable intWritable, int i) {
        return weather.getYear() % i;
    }
}
```

## 自定义组排序

```java
package com.sxt.weather;

import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

/**
 * \* Project: WeatherTemp
 * \* Package: com.sxt.weather
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-15 16:13:31
 * \* Description:
 * \
 */
public class TGroupComparator extends WritableComparator {
    Weather t1 = null;
    Weather t2 = null;

    public TGroupComparator() {
        super(Weather.class, true);
    }

    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        t1 = (Weather) a;
        t2 = (Weather) b;
        int c1 = Integer.compare(t1.getYear(), t2.getYear());
        if (c1 == 0){
            return Integer.compare(t1.getMonth(), t2.getMonth());
        }
        return c1;
    }
}
```

## 设置Reducer

```java
package com.sxt.weather;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * \* Project: WeatherTemp
 * \* Package: com.sxt.weather
 * \* Author: Hoodie_Willi
 * \* Date: 2019-08-15 16:16:52
 * \* Description:
 * \
 */
//年月相同
//1949-10-01  38
//1949-10-02  37
//1949-10-03  34
//1949-10-01  34
public class Treducer extends Reducer<Weather, IntWritable, Text, IntWritable> {
    Text tkey = new Text();
    IntWritable tval = new IntWritable();
    @Override
    protected void reduce(Weather key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        // 过滤掉同一天的别的温度，因为第一条数据已经是这一天的最高温
        int flag = 0;
        int day = 0;
        for (IntWritable val : values){
            if (flag == 0){
                tkey.set(key.toString());
                tval.set(val.get());
                context.write(tkey, tval);
                flag++;
                day = key.getDay();
            }
            if (flag != 0 && day != key.getDay()){
                tkey.set(key.toString());
                tval.set(val.get());
                context.write(tkey, tval);
                return;
            }
        }
    }
}
```

