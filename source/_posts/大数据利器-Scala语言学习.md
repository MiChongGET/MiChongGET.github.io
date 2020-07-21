---
title: 大数据利器--Scala语言学习(基础)
date: 2020-07-12 16:21:30
tags:
- Scala
categories:
- 编程语言
description: Scala combines object-oriented and functional programming in one concise, high-level language. Scala's static types help avoid bugs in complex applications, and its JVM and JavaScript runtimes let you build high-performance systems with easy access to huge ecosystems of libraries.
top_img: https://yanxuan.nosdn.127.net/5d836670c025c0771c404a98bf61d8e9.png
cover: https://file.buildworld.cn/img/2e9eb5fbd273a7db40541819e49bf576_scala-icon.png
---
## Scala

> Scala combines object-oriented and functional programming in one concise, high-level language. Scala's static types help avoid bugs in complex applications, and its JVM and JavaScript runtimes let you build high-performance systems with easy access to huge ecosystems of libraries.



> 给大家推荐一个在线的scala文档网站：https://static.runoob.com/download/Scala%E8%AF%AD%E8%A8%80%E8%A7%84%E8%8C%83.pdf



### 1、Scala基本的程序结构说明

```scala
//对 scala 的基本的程序结构说明
//1. object 是一个关键字，表示一个伴生对象
//2. 如果该文件只出现了一个 object HelloScala 就会在编译后两个.class 文件
//3. 第一个文件是 HelloScala.class 这个表示他的伴生类，但是空的.
//4. 第 2 个文件是 HelloScala$.class 对应的是 object HelloScala,但是本质是调用它对应的一个静态属性 MODULE$
//5. 这两个文件的关系和 main 函数的入口关系一会分析
object HelloScala {
  // 1. def 表示一个方法或者一个函数
  // 2. main 表示入口
  // 3. args: Array[String] 表示形参，args 是形参名 Array[String] 是形参类型表示一个 Array 数组
  // 4. :Unit 表示返回值类型为 Unit ，等价于 java 的 void
  // 5. = 表示 后面写的是函数体/方法体, 它还有返回值类型推导的作用
  def main(args: Array[String]):Unit = {
    // 表示是 输出， 类似 System.out.println("hello, scala 世界!")
    // 在 scala 语句后，不需要带; //体现简洁
    println("hello, scala 世界!")
  }
}
```

### 2、Scala 的数据类型一览图

![](https://file.buildworld.cn/img/20200712154042.png)

- 1) `Any` 是所有类的根类型,即所有类的父类(基类)

- 2) 在 `Scala`中类分为两个大的类型分支(`AnyVal` [**值类型，即可以理解成就是 java 的基本数据类型**],`AnyRef` 类型)

- 3) 在 `AnyVal` 虽然叫值类型，但是仍然是**类(对象)**

- 4) 在 `Scala`中有两个特别的类型(`Null` ), 还有一个是 `Nothing`

- 5) `Null` 类型只有一个实例 `null`, 他是 `bottom class` ,是 `AnyRef` 的子类.

- 6) `Nothing` 类型是所有类的子类， 它的价值是在于因为它是所有类的子类，就可以将 `Nothing` 类型的对象返回给任意的变量或者方法，比如案例

  ```scala
  def f1():Nothing= { //表示 f1 方法就是没有正常的返回值，专门用于返回异常
  throw new Exception("异常发生")
  }
  ```

- 7) 在 `Scala`中仍然遵守 低精度的数据自动的转成高精度的数据类型。

- 8) 在 `Scala`中，`Unit` 类型比较特殊，这个类型也只有一个实例 ()  



#### Scala数据类型列表

| 数据类型 | 描述                                                         |
| :------: | ------------------------------------------------------------ |
|   Byte   | 8位有符号补码整数。数值区间为 -128 到 127                    |
|  Short   | 16位有符号补码整数。数值区间为 -32768 到 32767               |
|   Int    | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647     |
|   Long   | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 |
|  Float   | 32 位 IEEE 754标准的单精度浮点数                             |
|  Double  | 64 位 IEEE 754标准的双精度浮点数                             |
|   Char   | 16位无符号Unicode字符, 区间值为 U+0000 到 U+FFFF             |
|  String  | 字符序列                                                     |
| Boolean  | true或false                                                  |
|   Unit   | 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。 |
|   Null   | **null  可以赋值给任意引用类型(AnyRef)，但是不能赋值给值类型** |
| Nothing  | Nothing类型在Scala的类层级的最低端；它是任何其他类型的子类型。 |
|   Any    | Any是所有其他类的超类                                        |
|  AnyRef  | AnyRef类是Scala里所有引用类(reference class)的基类           |

### 3、函数式编程

#### 函数式编程基础
> - 1) 函数定义/声明
> - 2) 函数运行机制
> - 3) 递归 [**推荐编程者递归来解决问题, 算法基础, 邮差问题，最短路径，背包问题, 迷宫，回溯** ]
> - 4) 过程
> - 5) 惰性函数和异常

#### 函数式编程高级
> - 1) 值函数(函数字面量)
> - 2) 高阶函数
> -  3) 闭包
> - 4) 应用函数
> - 5) 柯里化函数，抽象控制..



**在 Scala 当中，函数是一等公民，像变量一样，既可以作为函数的参数使用，也可以将函数赋值给一个变量. ，函数的创建不用依赖于类或者对象，而在 Java 当中，函数的创建则要依赖于类、抽象类或者接口。**



#### 惰性函数

> 当函数返回值被声明为 lazy 时，函数的执行将被推迟，直到我们首次对此取值，该函数才会执行。这种函数我们称之为惰性函数，在 Java 的某些框架代码中称之为懒加载(延迟加载）,Java中没有原生方法。

- 1) `lazy` 不能修饰 `var` 类型的变量
- 2) 不但是在调用函数时，加了 `lazy` ,会导致函数的执行被推迟，我们在声明一个变量时，如果给声明了 `lazy` ,那么变量值得分配也会推迟。 比如 `lazy val i = 10`

```scala
object Demo1 {

  def main(args: Array[String]): Unit = {

    lazy val res = sum(1, 2, 3, 4, 6)
    println(res) //使用res时，才会真正的开始计算

  }
  def sum(args: Int*): Int = {
    var res = 0
    for (n <- args) {
      res += n
    }
    res
  }
}
```

#### 异常

```scala
def main(args: Array[String]): Unit = {
    
    //scala 中去掉所谓的 checked（编译） 异常
    //设计者认为，如果程序员编程时，认为某段代码可疑，就直接 try 并处理
    //说明
    //1. 如果代码可疑，使用 try 进行处理
    //2. 在 catch 中，可以有多个 case ，对可能的异常进行匹配
    //3. case ex: Exception => println("异常信息=" + ex.getMessage)
    // (1) case 是一个关键字
    // (2) ex: Exception 异常的种类
    // (3) => 表明后的代码是对异常进行处理,如果处理的代码有多条语句可以{}扩起
    //4. 在 scala 中把范围小的异常放在后面，语法不会报错，但是不推荐
    //5. 如果捕获异常，代码即使出现异常，程序也不会崩溃。
    
  try{
    var n = 10 /0
  }catch {
    case ex:ArithmeticException=>{
      println("异常："+ex.getMessage)
    }
    case exception: Exception=>{
      println(exception.getMessage)
    }
  }finally {
    printf("完成")
  }
}
```



> Scala 提供了 `throws` 关键字来声明异常。可以使用方法定义声明异常。 它向调用者函数提供了此方法可能引发此异常的信息。 它有助于调用函数处理并将该代码包含在 try-catch 块中，以避免程序异常终止。在 scala 中，可以使用`@throws` 注释来声明异常

```scala
@throws (classOf[ArithmeticException])
def function1(): Unit ={
  var n = 10/0
}
```

### 4、Scala 构造器的基本语法

```scala
class  类名( 形参列表) { //  主构造器
//  类体
def this( 形参列表) { //  辅助构造器
	}
def this( 形参列表) { // 辅助构造器可以有多个...
	}
}
```

#### 属性高级

> - 1) Scala 类的主构造器的形参`未用任何修饰符修饰`，那么这个参数是局部变量。
> - 2) 如果参数使用 `val` 关键字声明，那么 Scala 会将参数作为类的私有的只读属性使用
> - 3) 如果参数使用 `var` 关键字声明，那么那么 Scala 会将参数作为类的成员属性使用,并会提供属性对应的xxx()[类似 getter]/xxx_$eq()[类似 setter]方法，即这时的成员属性是私有的，但是可读写。

#### Bean 属性

> JavaBeans 规范定义了 Java 的属性是像 getXxx（）和 setXxx（）的方法。许多 Java 工具（框架）都依赖这个命名习惯。为了 Java 的互操作性。将 Scala 字段加`@BeanProperty` 时，这样会自动生成规范的 setXxx/getXxx 方法。这时可以使用 对象.setXxx() 和 对象.getXxx() 来调用属性。

- 1) 给某个属性加入`@BeanPropetry` 注解后，会生成 getXXX 和 setXXX 的方法
- 2) 并且对 原来底层自动生成类似 xxx(),xxx_$eq()

```scala
class Cat {
    
  @BeanProperty var name: String = ""
  @BeanProperty var age: Int = 0
  @BeanProperty var color: String = ""

  def this(name: String) {
    this()
    this.name = name
  }

  override def toString = s"Cat($name, $age, $color)"
}
```



### 5、Scala 中包的可见性和访问修饰符的使用

> - 1) 当属性访问权限为默认时，从底层看属性是 private 的，但是因为提供了 xxx_$eq()[类似 setter]/xxx()[类似getter] 方法，因此从使用效果看是任何地方都可以访问)。
> - 2) 当方法访问权限为默认时，默认为 public 访问权限。
> - 3) `private` 为私有权限，只在类的内部和伴生对象中可用。
> - 4) `protected` 为受保护权限，scala  中受保护权限比 Java  中更严格 ， **只能子类访问，问同包无法访问** (编译器从语法层面控制)。
> - 5) **在 scala 中没有 public 关键字**,即不能用 public 显式的修饰属性和方法。

**scala 设计者将访问的方式分成三大类: (1) 处处可以访问 public (2) 子类和伴生对象能访问 protected (3) 本类和伴生对象访问 private**

```scala
class Dog {
//import 可以放在任何的地方，同时他的作用范围就是{} 块中
//import 如果使用到 3 次及以上，则可以放在文件前面,否则可以使用就近引入.
import scala.beans.BeanProperty
@BeanProperty var name : String = ""
}

//Java 中如果想要导入包中所有的类，可以通过通配符*，Scala 中采用下 _
//如果不想要某个包中全部的类，而是其中的几个类，可以采用选取器,使用{} 括起来即可。

//如果有多个同名的类或者 trait 等，可以使用 scala 重命名的机制来解决.
import java.util.{ HashMap=>JavaHashMap, List}

//如果某个冲突的类根本就不会用到，那么这个类可以直接隐藏掉
import java.util.{ HashMap=>_, _} // 含义为 引入 java.util 包的所有类，但是忽略 HahsMap 类
```



### 6、继承

#### 重写方法

**Scala 明确规定， 重写一个非抽象方法需要用 override  修饰符，调用超类的方法使用 super  关键字**

```scala
def main(args: Array[String]): Unit = {
  var student = new student
  student.name = "michong"
  student.num = "123123"

  student.getInfo()
}

class person {

  var name: String = _

  def getInfo(): Unit = {
    println("姓名：" + name)
  }
}

class student extends person {

  var num: String = _

  override def getInfo(): Unit = {
    super.getInfo()
    println("学号：" + num)
  }
}
```



#### Scala 中类型检查和转换

**基本介绍**

> 要测试某个对象是否属于某个给定的类，可以用 `isInstanceOf` 方法。用 `asInstanceOf` 方法将引用转换为子类的引用。`classOf` 获取对象的类名。

- 1) classOf[String]就如同 Java 的 String.class 。
- 2) obj.isInstanceOf[T]就如同 Java 的 obj instanceof T 判断 obj 是不是 T 类型。
- 3) obj.asInstanceOf[T]就如同 Java 的(T)obj 将 obj 强转成 T 类型

```scala
var student:person = new student
student.name = "michong"
student.asInstanceOf[student].num = "123123"
```



#### var 重写抽象的 var 属性小结

> - 一个属性没有初始化，那么这个属性就是抽象属性
> - 抽象属性在编译成字节码文件时，属性并不会声明，但是会自动生成抽象方法，所以类必须声明为抽象类
> - 如果是覆写一个父类的抽象属性，那么 `override` 关键字可省略 [原因：父类的抽象属性，生成的是抽象方法，因此就不涉及到方法重写的概念，因此 `override` 可省略]




#### Scala 抽象类

> - 1) 抽象类不能被实例
> - 2) 抽象类不一定要包含 abstract 方法。也就是说,抽象类可以没有 abstract 方法
> - 3) 一旦类包含了抽象方法或者抽象属性,则这个类必须声明为 abstract
> - 4) 抽象方法不能有主体，不允许使用 abstract 修饰。
> - 5) 如果一个类继承了抽象类，则它必须实现抽象类的所有抽象方法和抽象属性，除非它自己也声明为 abstract类。【案例演示+反编译】
> - 6) 抽象方法和抽象属性不能使用 private、final 来修饰，因为这些关键字都是和重写/实现相违背的。
> - 7) 抽象类中可以有实现的方法.
> - 8) 子类重写抽象方法不需要 override，写上也不会错.

```scala
abstract class person {
  var name:String
}

class student extends person {
  var name:String=""
}
```



#### 匿名子类

```scala
def main(args: Array[String]): Unit = {
  //抽象类，他不能实例化，我们可以通过匿名子类的方式创建一个实例
  val p = new person {
    override var name: String = "123"
    override def info(): Unit = {
      println("test")
    }
  }
  p.info()
}

abstract class person {
  var name:String
  def info()
	}
}
```

### 7、伴生对象

> - 1) Scala 中伴生对象采用 `object` 关键字声明，伴生对象中声明的全是 "`静态`"内容，可以通过伴生对象名称直接调用。
> - 2) 伴生对象对应的类称之为伴生类，伴生对象的名称应该和伴生类名一致。
> - 3) 伴生对象中的属性和方法都可以通过伴生对象名直接调用访问
> - 4) 从语法角度来讲，所谓的伴生对象其实就是类的静态方法和静态变量的集合
> - 5) 从技术角度来讲，scala 还是没有生成静态的内容，只不过是将伴生对象生成了一个新的类，实现属性和方法的调用。[反编译看源码]
> - 6) 从底层原理看，伴生对象实现静态特性是依赖于 `public static final MODULE$` 实现的。
> - 7)  伴生对象的声明应该和伴生类的声明在同一个源码文件中(如果不在同一个文件中会运行错误!)，但是如果没有伴生类，也就没有所谓的伴生对象了，所以放在哪里就无所谓了。
> - 8) 如果 class A 独立存在，那么 A 就是一个类， 如果 object A 独立存在，那么 A 就是一个"静态"性质的对象[即类对象], 在 object A 中声明的属性和方法可以通过 A.属性 和 A.方法 来实现调用

**案例**

```scala
package cn.buildworld.scala.day2

object demo1 {
  def main(args: Array[String]): Unit = {

    val p1 = new Person("michong")
    val p2 = new Person("qjzxzxd")
    val p3 = new Person("米虫")

    Person.joinGroup(p1)
    Person.joinGroup(p2)
    Person.joinGroup(p3)

    Person.showInfo()
  }

  class Person(pname: String) {
    var name: String = pname
  }

  object Person {

    var num: Int = 0

    def joinGroup(person: Person): Unit = {
      num += 1
      println(person.name + "--加入组织")
    }

     def showInfo(): Unit ={
       println("当前"+num+"人")
     }

    def say(): Unit = {
      println("Say Hi")
    }
  }
}
```

### 8、特质(trait)

#### **trait的声明**

```scala
trait 名 特质名 {
trait 体 体
}

1) trait 命名 一般首字母大写.
2) 在 scala 中，java 中的接口可以当做特质使用
```

**trait使用**

```scala
#没有父类
class 类名 extends 质 特质 1 with 质 特质 2 with 质 特质 3 ..
#有父类
class 类名 extends 父类 with 质 特质 1 with 质 特质 2 with  
```

#### **带有特质的对象，动态混入**

> - 1) 除了可以在类声明时继承特质以外，还可以在构建对象时混入特质，扩展目标类的功能【反编译看动态混入本质】
> - 2) 此种方式也可以应用于对抽象类功能进行扩展
> - 3) 动态混入是 Scala 特有的方式（java 没有动态混入），可在不修改类声明/定义的情况下，扩展类的功能，非常的灵活，耦合性低 。
> - 4) 动态混入可以在不影响原有的继承关系的基础上，给指定的类扩展功能。[如何理解]
> - 5) 抽象类中有 抽象的方法，如何动态混入特质->可以，在创建实例时，实现抽象方法即可



```scala
package cn.buildworld.scala.day2

object demo2 {

  def main(args: Array[String]): Unit = {

    val c = new C
    c.getConnect("root","123456")

    // 带有特质的对象，动态混入
    val b = new B with Trait1{
      override def getConnect(user: String, pwd: String): Unit = {
        println("用户登录信息：user: "+user+" pwd: "+pwd)
      }
    }

    b.getConnect("root","123456")
  }

  trait Trait1{
    def getConnect(user:String,pwd:String): Unit
  }

  class A{}

  class B extends A{}

  class C extends A with Trait1{
    override def getConnect(user: String, pwd: String): Unit = {
      println("用户登录信息：user: "+user+" pwd: "+pwd)
    }
  }

  class D{}

  class E extends D with Trait1{
    override def getConnect(user: String, pwd: String): Unit = {}
  }
}
```

#### **特质构造顺序**

> 特质也是有构造器的，构造器中的内容由“字段的初始化”和一些其他语句构成。具体实现请参考“特质叠加”



- **第一种特质构造顺序(声明类的同时混入特质)**

> 1) 调用当前类的超类构造器
> 2) 第一个特质的父特质构造器
> 3) 第一个特质构造器
> 4) 第二个特质构造器的父特质构造器, 如果已经执行过，就不再执行
> 5) 第二个特质构造器
> 6) .......重复 4,5 的步骤(如果有第 3 个，第 4 个特质)

- **第 2 种特质构造顺序(在构建对象时，动态混入特质)**

> 1) 调用当前类的超类构造
> 2) 当前类构造
> 3) 第一个特质构造器的父特质构造器
> 4) 第一个特质构造器.
> 5) 第二个特质构造器的父特质构造器, 如果已经执行过，就不再执行
> 6) 第二个特质构造器
> 7) ......重复 5,6 的步骤(如果有第 3 个，第 4 个特质)

#### 自身类型

```scala
#在Logger中已经可以使用Exception中的相关的方法了
trait Logger{
  this:Exception=>

  def log()={
    println(getMessage)
  }
}

#使用MyLogger时先继承Exception类
class MyLogger extends Exception with Logger {}
```

#### 嵌套

> - 方式 1
>   内部类如果想要访问外部类的属性，可以通过外部类对象访问。即：访问方式：**外部类名.this.属性名**
> - 方式 2
>   内部类如果想要访问外部类的属性，也可以通过外部类别名访问(推荐)。即：**访问方式：外部类名别名.属性名** 【外部类名.this 等价 外部类名别名】
>

```scala
class AAA{

  myOuter=>
  class InnerAAA{

    //使用别名的方式来访问外部类的属性和方法，相当于 myouter 是一个外部类的实例 AAA.this
    //这时需要将外部类的属性和方法的定义/声明放在别名后
    def test(innerAAA: InnerAAA): Unit ={
      println(innerAAA)

      println(myOuter.name)
      println(myOuter.sal)
    }
  }

  var name:String=_
  private var sal:Double=_
}
```

### 9、隐式转换

#### 隐式值

> 隐式值也叫隐式变量，将某个形参变量标记为 implicit，所以编译器会在方法省略隐式参数的情况下去搜索作用域内的隐式值作为缺省参数

```scala
package cn.buildworld.scala.day2

object demo5 {

  def main(args: Array[String]): Unit = {

    //隐式值
    implicit val str1:String = "michong"

    def hello(implicit name:String): Unit ={
      println(name)
    }
    hello

  }

}
```

#### 隐式类

> - 1) 其所带的 构造参数有且只能有一个
> - 2) 隐式类必须被定义在“类”或“伴生对象”或“包对象”里，即隐式类不能是顶级的(top-level objects)
> - 3) 隐式类不能是 case class（case class 在后续介绍样例类）
> - 4) 作用域内不能有与之相同名称的标识符

```scala
package cn.buildworld.scala.day2

object demo5 {

  def main(args: Array[String]): Unit = {

    val db = new MySql
    db.add()
  }

    
  //隐式类
  implicit class DB(val m :MySql){
    def add(): Unit ={
      println("添加")
    }
  }
  class MySql{}

}
```



#### 隐式函数

```scala
package cn.buildworld.scala.day2

object demo4 {

  def main(args: Array[String]): Unit = {

    //隐式函数
    implicit def addDelete(mySql: MySql):DB={
       new DB
    }
 
    var db = new MySql
    db.delete()

  }

  class MySql{}

  class DB {
    def delete(): Unit ={
      println("删除数据！！！")
    }
  }
}
```

#### 隐式解析机制

> - 1) 首先会在 当前代码作用域下查找隐式实体（隐式方法、隐式类、隐式对象）。(一般是这种情况)
>
> - 2) 如果第一条规则查找隐式实体失败，会继续在隐式参数的类型的作用域里查找。类型的作用域是指与该类型相关联的全部伴生模块，一个隐式实体的类型 T 它的查找范围如下( 第二种情况范围广且复杂在使用时，应当尽量避免出现)：
> - a) 如果 T 被定义为 T with A with B with C,那么 A,B,C 都是 T 的部分，在 T 的隐式解析过程中，它们的伴生对象都会被搜索。
>
> - b) 如果 T 是参数化类型，那么类型参数和与类型参数相关联的部分都算作 T 的部分，比如 List[String]的隐式搜索会搜索 List 的伴生对象和 String 的伴生对象。
>
> - c) 如果 T 是一个单例类型 p.T，即 T 是属于某个 p 对象内，那么这个 p 对象也会被搜索。
>
> - d) 如果 T 是个类型注入 S#T，那么 S 和 T 都会被搜索。