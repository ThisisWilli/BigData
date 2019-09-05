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

  



