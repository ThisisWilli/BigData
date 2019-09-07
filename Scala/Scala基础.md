# Scala基础

## 数据类型

* Byte 8bit的有符号数字，范围[-128, 127]

* Short 16 bit有符号数字，范围在[-32768, 32767]

* Int 32 bit有符号数字，范围[-2147483648, 2147483647]

* Long 64 bit 有符号数字

* Float 32 bit IEEE 754 单精度浮点数

* Double 64 bit IEEE 7854双精度浮点数

* Char 16 bit Unicode字符 

* String字符串

* Boolean布尔类型

* Unit表示无值

* Null空值或者空引用

* Nothing 所有其他类型的子类型，表示没有值

* Any所有类型的超类，所有类型都属于Any类型

* AnyRef所有引用类型的超类

* AnyVal所有值类型的超类

  ![](pic\Scala类之间的关系.png)

| Null    | trait,其唯一实例为null，是AnyRef的子类，不是AnyVal的子类 |
| ------- | -------------------------------------------------------- |
| Nothing | trait，所有类型(包括AnyRef和AnyVal)的子类，没有实例      |
| None    | Option的两个子类之一，另一个是Some，用于安全的函数返回值 |
| Unit    | 无返回值的函数类型，和java的void对应                     |
| Nil     | 长度为0的List                                            |

## 类和对象

* 1.Scala Object 相当于java中的单例，Object中定义的全是静态的,相当于java的工具类，**Object不可传参**,对象要传参，使用apply方法 

* Scala中Object和Class的区别：scala 中没有 static 关键字对于一个class来说，所有的方法和成员变量在实例被 new 出来之前都是无法访问的因此class文件中的main方法也就没什么用了，**scala object 中所有成员变量和方法默认都是 static 的所以 可以直接访问main方法**，object中必须有main方法，才能执行其中的方法

* 2.Scala中定义变量使用var，定义常量使用val，**变量可变，常量不可变**，变量和常量类型可以省略不写，会自动推断                        

* 3.Scala中每行后面都有分号自动推断机制，不用显式写出";"                                                  

* 4.Scala中命名建议驼峰                                                                    

* 5.Scala类中可以传参，传参一定要指定类型，有了参数就默认有了构造，类中的属性默认有getter和setter方法                       

* 6.类中重写构造时，构造中第一行必须先调用默认的构造   def this(){....}                                     

* 7.Scala中当new class时，**类中除了方法不执行【除了构造方法，但是辅助构造方法执行】其他都执行**，所以println会起作用                                        

* 8.在同一Scala文件中，class名称和Object名称一样是，这个类叫做对象的半生类，这个对象叫做这个类的伴生对象，**他们之间可以互相访问私有变量**       

  ```scala
  class Person(xname:String, xage:Int){                            
    val name = xname;                                              
    val age = xage                                                 
    var gender = 'F'                                               
                                                                   
  //  println("*********Person Class************")                 
    def this(yname:String, yage:Int, ygender:Char){                
      this(yname, yage)                                            
      this.gender = ygender                                        
    }                                                              
                                                                   
    def sayName()={                                                
      println("hello world...." + LessonClassAndObject.name)       
    }                                                              
  //  println("==========Person Class============")                
  }                                                                
                                                                   
  object LessonClassAndObject {                                    
                                                                   
    def apply(i:Int) = {                                           
      println("Score is " + i)                                     
    }                                                              
                                                                   
    val name = "wangwu"                                            
    def main(args: Array[String]): Unit = {                        
      LessonClassAndObject(1000)                                   
  //    val a = 100;                                               
  //    val b = 200;                                               
  //    println(b)                                                 
  //    val p = new Person("zhangsan", 20)                         
  //    println(p.gender)                                          
  //    val p1 = new Person("diaochan", 18, 'F')                   
  //    println(p1.gender)                                         
  //    println(p.name)                                            
  //    println(p.age)                                             
  //    p.sayName()                                                
    }                                                              
  }                                                                
  ```


## Circulation

* 创建多层for循环

  ```scala
  //可以分号隔开，写入多个list赋值的变量，构成多层for循环
      //scala中 不能写count++ count-- 只能写count+
      var count = 0;
      for(i <- 1 to 10; j <- 1 until 10){
        println("i="+ i +",	j="+j)
        count += 1
      }
      println(count);
      
      //例子： 打印小九九
      for(i <- 1 until 10 ;j <- 1 until 10){
        if(i>=j){
      	  print(i +" * " + j + " = "+ i*j+"	")
          
        }
        if(i==j ){
          println()
        }
      }
  
  ```

* 循环中可以增加条件

  ```scala
  for(i<- 1 to 10 ;if (i%2) == 0 ;if (i == 4) ){
        println(i)
  }
  ```

## Scala中方法的定义

![](pic\Scala中方法的定义.png)

```Scala
def fun (a: Int , b: Int ) : Unit = {
   println(a+b)
 }
fun(1,1)
    
def fun1 (a : Int , b : Int)= a+b
    println(fun1(1,2))  
```

* 方法定义语法 用def来定义
* 可以定义传入的参数，要指定传入参数的类型
* 方法可以写返回值的类型也可以不写，会自动推断，有时候不能省略，必须写，比如在递归方法中或者方法的返回值是函数类型的时候。
* scala中方法有返回值时，**可以写return，也可以不写return**，会把方法中最后一行当做结果返回。当写return时，必须要写方法的返回值。
* 如果返回值可以一行搞定，可以将{}省略不写
* 传递给方法的参数可以在方法中使用，并且scala规定方法的传过来的参数为val的，不是var的。
* **如果去掉方法体前面的等号，那么这个方法返回类型必定是Unit的**。这种说法无论方法体里面什么逻辑都成立，scala可以把任意类型转换为Unit.假设，里面的逻辑最后返回了一个string，那么这个返回值会被转换成Unit，并且值会被丢弃。

### 递归方法

```scala
 /**
     * 递归方法 
     * 5的阶乘
     */
    def fun2(num :Int) :Int= {
      if(num ==1)
        num
      else 
        num * fun2(num-1)
    }
    print(fun2(5))
```

### 参数中有默认值的方法

* 默认值的函数中，如果传入的参数个数与函数定义相同，则传入的数值会覆盖默认值。

* 如果不想覆盖默认值，传入的参数个数小于定义的函数的参数，则需要指定参数名称。

  ```scala
  def fun3(a :Int = 10,b:Int) = {
        println(a+b)
      }
      fun3(b=2)
  ```

### 可变参数的方法

* 多个参数用逗号隔开

  ```scala
  /**
       * 可变参数个数的函数
       * 注意：多个参数逗号分开
       */
      def fun4(elements :Int*)={
        var sum = 0;
        for(elem <- elements){
          sum += elem
        }
        sum
      }
      println(fun4(1,2,3,4))
  ```

### 匿名函数

* 有参匿名函数

* 无参匿名函数

* 有返回值的匿名函数

* 可以将匿名函数返回给val定义的值

* 匿名函数不能显式声明函数的返回类型

  ```scala
   //有参数匿名函数
      val value1 = (a : Int) => {
        println(a)
      }
      value1(1)
      //无参数匿名函数
      val value2 = ()=>{
        println("我爱尚学堂")
      }
      value2()
      //有返回值的匿名函数
      val value3 = (a:Int,b:Int) =>{
        a+b
      }
      println(value3(4,4)) 
  ```

### 嵌套方法

```scala
    /**
     * 嵌套方法
     * 例如：嵌套方法求5的阶乘
     */
    def fun5(num:Int)={
      def fun6(a:Int,b:Int):Int={
        if(a == 1){
          b
        }else{
          fun6(a-1,a*b)
        }
      }
      fun6(num,1)
    }
    println(fun5(5))
```

### 偏应用函数

* 偏应用函数是一种表达式，不需要提供函数所需要的所有参数，只需要提供部分，或者不提供所需参数

  ```scala
  /**
       * 偏应用函数
       */
      def log(date :Date, s :String)= {
        println("date is "+ date +",log is "+ s)
      }
      
      val date = new Date()
      log(date ,"log1")
      log(date ,"log2")
      log(date ,"log3")
      
      //想要调用log，以上变化的是第二个参数，可以用偏应用函数处理
      val logWithDate = log(date,_:String)
      logWithDate("log11")
      logWithDate("log22")
      logWithDate("log33")
  ```

### 高阶函数

函数的参数是函数，或者函数的返回类型是函数，或者函数的参数和函数的返回类型是函数的函数。

* 函数的参数是函数
* 函数的返回是函数
* 函数的参数和函数的返回是函数

```scala
/**
     * 高阶函数
     * 函数的参数是函数		或者函数的返回是函数		或者函数的参数和返回都是函数
     */
    
    //函数的参数是函数
    def hightFun(f : (Int,Int) =>Int, a:Int ) : Int = {
      f(a,100)
    }
    def f(v1 :Int,v2: Int):Int  = {
      v1+v2
    }
    
    println(hightFun(f, 1))
    
    //函数的返回是函数
    //1，2,3,4相加
    def hightFun2(a : Int,b:Int) : (Int,Int)=>Int = {
      def f2 (v1: Int,v2:Int) :Int = {
        v1+v2+a+b
      }
      f2
    }
    println(hightFun2(1,2)(3,4))
    
    //函数的参数是函数，函数的返回是函数
    def hightFun3(f : (Int ,Int) => Int) : (Int,Int) => Int = {
      f
    } 
    println(hightFun3(f)(100,200))
    println(hightFun3((a,b) =>{a+b})(200,200))
    //以上这句话还可以写成这样
    //如果函数的参数在方法体中只使用了一次 那么可以写成_表示
    println(hightFun3(_+_)(200,200))

```

### 柯里化函数

* 高阶函数的简化

  ```scala
  /**
       * 柯里化函数
       */
      def fun7(a :Int,b:Int)(c:Int,d:Int) = {
        a+b+c+d
      }
      println(fun7(1,2)(3,4))
  ```

## Scala字符串

### String

```scala
char charAt(int index)
返回指定位置的字符  从0开始
	
int compareTo(Object o)
比较字符串与对象
	
int compareTo(String anotherString)
按字典顺序比较两个字符串
	
int compareToIgnoreCase(String str)
按字典顺序比较两个字符串，不考虑大小写
	
String concat(String str)
将指定字符串连接到此字符串的结尾
	
boolean contentEquals(StringBuffer sb)
将此字符串与指定的 StringBuffer 比较。
	
static String copyValueOf(char[] data)
返回指定数组中表示该字符序列的 String
	
static String copyValueOf(char[] data, int offset, int count)
返回指定数组中表示该字符序列的 String
	
boolean endsWith(String suffix)
测试此字符串是否以指定的后缀结束
	
boolean equals(Object anObject)
将此字符串与指定的对象比较
	
boolean equalsIgnoreCase(String anotherString)
将此 String 与另一个 String 比较，不考虑大小写
	
byte getBytes()
使用平台的默认字符集将此 String 编码为 byte 序列，并将结果存储到一个新的 byte 数组中
	
byte[] getBytes(String charsetName
使用指定的字符集将此 String 编码为 byte 序列，并将结果存储到一个新的 byte 数组中
	
void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
将字符从此字符串复制到目标字符数组
	
int hashCode()
返回此字符串的哈希码
16	
int indexOf(int ch)
返回指定字符在此字符串中第一次出现处的索引（输入的是ascii码值）
	
int indexOf(int ch, int fromIndex)
返返回在此字符串中第一次出现指定字符处的索引，从指定的索引开始搜索
	
int indexOf(String str)
返回指定子字符串在此字符串中第一次出现处的索引
	
int indexOf(String str, int fromIndex)
返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
	
String intern()
返回字符串对象的规范化表示形式
	
int lastIndexOf(int ch)
返回指定字符在此字符串中最后一次出现处的索引
	
int lastIndexOf(int ch, int fromIndex)
返回指定字符在此字符串中最后一次出现处的索引，从指定的索引处开始进行反向搜索
	
int lastIndexOf(String str)
返回指定子字符串在此字符串中最右边出现处的索引
	
int lastIndexOf(String str, int fromIndex)
返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索
	
int length()
返回此字符串的长度
	
boolean matches(String regex)
告知此字符串是否匹配给定的正则表达式
	
boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len)
测试两个字符串区域是否相等
28	
boolean regionMatches(int toffset, String other, int ooffset, int len)
测试两个字符串区域是否相等
	
String replace(char oldChar, char newChar)
返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所有 oldChar 得到的
	
String replaceAll(String regex, String replacement
使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串
	
String replaceFirst(String regex, String replacement)
使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串
	
String[] split(String regex)
根据给定正则表达式的匹配拆分此字符串
	
String[] split(String regex, int limit)
根据匹配给定的正则表达式来拆分此字符串
	
boolean startsWith(String prefix)
测试此字符串是否以指定的前缀开始
	
boolean startsWith(String prefix, int toffset)
测试此字符串从指定索引开始的子字符串是否以指定前缀开始。
	
CharSequence subSequence(int beginIndex, int endIndex)
返回一个新的字符序列，它是此序列的一个子序列
	
String substring(int beginIndex)
返回一个新的字符串，它是此字符串的一个子字符串
	
String substring(int beginIndex, int endIndex)
返回一个新字符串，它是此字符串的一个子字符串
	
char[] toCharArray()
将此字符串转换为一个新的字符数组
	
String toLowerCase()
使用默认语言环境的规则将此 String 中的所有字符都转换为小写
	
String toLowerCase(Locale locale)
使用给定 Locale 的规则将此 String 中的所有字符都转换为小写
	
String toString()
返回此对象本身（它已经是一个字符串！）
	
String toUpperCase()
使用默认语言环境的规则将此 String 中的所有字符都转换为大写
	
String toUpperCase(Locale locale)
使用给定 Locale 的规则将此 String 中的所有字符都转换为大写
	
String trim()
删除指定字符串的首尾空白符
	
static String valueOf(primitive data type x)
返回指定类型参数的字符串表示形式
```

### StringBuilder可变

### String方法举例

* 比较equals
* 比较忽略大小写
* indexOf：如果字符串中有传入的assco码对应的值，返回下标

```scala
    /**
     * String && StringBuilder
     */
    val str = "abcd"
    val str1 = "ABCD"
    
    println(str.indexOf(97))
    println(str.indexOf("b"))

    println(str==str1)
    /**
     * compareToIgnoreCase
     * 
     * 如果参数字符串等于此字符串，则返回值 0；
     * 如果此字符串小于字符串参数，则返回一个小于 0 的值；
     * 如果此字符串大于字符串参数，则返回一个大于 0 的值。
     * 
     */
    println(str.compareToIgnoreCase(str1))
    
    val strBuilder = new StringBuilder
    strBuilder.append("abc")
//    strBuilder.+('d')
    strBuilder+ 'd'
//    strBuilder.++=("efg")
    strBuilder++= "efg" 
//    strBuilder.+=('h')
    strBuilder+= 'h' 
    strBuilder.append(1.0)
    strBuilder.append(18f)
    println(strBuilder)

```

## 集合

### 数组

* 创建数组`new Array[Int](10)`,赋值：`arr(0) = xxx`或者`Array[String]("s1", "s2", "s3")`

* 数组遍历

  * for
  * foreach

* 创建一维数组，二维数组

* 数组中方法举例

  * Array.concate:合并数组
  * Array.fill(5)("bjsxt")：**创建初始值的定长数组**

* 创建的两种方式

  ```scala
  /**
       * 创建数组两种方式：
       * 1.new Array[String](3)
       * 2.直接Array
       */
      
      //创建类型为Int 长度为3的数组
      val arr1 = new Array[Int](3)
      //创建String 类型的数组，直接赋值
      val arr2 = Array[String]("s100","s200","s300")
      //赋值
      arr1(0) = 100
      arr1(1) = 200
      arr1(2) = 300
  ```

* 遍历的两种方式

  ```scala
  /**
     * 遍历的两种方式
     * @param args
     */
      for (i <- arr1){
        println(i)
      }
      arr1.foreach(i => {
        println(i)
      })
      for (s <- arr2){
        println(s)
      }
      arr2.foreach{
        x => println(x)
      }
  ```

* 创建二维数组以及遍历

  ```scala
  /**
     * 创建二维数组和遍历
     * @param args
     */
    val arr3 = new Array[Array[String]](3)
    arr3(0)=Array("1","2","3")
    arr3(1)=Array("4","5","6")
    arr3(2)=Array("7","8","9")
    for (i <- 0 until arr3.length){
      for (j <- 0 until arr3(i).length){
        print(arr3(i)(j) + " ")
      }
      println()
    }
    var count = 0
    for(arr <- arr3 ;i <- arr){
      if(count%3 == 0){
        println()
      }
      print(i+"	")
      count +=1
    }
  
    println()
    arr3.foreach { arr  => {
      arr.foreach { println }
    }}
  
  
    val arr4 = Array[Array[Int]](Array(1,2,3),Array(4,5,6))
    arr4.foreach { arr => {
      arr.foreach(i => {
        println(i)
      })
    }}
    println("-------")
    for(arr <- arr4;i <- arr){
      println(i)
    }
  ```

* 数组中的方法及描述

  ```
  序号	方法和描述
  1	
  def apply( x: T, xs: T* ): Array[T]
  创建指定对象 T 的数组, T 的值可以是 Unit, Double, Float, Long, Int, Char, Short, Byte, Boolean。
  2	
  def concat[T]( xss: Array[T]* ): Array[T]
  合并数组
  3	
  def copy( src: AnyRef, srcPos: Int, dest: AnyRef, destPos: Int, length: Int ): Unit
  复制一个数组到另一个数组上。相等于 Java's System.arraycopy(src, srcPos, dest, destPos, length)。
  4	
  def empty[T]: Array[T]
  返回长度为 0 的数组
  5	
  def iterate[T]( start: T, len: Int )( f: (T) => T ): Array[T]
  返回指定长度数组，每个数组元素为指定函数的返回值。
  以上实例数组初始值为 0，长度为 3，计算函数为a=>a+1：
  scala> Array.iterate(0,3)(a=>a+1)
  res1: Array[Int] = Array(0, 1, 2)
  6	
  def fill[T]( n: Int )(elem: => T): Array[T]
  返回数组，长度为第一个参数指定，同时每个元素使用第二个参数进行填充。
  7	
  def fill[T]( n1: Int, n2: Int )( elem: => T ): Array[Array[T]]
  返回二数组，长度为第一个参数指定，同时每个元素使用第二个参数进行填充。
  8	
  def ofDim[T]( n1: Int ): Array[T]
  创建指定长度的数组
  9	
  def ofDim[T]( n1: Int, n2: Int ): Array[Array[T]]
  创建二维数组
  10	
  def ofDim[T]( n1: Int, n2: Int, n3: Int ): Array[Array[Array[T]]]
  创建三维数组
  11	
  def range( start: Int, end: Int, step: Int ): Array[Int]
  创建指定区间内的数组，step 为每个元素间的步长
  12	
  def range( start: Int, end: Int ): Array[Int]
  创建指定区间内的数组
  13	
  def tabulate[T]( n: Int )(f: (Int)=> T): Array[T]
  返回指定长度数组，每个数组元素为指定函数的返回值，默认从 0 开始。
  以上实例返回 3 个元素：
  scala> Array.tabulate(3)(a => a + 5)
  res0: Array[Int] = Array(5, 6, 7)
  14	
  def tabulate[T]( n1: Int, n2: Int )( f: (Int, Int ) => T): Array[Array[T]]
  返回指定长度的二维数组，每个数组元素为指定函数的返回值，默认从 0 开始。
  ```

* 可变长数组ArrayBuffer

  ```scala
  val arr = ArrayBuffer[String]("a", "b", "c")
  arr.append("hello", "scala") //添加多个元素
  arr. +=("end") //在最后添加元素
  arr. +=:("start") //在开头添加元素
  arr.foreach(println)
  ```

### List

* 创建list`val list = List(1, 2, 3, 4)`

* List遍历 foreach，for

* list方法举例

  * filter:过滤元素
  * count:计算符合条件的元素个数
  * map：对元素操作
  * flatmap ：压扁扁平,先map再flat

  ![](pic\flatmap示例图.png)

  ```scala
  //创建
    val list = List(1,2,3,4,5)
  
    //遍历
    list.foreach { x => print(x)}
    println()
    //    list.foreach { println}
    //filter , 滤除掉使函数返回false的元素
    val list1  = list.filter { x => x>3 }
    list1.foreach { print}
    println()
    //count
    val value = list1.count { x => x>3 }
    println(value)
  
    //map
    val nameList = List(
      "hello bjsxt",
      "hello xasxt",
      "hello shsxt"
    )
    val mapResult:List[Array[String]] = nameList.map{ x => x.split(" ") }
    mapResult.foreach{println}
  
    //flatmap
    val flatMapResult : List[String] = nameList.flatMap{ x => x.split(" ") }
    flatMapResult.foreach { println }
  ```

* 可变长List

  ```scala
   /**
     * 可变长list
     * @param args
     */
  
    val listBuffer: ListBuffer[Int] = ListBuffer[Int](1, 2, 3, 4, 5)
    listBuffer.append(6, 7, 8, 9) //追加元素
    listBuffer.+=(10) //在后面追加元素
    listBuffer.+=(100) //在开头追加元素
    listBuffer.foreach(print)
  ```

### Set

* 创建set ，**set集合会自动去重**

* set遍历 foreach， for

* set方法举例

  * 交集：intersect
  * 差集: diff
  * 子集:subsetOf
  * 最大:max
  * 最小:min
  * 转成数组，toList
  * 转成字符串：mkString(“~”)

  ```scala
  //创建
    val set1 = Set(1,2,3,4,4)
    val set2 = Set(1,2,5)
    //遍历
    //注意：set会自动去重
    set1.foreach { print}
    println()
    for(s <- set1){
      print(s)
    }
    println()
    println("*******")
    /**
     * 方法举例
     */
  
    //交集
    val set3 = set1.intersect(set2)
    set3.foreach{print}
    println()
    val set4 = set1.&(set2)
    set4.foreach{print}
    println()
    println("*******")
    //差集
    set1.diff(set2).foreach { println }
    set1.&~(set2).foreach { println }
    //子集
    set1.subsetOf(set2)
  
    //最大值
    println(set1.max)
    //最小值
    println(set1.min)
    println("****")
  
    //转成数组，list
    set1.toArray.foreach{println}
    println("****")
    set1.toList.foreach{println}
  
    //mkString
    println(set1.mkString)
    println(set1.mkString("\t"))
  ```

* 可变长set

  ```scala
  /**
     * 可变长Set
     * @param args
     */
    val set = Set[Int](1, 2, 3, 4, 5)
    set.add(100)
    set.+=(200)
    set.+=(1, 210, 300)
    set.foreach(println())
  ```

* set方法总结

  ```scala
  Scala Set 常用方法
  下表列出了 Scala Set 常用的方法：
  序号	方法及描述
  1	
  def +(elem: A): Set[A]
  为集合添加新元素，x并创建一个新的集合，除非元素已存在
  2	
  def -(elem: A): Set[A]
  移除集合中的元素，并创建一个新的集合
  3	
  def contains(elem: A): Boolean
  如果元素在集合中存在，返回 true，否则返回 false。
  4	
  def &(that: Set[A]): Set[A]
  返回两个集合的交集
  5	
  def &~(that: Set[A]): Set[A]
  返回两个集合的差集
  6	
  def +(elem1: A, elem2: A, elems: A*): Set[A]
  通过添加传入指定集合的元素创建一个新的不可变集合
  7	
  def ++(elems: A): Set[A]
  合并两个集合
  8	
  def -(elem1: A, elem2: A, elems: A*): Set[A]
  通过移除传入指定集合的元素创建一个新的不可变集合
  9	
  def addString(b: StringBuilder): StringBuilder
  将不可变集合的所有元素添加到字符串缓冲区
  10	
  def addString(b: StringBuilder, sep: String): StringBuilder
  将不可变集合的所有元素添加到字符串缓冲区，并使用指定的分隔符
  11	
  def apply(elem: A)
  检测集合中是否包含指定元素
  12	
  def count(p: (A) => Boolean): Int
  计算满足指定条件的集合元素个数
  13	
  def copyToArray(xs: Array[A], start: Int, len: Int): Unit
  复制不可变集合元素到数组
  14	
  def diff(that: Set[A]): Set[A]
  比较两个集合的差集
  15	
  def drop(n: Int): Set[A]]
  返回丢弃前n个元素新集合
  16	
  def dropRight(n: Int): Set[A]
  返回丢弃最后n个元素新集合
  17	
  def dropWhile(p: (A) => Boolean): Set[A]
  从左向右丢弃元素，直到条件p不成立
  18	
  def equals(that: Any): Boolean
  equals 方法可用于任意序列。用于比较系列是否相等。
  19	
  def exists(p: (A) => Boolean): Boolean
  判断不可变集合中指定条件的元素是否存在。
  20	
  def filter(p: (A) => Boolean): Set[A]
  输出符合指定条件的所有不可变集合元素。
  21	
  def find(p: (A) => Boolean): Option[A]
  查找不可变集合中满足指定条件的第一个元素
  22	
  def forall(p: (A) => Boolean): Boolean
  查找不可变集合中满足指定条件的所有元素
  23	
  def foreach(f: (A) => Unit): Unit
  将函数应用到不可变集合的所有元素
  24	
  def head: A
  获取不可变集合的第一个元素
  25	
  def init: Set[A]
  返回所有元素，除了最后一个
  26	
  def intersect(that: Set[A]): Set[A]
  计算两个集合的交集
  27	
  def isEmpty: Boolean
  判断集合是否为空
  28	
  def iterator: Iterator[A]
  创建一个新的迭代器来迭代元素
  29	
  def last: A
  返回最后一个元素
  30	
  def map[B](f: (A) => B): immutable.Set[B]
  通过给定的方法将所有元素重新计算
  31	
  def max: A
  查找最大元素
  32	
  def min: A
  查找最小元素
  33	
  def mkString: String
  集合所有元素作为字符串显示
  34	
  def mkString(sep: String): String
  使用分隔符将集合所有元素作为字符串显示
  35	
  def product: A
  返回不可变集合中数字元素的积。
  36	
  def size: Int
  返回不可变集合元素的数量
  37	
  def splitAt(n: Int): (Set[A], Set[A])
  把不可变集合拆分为两个容器，第一个由前 n 个元素组成，第二个由剩下的元素组成
  38	
  def subsetOf(that: Set[A]): Boolean
  如果集合A中含有子集B返回 true，否则返回false
  39	
  def sum: A
  返回不可变集合中所有数字元素之和
  40	
  def tail: Set[A]
  返回一个不可变集合中除了第一元素之外的其他元素
  41	
  def take(n: Int): Set[A]
  返回前 n 个元素
  42	
  def takeRight(n: Int):Set[A]
  返回后 n 个元素
  43	
  def toArray: Array[A]
  将集合转换为数组
  44	
  def toBuffer[B >: A]: Buffer[B]
  返回缓冲区，包含了不可变集合的所有元素
  45	
  def toList: List[A]
  返回 List，包含了不可变集合的所有元素
  46	
  def toMap[T, U]: Map[T, U]
  返回 Map，包含了不可变集合的所有元素
  47	
  def toSeq: Seq[A]
  返回 Seq，包含了不可变集合的所有元素
  48	
  def toString(): String
  返回一个字符串，以对象来表示
  ```

### Map

* map创建

  * Map（1 –>”bjsxt’）
  * Map((1,”bjsxt”))
  * 注意：创建map时，相同的key被后面的相同的key顶替掉，只保留一个

  ```scala
  val map = Map(
        "1" -> "bjsxt",
        2 -> "shsxt",
        (3,"xasxt")
      )
  ```

* 获取map的值

  * map.get(“1”).get
  * map.get(100).getOrElse(“no value”)：如果map中没有对应项，赋值为getOrElse传的值。

  ```scala
  //获取值
      println(map.get("1").get)
      val result = map.get(8).getOrElse("no value")
      println(result)
  
  ```

* 遍历map

  * for，foreach

  ```scala
  //map遍历
      for(x <- map){
        println("====key:"+x._1+",value:"+x._2)
      }
      map.foreach(f => {
        println("key:"+ f._1+" ,value:"+f._2)
      })
  ```

* 遍历key

  * map.keys

  ```scala
  //遍历key
      val keyIterable = map.keys
      keyIterable.foreach { key => {
        println("key:"+key+", value:"+map.get(key).get)
      } }
      println("---------")
  ```

* 遍历value

  * map.values

  ```scala
  //遍历value
      val valueIterable = map.values
      valueIterable.foreach { value => {
        println("value: "+ value)
      } }
  ```

* 合并map

  * 例：map1.++(map2)  --map1中加入map2
  * 例：map1.++:(map2) –map2中加入map1
  * 合并map会将map中的**相同key的value替换**

  ```scala
  //合并map
    val map1 = Map(
      (1,"a"),
      (2,"b"),
      (3,"c")
    )
    val map2 = Map(
      (1,"aa"),
      (2,"bb"),
      (2,90),
      (4,22),
      (4,"dd")
    )
    map1.++:(map2).foreach(println)
  ```

* map中的方法举例

  * filter:过滤，留下符合条件的记录
  * count:统计符合条件的记录数
  * contains：map中是否包含某个key
  * exist：符合条件的记录存在不存在

  ```scala
  /**
     * map方法
     */
    //count
    val countResult  = map.count(p => {
      p._2.equals("shsxt")
    })
    println(countResult)
  
    //filter
    map.filter(_._2.equals("shsxt")).foreach(println)
  
    //contains
    println(map.contains(2))
  
    //exist
    println(map.exists(f =>{
      f._2.equals("xasxt")
    }))
  ```

* 可变长map

  ```scala
    val map = Map[String, Int]()
    map.put("hello", 100)
    map.put("world", 200)
    //map遍历
    for(x <- map){
        println("====key:"+x._1+",value:"+x._2)
    }
  ```

* Map方法

  ```scala
  Scala Map 方法
  下表列出了 Scala Map 常用的方法：
  序号	方法及描述
  1	
  def ++(xs: Map[(A, B)]): Map[A, B]
  返回一个新的 Map，新的 Map xs 组成
  2	
  def -(elem1: A, elem2: A, elems: A*): Map[A, B]
  返回一个新的 Map, 移除 key 为 elem1, elem2 或其他 elems。
  3	
  def --(xs: GTO[A]): Map[A, B]
  返回一个新的 Map, 移除 xs 对象中对应的 key
  4	
  def get(key: A): Option[B]
  返回指定 key 的值
  5	
  def iterator: Iterator[(A, B)]
  创建新的迭代器，并输出 key/value 对
  6	
  def addString(b: StringBuilder): StringBuilder
  将 Map 中的所有元素附加到StringBuilder，可加入分隔符
  7	
  def addString(b: StringBuilder, sep: String): StringBuilder
  将 Map 中的所有元素附加到StringBuilder，可加入分隔符
  8	
  def apply(key: A): B
  返回指定键的值，如果不存在返回 Map 的默认方法
  
  10	
  def clone(): Map[A, B]
  从一个 Map 复制到另一个 Map
  11	
  def contains(key: A): Boolean
  如果 Map 中存在指定 key，返回 true，否则返回 false。
  12	
  def copyToArray(xs: Array[(A, B)]): Unit
  复制集合到数组
  13	
  def count(p: ((A, B)) => Boolean): Int
  计算满足指定条件的集合元素数量
  14	
  def default(key: A): B
  定义 Map 的默认值，在 key 不存在时返回。
  15	
  def drop(n: Int): Map[A, B]
  返回丢弃前n个元素新集合
  16	
  def dropRight(n: Int): Map[A, B]
  返回丢弃最后n个元素新集合
  17	
  def dropWhile(p: ((A, B)) => Boolean): Map[A, B]
  从左向右丢弃元素，直到条件p不成立
  18	
  def empty: Map[A, B]
  返回相同类型的空 Map
  19	
  def equals(that: Any): Boolean
  如果两个 Map 相等(key/value 均相等)，返回true，否则返回false
  20	
  def exists(p: ((A, B)) => Boolean): Boolean
  判断集合中指定条件的元素是否存在
  21	
  def filter(p: ((A, B))=> Boolean): Map[A, B]
  返回满足指定条件的所有集合
  22	
  def filterKeys(p: (A) => Boolean): Map[A, B]
  返回符合指定条件的的不可变 Map
  23	
  def find(p: ((A, B)) => Boolean): Option[(A, B)]
  查找集合中满足指定条件的第一个元素
  24	
  def foreach(f: ((A, B)) => Unit): Unit
  将函数应用到集合的所有元素
  25	
  def init: Map[A, B]
  返回所有元素，除了最后一个
  26	
  def isEmpty: Boolean
  检测 Map 是否为空
  27	
  def keys: Iterable[A]
  返回所有的key/p>
  28	
  def last: (A, B)
  返回最后一个元素
  29	
  def max: (A, B)
  查找最大元素
  30	
  def min: (A, B)
  查找最小元素
  31	
  def mkString: String
  集合所有元素作为字符串显示
  32	
  def product: (A, B)
  返回集合中数字元素的积。
  33	
  def remove(key: A): Option[B]
  移除指定 key
  34	
  def retain(p: (A, B) => Boolean): Map.this.type
  如果符合满足条件的返回 true
  35	
  def size: Int
  返回 Map 元素的个数
  36	
  def sum: (A, B)
  返回集合中所有数字元素之和
  37	
  def tail: Map[A, B]
  返回一个集合中除了第一元素之外的其他元素
  38	
  def take(n: Int): Map[A, B]
  返回前 n 个元素
  39	
  def takeRight(n: Int): Map[A, B]
  返回后 n 个元素
  40	
  def takeWhile(p: ((A, B)) => Boolean): Map[A, B]
  返回满足指定条件的元素
  41	
  def toArray: Array[(A, B)]
  集合转数组
  42	
  def toBuffer[B >: A]: Buffer[B]
  返回缓冲区，包含了 Map 的所有元素
  43	
  def toList: List[A]
  返回 List，包含了 Map 的所有元素
  44	
  def toSeq: Seq[A]
  返回 Seq，包含了 Map 的所有元素
  45	
  def toSet: Set[A]
  返回 Set，包含了 Map 的所有元素
  46	
  def toString(): String
  返回字符串对象
  ```

  ### 元组

  * 元组定义

    * 与列表一样，与列表不同的是**元组可以包含不同类型的元素**。元组的值是通过将单个的值包含在圆括号中构成的。

  * 创建元组与取值

    * val  tuple = new Tuple（1） 可以使用new
    * val tuple2  = Tuple（1,2） 可以不使用new，也可以直接写成val tuple3 =（1,2,3） 
    * 取值用”._XX” 可以获取元组中的值

    ```scala
    //创建，最多支持22个
      val tuple = new Tuple1(1)
      val tuple2 = Tuple2("zhangsan",2)
      val tuple3 = Tuple3(1,2,3)
      val tuple4 = (1,2,3,4)
      val tuple18 = Tuple18(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18)
      val tuple22 = new Tuple22(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22)
    
      //使用
      println(tuple2._1 + "\t"+tuple2._2)
      val t = Tuple2((1,2),("zhangsan","lisi"))
      println(t._1._2)
    
      def main(args: Array[String]): Unit = {
    
      }
    ```

  * 元组的遍历

    * tuple.productIterator得到迭代器，进而遍历

    ```scala
    //遍历
        val tupleIterator = tuple22.productIterator
        while(tupleIterator.hasNext){
          println(tupleIterator.next())
        }
    ```

  * swap，toString方法， swap元素翻转，只针对二元组

    ```scala
    /**
         * 方法
         */
        //翻转，只针对二元组
        println(tuple2.swap)
        
        //toString
        println(tuple3.toString())
    ```

  
## trait特性

*  概念理解
  
    * Scala Trait(特征) 相当于 Java 的接口，实际上它比接口还功能强大。
    
    * 与接口不同的是，它还可以定义属性和方法的实现。
    
  * 一般情况下Scala的类可以继承多个Trait，从结果来看就是实现了多重继承。Trait(特征) 定义的方式与类类似，但它使用的关键字是 trait
    
  * 举例“trait中带属性带方法实现
  
    * 继承的多个trait中如果有同名的方法和属性，**必须要在类中使用"override"重新定义**
    * trait中不可以传参数
    * 类继承多个trait时使用extends....with...
  
    ```scala
    trait Read {
      val readType = "Read"
      val gender = "m"
      def read(name:String): Unit ={
        println(name + " is reading")
      }
    }
    
    trait Listen {
      val listenType = "Listen"
      val gender = "m"
      def listen(name:String): Unit ={
        println(name + "is listening")
      }
    }
    
    class Human() extends Read with Listen{
      override val gender = "f"
    }
    
    object test {
      //override val gender = " f"
    
      def main(args: Array[String]): Unit = {
        val person = new Human()
        person.read("zhangsan")
        person.listen("lisi")
        println(person.listenType)
        println(person.readType)
        println(person.gender)
    
      }
    }
    ```
  
  * trait中带方法不实现
  
    ```scala
    object LessonTrait2 {
      def main(args: Array[String]): Unit = {
        val p1 = new Point(1,2)
        val p2 = new Point(1,3)
        println(p1.isEqule(p2))
        println(p1.isNotEqule(p2))
      }
    }
    
    trait Equle{
      // 不知道会传什么类型所以传Any，isEqule没有实现，isNotEqule实现了，但是如果一个类继承了，那么就要实现这些方法
      def isEqule(x:Any) :Boolean
      def isNotEqule(x : Any)  = {
        !isEqule(x)
      }
    }
    
    class Point(x:Int, y:Int) extends Equle {
      val xx = x
      val yy = y
    
      def isEqule(p:Any) = {
        // 前面判断其是不是一个Point实例，如果时true，往后走
        p.isInstanceOf[Point] && p.asInstanceOf[Point].xx==xx
      }
    }
    ```
  
## 模板匹配match

  * 概念理解
  
    Scala 提供了强大的模式匹配机制，应用也非常广泛。一个模式匹配包含了一系列备选项，每个都开始于关键字 case。每个备选项都包含了一个模式及一到多个表达式。箭头符号 => 隔开了模式和表达式。

* 代码及注意点

  * 模式匹配不仅可以匹配值还可以匹配类型
  * 从上到下顺序匹配，**如果匹配到则不再往下匹配**
  * 都匹配不上时，会匹配到case _ ,相当于default
  * match 的最外面的”{ }”可以去掉看成一个语句

  ```scala
  object LessonMatch {
    def main(args: Array[String]): Unit = {
      val tuple = Tuple6(1,2,3f,4,"abc",55d)
      val tupleIterator = tuple.productIterator
      while(tupleIterator.hasNext){
        matchTest(tupleIterator.next())
      }
  
    }
    /**
     * 注意点：
     * 1.模式匹配不仅可以匹配值，还可以匹配类型
     * 2.模式匹配中，如果匹配到对应的类型或值，就不再继续往下匹配
     * 3.模式匹配中，都匹配不上时，会匹配到 case _ ，相当于default
     */
    def matchTest(x:Any) ={
      x match {
        case x:Int=> println("type is Int")
        case 1 => println("result is 1")
        case 2 => println("result is 2")
        case 3=> println("result is 3")
        case 4 => println("result is 4")
        case x:String => println("type is String")
        //      case x :Double => println("type is Double")
        case _ => println("no match")
      }
    }
  }
  ```

## 偏函数

​	**如果一个方法中没有match (就是没有match那个大括号)只有case**，这个函数可以定义成PartialFunction偏函数。偏函数定义时，**不能使用括号传参**，默认定义PartialFunction中传入一个值，匹配上了对应的case,返回一个值

```scala
object LessonPartialFunction {
  def MyTest:PartialFunction[String, String]={
      case "scala" => {"scala"}
      case "hello" => {"hello"}
      case _=>{"no match..."}
  }

  def main(args: Array[String]): Unit = {
    println(MyTest("scala"))
  }
}
```

## 样例类(case classes)

* 概念理解

  使用了case关键字的类定义就是样例类(case classes)，样例类是种特殊的类。实现了类构造参数的getter方法（构造参数默认被声明为val），当构造参数是声明为var类型的，它将帮你实现setter和getter方法。

  * 样例类默认帮你实现了toString,equals，copy和hashCode等方法。

  * 样例类可以new, 也可以不用new

  ```scala
  object LessonCaseLesson {
    case class Person1(name:String,age:Int)
    def main(args: Array[String]): Unit = {
      val p1 = new Person1("zhangsan",10)
      val p2 = Person1("lisi",20)
      val p3 = Person1("wangwu",30)
  
      val list = List(p1,p2,p3)
      list.foreach { x => {
        x match {
          case Person1("zhangsan",10) => println("zhangsan")
          case Person1("lisi",20) => println("lisi")
          case _ => println("no match")
        }
      }
      }
    }
  }
  ```

## 隐式转换

​	隐式转换是在Scala编译器进行类型匹配时，如果找不到合适的类型，那么隐式转换会让编译器在作用范围内自动推导出来合适的类型。

### 隐式值与隐式参数

​	隐式值是指在定义参数时前面加上implicit。隐式参数是指在定义方法时，方法中的部分参数是由implicit修饰【必须使用柯里化的方式，将隐式参数写在后面的括号中】。隐式转换作用就是：当调用方法时，不必手动传入方法中的隐式参数，Scala会自动在作用域范围内寻找隐式值自动传入。

```scala
object LessonImplicitTrans {
  def student(age:Int)(implicit name:String, i : Int){
    println(s"student : $name, age = $age, score = $i")
  }

  def Teacher(implicit name:String): Unit ={
    println(s"teacher name is = $name")
  }
//  def sayName(implicit name:String): Unit ={
//    println(name + " is a student")
//  }
  def main(args: Array[String]): Unit = {
    implicit val name:String = "zhangsan"
    implicit val sr = 100
    student(18)
    Teacher
  }
}
```



隐式值和隐式参数注意：

* 同类型的参数的隐式值只能在作用域内出现一次，同一个作用域内不能定义多个类型一样的隐式值。
* implicit 关键字必须放在隐式参数定义的开头
* 一个方法只有一个参数是隐式转换参数时，那么可以直接定义implicit关键字修饰的参数，调用时直接创建类型不传入参数即可。
* 一个方法如果有多个参数，要实现部分参数的隐式转换,必须使用柯里化这种方式,隐式关键字出现在后面，只能出现一次

### 隐式转换函数

​	隐式转换函数是使用关键字implicit修饰的方法。当Scala运行时，假设如果A类型变量调用了method()这个方法，发现A类型的变量没有method()方法，而B类型有此method()方法，**会在作用域中寻找有没有隐式转换函数将A类型转换成B类型，如果有隐式转换函数，那么A类型就可以调用method()这个方法**。
​	隐式转换函数注意：隐式转换函数只与函数的参数类型和返回类型有关，与函数名称无关，所以作用域内不能有相同的参数类型和返回类型的不同名称隐式转换函数。

```scala
class Animal(name:String){
  def canFly(): Unit ={
    println(s"$name can fly....")
  }
}
class Rabbit(xname : String){
  val name = xname
}

object LessonImplicitFunction {
  implicit def rabbitToAnimal(rabbit: Rabbit):Animal = {
    new Animal(rabbit.name)
  }

  def main(args: Array[String]): Unit = {
    val rabbit = new Rabbit("RABBIT")
    // 在作用域中寻找有没有隐式转换函数将rabbit转换成animal，如果有，就调用该函数
    rabbit.canFly()
  }
}
```

### 隐式类

使用implicit关键字修饰的类就是隐式类。若一个变量A没有某些方法或者某些变量时，而这个变量A可以调用某些方法或者某些变量时，可以定义一个隐式类，隐式类中定义这些方法或者变量，隐式类中传入A即可。
隐式类注意：

* 隐式类必须定义在类，包对象，伴生对象中。
* 隐式类的构造必须只有一个参数，同一个类，包对象，伴生对象中不能出现同类型构造的隐式类。

```scala
class Animal(name:String){
  def canFly(): Unit ={
    println(s"$name can fly....")
  }
}
class Rabbit1(xname : String){
  val name = xname
}

object LessonImplicitFunction {
  implicit def rabbitToAnimal(rabbit: Rabbit1):Animal = {
    new Animal(rabbit.name)
  }

  def main(args: Array[String]): Unit = {
    val rabbit = new Rabbit1("RABBIT")
    // 在作用域中寻找有没有隐式转换函数将rabbit转换成animal，如果有，就调用该函数
    rabbit.canFly()
  }
}
```

## Actor Model

* 概念理解

  ​	Actor Model是用来编写并行计算或分布式系统的高层次抽象（类似java中的Thread）让程序员不必为**多线程模式**下共享锁而烦恼,被用在Erlang 语言上, 高可用性99.9999999 % 一年只有31ms 宕机Actors将状态和行为封装在一个轻量的进程/线程中，但是不和其他Actors分享状态，每个Actors有自己的世界观，当需要和其他Actors交互时，通过发送事件和消息，**发送是异步的，非堵塞的(fire-andforget)**，发送消息后**不必等另外Actors回复，也不必暂停**，每个Actors有自己的消息队列，进来的消息按先来后到排列，这就有很好的并发策略和可伸缩性，可以建立性能很好的事件驱动系统.
  
* Actor特征

  * ActorModel是消息传递模型,基本特征就是消息传递
  * 消息发送是异步的，非阻塞的
  * 消息一旦发送成功，不能修改
  * Actor之间传递时，自己决定决定去检查消息，而不是一直等待，是异步非阻塞的

* Akka

  ​	Akka 是一个用 Scala 编写的库，用于简化编写容错的、高可伸缩性的 Java 和Scala 的 Actor 模型应用，底层实现就是Actor,Akka是一个开发库和运行环境，可以用于构建高并发、分布式、可容错、事件驱动的基于JVM的应用。使构建高并发的分布式应用更加容易。
  ​	spark1.6版本之前，spark分布式节点之间的消息传递使用的就是Akka，底层也就是actor实现的。**1.6之后使用的netty传输。**

actor发送接受消息样例

```scala
import scala.actors.Actor

class myActor extends Actor{
  def act(): Unit ={
    while (true){
      receive{
        case x:String => println("get String = " + x)
        case x:Int => println("get Int")
        case _ => println("get default")
      }
    }
  }
}
object LessonActor {
  def main(args: Array[String]): Unit = {
    val actor = new myActor()
    actor.start()
    actor ! "I love scala"
  }
}
```

actor与actor之间互相通信

```scala
import scala.actors.Actor
case class Message(actor:Actor,msg:Any)
class Actor1 extends Actor{
  def act(){
    while(true){
      receive{
        case  msg :Message => {
          println("i sava msg! = "+ msg.msg)

          msg.actor!"i love you too !"
        }
        case msg :String => println(msg)
        case  _ => println("default msg!")
      }
    }
  } 
}

class Actor2(actor :Actor) extends Actor{
  actor ! Message(this,"i love you !")
  def act(){
    while(true){
      receive{
        case msg :String => {
          if(msg.equals("i love you too !")){
            println(msg)
            actor! "could we have a date !"
          }
        }
        case  _ => println("default msg!")
      }
    }
  }
}
object LessonActorCommunicate {
  def main(args: Array[String]): Unit = {
    val actor1 = new Actor1()
    actor1.start()
    val actor2 = new Actor2(actor1)
    actor2.start()
  }
}
```

