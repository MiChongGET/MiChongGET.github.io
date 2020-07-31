---
title: 大数据利器--Scala语言学习(高级)
date: 2020-07-20 15:24:08
tags:
- Scala
categories:
- 编程语言
description: Scala combines object-oriented and functional programming in one concise, high-level language. Scala's static types help avoid bugs in complex applications, and its JVM and JavaScript runtimes let you build high-performance systems with easy access to huge ecosystems of libraries.
top_img: https://yanxuan.nosdn.127.net/18050f4caec3423ca456d74729f8a9ea.png
cover: https://file.buildworld.cn/img/2e9eb5fbd273a7db40541819e49bf576_scala-icon.png
---
## Scala高级

## 一、集合

> Scala 的集合有三大类：`序列 Seq、集 Set、映射 Map`，所有的集合都扩展自 Iterable 特质，在 Scala 中集合有**可变（mutable）**和**不可变（immutable）**两种类型。

![](https://file.buildworld.cn/img/20200720092724.png)

> - 1.Set、Map 是 Java 中也有的集合。
> - 2.`Seq` 是 Java 没有的，我们发现 List 归属到 Seq 了,因此这里的 List 就和 java 不是同一个概念了。
> - 3.我们前面的 for 循环有一个 1 to 3 ,就是 `IndexedSeq` 下的 `Vector`。
> - 4.String 也是属于 `IndexeSeq`。
> - 5.我们发现经典的数据结构比如 `Queue` 和 `Stack` 被归属到 `LinearSeq`。
> - 6.大家注意 Scala 中的 Map 体系有一个 `SortedMap`,说明 Scala 的 Map 可以支持排序。
> - 7.`IndexSeq` 和 `LinearSeq` 的区别[**IndexSeq 是通过索引来查找和定位，因此速度快，比如 String 就是一个索引集合，通过索引即可定位**] [**LineaSeq 是线型的，即有头尾的概念，这种数据结构一般是通过遍历来查找，它的价值在于应用到一些。具体的应用场景 (电商网站, 大数据推荐系统 :最近浏览的 10 个商品)**]

### 1、数组

#### 不可变数组

```scala
//第一种创建方法
val num = new Array[Int](5)
//赋值,集合元素采用小括号访问
num(1) = 10


//第二种创建方法,在定义数组时，直接赋值//使用 apply 方法创建数组对象
val num2 = Array(1,2,3,4,5,6)
```

#### 可变数组

**变长数组(声明泛型)**

```scala
val num = ArrayBuffer[Any](1, "michong", 3.14)
//添加
num.appendAll("hi")
for (i <- num) {
  println(i)
}
//删除
num.remove(4)

//修改
num(0) = "hello"
```

#### 相互转化

> - 1) `arr1.toBuffer` //定长数组转可变数组
> - 2) `arr2.toArray` //可变数组转定长数组



### 2、元组

```scala
package cn.buildworld.scala.day3

object demo2 {
  def main(args: Array[String]): Unit = {

    //创建元组
    val tuple = (1,2,"hello")
    //访问元组
    //1. 使用 _顺序号
    println(tuple._2) // "hello"
    //2. 使用
    println(tuple.productElement(2)) //下标是从 0 开始计算
    //遍历元组
    for(i<-tuple.productIterator){
      println(i)
    }

  }

  final case class Tuple3[+T1, +T2, +T3](_1: T1, _2: T2, _3: T3) extends Product3[T1, T2, T3] {
    override def toString: String = "(" + _1 + "," + _2 + "," + _3 + ")"
  }
}
```

### 3、List

> - 1) List 默认为不可变的集合
> - 2) List 在 scala 包对象声明的,因此不需要引入其它包也可以使用
> - 3) val List = scala.collection.immutable.List
> - 4) List 中可以放任何数据类型，比如 arr1 的类型为 List[Any]
> - 5) 如果希望得到一个空列表，可以使用 Nil 对象, 在 scala 包对象声明的,因此不需要引入其它包也可以使用

```scala
val list = List(1,2,3)
println(list )

//创建一个空list
val list02 = Nil
println(list02)

//在list后面添加元素
val list03 = list:+4
println(list03)

//在list前面添加元素
val list04 = "hi"+:list
println(list04)

//符号::表示向集合中,新建集合添加元素。从右向左
val list5 = 4 :: 5 :: 6 :: list :: Nil
println(list5)

// ::: 运算符是将集合中的每一个元素加入到空集合中去, ::: 左右两边需要时集合.
val list6 = 4 :: 5 :: 6 :: list ::: Nil
println(list6)

//结果
List(1, 2, 3)
List()
List(1, 2, 3, 4)
List(hi, 1, 2, 3)
List(4, 5, 6, List(1, 2, 3))
List(4, 5, 6, 1, 2, 3)
```

#### 列表ListBuffer

**ListBuffer:ListBuffer 是可变的 list 集合，可以添加，删除元素,ListBuffer 属于序**

```scala
object demo4 {
  def main(args: Array[String]): Unit = {

    val list = ListBuffer[Int](1, 2, 3)
    list.addOne(4)
    list.append(5)
    println(list)

    list += 6
    println(list)

    //++ 表示的是加入的是集合中的各个元素
    val list2 = list ++ list
    println(list2)

    list2.remove(0)
    println(list2)
  }
}

//输出
ListBuffer(1, 2, 3, 4, 5)
ListBuffer(1, 2, 3, 4, 5, 6)
ListBuffer(1, 2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6)
ListBuffer(2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6)
```

### 4、队列

> - 1) 队列是一个有序列表，在底层可以用数组或是链表来实现。
> - 2) 其输入和输出要遵循先入先出的原则。即：先存入队列的数据，要先取出。后存入的要后取出。
> - 3) 在 Scala 中，由设计者直接给我们提供队列类型使用。
> - 4) 在 scala 中, 有 `scala.collection.mutable.Queue` 和 `scala.collection.immutable.Queue` , 一般来说，我们在开发中通常使用可变集合中的队列

```scala
import scala.collection.mutable

object demo5 {
  def main(args: Array[String]): Unit = {

    //创建队列
    val q1  = new mutable.Queue[Int]

    //追加数据
    q1+=1
    q1+=2
    println(q1)

    //添加list
    q1++=List(3,4)
    println(q1)

    //取出队列最前面的那个元素
    println(q1.dequeue())
    println(q1)

    //队尾追加元素
    q1.enqueue(5,6,7)
    println(q1)

    //返回队列的第一个元素
    println(q1.head)
    //返回队列的最后一个元素
    println(q1.last)

    //返回除第一个元素以外的其他元素的队列
    println(q1.tail)
  }
}


//结果
Queue(1, 2)
Queue(1, 2, 3, 4)
1
Queue(2, 3, 4)
Queue(2, 3, 4, 5, 6, 7)
2
7
Queue(3, 4, 5, 6, 7)
```

### 5、映射--Map

> - 1) Scala 中的 Map 和 Java 类似，也是一个散列表，它存储的内容也是键值对(key-value)映射，Scala 中**不可变的 Map 是有序的**，**可变的 Map 是无序的**。
> - 2) Scala 中，有`可变 Map (scala.collection.mutable.Map)` 和 `不可变 Map(scala.collection.immutable.Map)`。

#### 构建map

```scala
//构造不可变map，输入输出结果一致
val map1 = Map("MiChong" -> 25, "Zz" -> 24, "Alice" -> 18)
println(map1)

//构造可变map
val map2 = scala.collection.mutable.Map("MiChong" -> 25, "Zz" -> 24, "Alice" -> 18)
println(map2)

//创建空的map
val map3 = new mutable.HashMap[String,Int]
println(map3)

//对偶元组,即创建包含键值对的二元组
val map4 = mutable.Map(("A",1),("B",2),("C",3))
println(map4("A"))
```

#### 取值

```scala
object demo7 {
  def main(args: Array[String]): Unit = {

    val map1 = Map("MiChong" -> 25, "Zz" -> 24, "Alice" -> 18)

    //方式1
    //    1) 如果 key 存在，则返回对应的值
    //    2) 如果 key 不存在，则抛出异常[java.util.NoSuchElementException]
    println(map1("Zz"))

    //方式2、
    // 1.如果 key 存在，则返回 true
    // 2.如果 key 不存在，则返回 false
    println(map1.contains("Zz"))

    //方式3、通过 映射.get(键) 这样的调用返回一个 Option 对象，要么是 Some，要么是 None
    //  2) 如果 map.get(key) key 存在返回 some,如果 key 不存在，则返回 None
    //  3) 如果 map.get(key).get key 存在，返回 key 对应的值,否则，抛出异常 java.util.NoSuchElementException: None.get
    println(map1.get("Zz").get)

    //方式4、getOrElse()
    //如果 key 存在，返回 key 对应的值。
    //如果 key 不存在，返回默认值。在 java 中底层有很多类似的操作。
    println(map1.getOrElse("zz","default"))

  }
}
```

#### 修改和添加

```scala
import scala.collection.mutable

object demo8 {
  def main(args: Array[String]): Unit = {

    val map1 = mutable.Map("MiChong" -> 25, "Zz" -> 24, "Alice" -> 18)

    //修改和添加
    //1) map 是可变的，才能修改，否则报错
    //2) 如果 key 存在：则修改对应的值,key 不存在,等价于添加一个 key-val
    map1("qjzxzxd")= 22
    println(map1)

    //添加多个元素
    map1 +=("qjzxzxd"->1,"demo2"->2)
    println(map1)
    
  }
}
```

#### 遍历

```scala
package cn.buildworld.scala.day3

import scala.collection.mutable

object demo8 {
  def main(args: Array[String]): Unit = {

    val map1 = mutable.Map("MiChong" -> 25, "Zz" -> 24, "Alice" -> 18)

    //遍历map
    for ((k, v) <- map1) {
      println("键：" + k + "  值：" + v)
    }
    
    for (k<-map1.keys){}
    
    for(v<-map1.values){}
  }
}
```

### 6、集--Set

> 默认情况下，Scala 使用的是不可变集合，如果你想使用可变集合，需要引用 `scala.collection.mutable.Set` 包

#### 新建set

```scala
package cn.buildworld.scala.day3

import scala.collection.mutable

object demo9 {
  def main(args: Array[String]): Unit = {

    //不可变
    val set1 = Set(1,2,3,4)
    println(set1)

    //可变
    val set2 = mutable.Set(1,2,3,4,"abc")
    println(set2)
  }
}

//结果
Set(1, 2, 3, 4)
HashSet(1, 2, 3, abc, 4)
```

#### 添加set

```scala
//可变
val set2 = mutable.Set(1,2,3,4,"abc")
println(set2)

set2.add(5)
set2 += 6
set2 +=(7)
println(set2)

//结果
HashSet(1, 2, 3, abc, 4, 5, 6, 7)
```

#### 删除

```scala 
//可变
val set2 = mutable.Set(1, 2, 3, 4, "abc")
println(set2)

set2.add(5)
set2 += 6
set2 += (7)
println(set2)

set2 -= 2
set2 -= (3)
set2.remove(4)
println(set2)

//结果
HashSet(1, abc, 5, 6, 7)
```

#### 遍历

```scala
//可变
val set2 = mutable.Set(1, 2, 3, 4, "abc")
println(set2)

for (x <- set2) {
  println(x)
}
```

#### Set更多操作

| 序号 | 方法                                     | 描述                                                 |
| :--: | ---------------------------------------- | ---------------------------------------------------- |
|  1   | def +(elem: A): Set[A]                   | 为集合添加新元素，并创建一个新的集合，除非元素已存在 |
|  2   | def -(elem: A): Set[A]                   | 移除集合中的元素，并创建一个新的集合                 |
|  3   | def contains(elem: A): Boolean           | 如果元素在集合中存在，返回 true，否则返回 false。    |
|  4   | def &(that: Set[A]): Set[A]              | 返回两个集合的交集                                   |
|  5   | def &~(that: Set[A]): Set[A]             | 返回两个集合的差集                                   |
|  6   | def ++(elems: A): Set[A]                 | 合并两个集合                                         |
|  7   | def drop(n: Int): Set[A]]                | 返回丢弃前n个元素新集合                              |
|  8   | def dropRight(n: Int): Set[A]            | 返回丢弃最后n个元素新集合                            |
|  9   | def dropWhile(p: (A) => Boolean): Set[A] | 从左向右丢弃元素，直到条件p不成立                    |
|  10  | def max: A //演示下                      | 查找最大元素                                         |
|  11  | def min: A //演示下                      | 查找最小元素                                         |
|  12  | def take(n: Int): Set[A]                 | 返回前 n 个元素                                      |

### 7、集合元素的映射-map 映射操作

```scala
def main(args: Array[String]): Unit = {

  val list = List(3,5,7)
  //map中传入一个方法，用于处理list中每个元素
  val list2 = list.map(f1)
  println(list2)
  
  
  val f2 = f1 _
  println(f2(10))

}
def f1(n:Int): Int = {
  n*2
}
```

### 8、flatmap 映射：flat 即压扁，压平，扁平化映射

> flatmap：flat 即压扁，压平，扁平化，效果就是将集合中的每个元素的子元素映射到某个函数并返回新的集合。

```scala
def main(args: Array[String]): Unit = {

  val names = List("MiChong")
  
  println(names.flatMap(upper))
}
def upper(string: String):String={
  string.toUpperCase
}
```

### 9、集合元素的过滤-filter

> filter：将符合要求的数据(筛选)放置到新的集合中

```scala
object demo3 {
  def main(args: Array[String]): Unit = {

    //只保留A开头的单词
    val list  = List("Ace","Baby","Zoom")
    println(list.filter(startA))
  }

  def startA(string: String): Boolean ={
    if(string.startsWith("A")) true else false
  }
}
```

 ### 10、reduceLeft 化简

> 化简：将 二元函数引用于集合中的函数,。

```scala
object demo4 {

  def main(args: Array[String]): Unit = {
    val list  = List(1,2,3,4,5)
    println(list.reduceLeft(sum))
  }

  //执行相加程序
  def sum(n1: Int, n2: Int): Int = {
    n1 + n2
  }
}
```

### 11、折叠--fold

> - 1) fold 函数将上 一步返回的值作为函数的第一个参数继续传递参与运算，直到 list  中的所有元素被遍历。
> - 2) 可以把 `reduceLeft`  看做简化版的 `foldLeft`

```scala
object demo5 {
  def main(args: Array[String]): Unit = {

    val list = List(1,2,3,4)
    println(list.foldLeft(5)(minus)) //从左到右
    println(list.foldRight(5)(minus))//从右到左
  }

  def minus(n1:Int,n2:Int):Int={
    n1 - n2
  }
}
```

### 12、扫描--scan

> 扫描，即对做 某个集合的所有元素做 fold  操作，但是会把产生的存 所有中间结果放置于一个集合中保存 // 斐波那契

```scala
object demo {

  def main(args: Array[String]): Unit = {

    val list = (1 to 5).scanLeft(5)(minus)
    println(list)
  }

  def minus(n1:Int,n2:Int):Int={
    n1 - n2
  }
}

//结果
Vector(5, 4, 2, -1, -5, -10)
```

## 二、模式匹配

> - 1) 如果所有 case 都不匹配，那么会执行 `case _` 分支，类似于 Java 中 default 语句
> - 2) 如果所有 case 都不匹配，又没有写 case _ 分支，那么会抛出 `MatchError`
> - 3) 每个 case 中，**不用 break 语句**，自动中断 case
> - 4) 可以在 match 中使用其它类型，而不仅仅是字符,可以是表达式
> - 5) `=>` 等价于 java swtich 的 `:`
> - 6) => 后面的代码块到下一个 case， 是作为一个整体执行，可以使用{} 扩起来，也可以不扩。

### 1、守卫

```scala
for (ch <- "+-3!"){
  ch match {
    case '+' => println(ch)
    case '-' => println(ch)
    //case _ if ... 这里不是默认匹配, 表示忽略 ch ， 而是进行后面的 if 匹配.
    case _ if ch.toString.equals('3') =>println(ch)
    case _ => println("end")
  }
}
```

### 2、匹配数组

> - 1) `Array(0)` 匹配只有一个元素且为 0 的数组。
> - 2) `Array(x,y)` 匹配数组有两个元素，并将两个元素赋值为 x 和 y。当然可以依次类推 Array(x,y,z) 匹配数组有 3 个元素的等等....
> - 3) `Array(0,_*)` 匹配数组以 0 开始

### 3、匹配列表

```scala
for (list <- Array(List(0), List(1, 0), List(88), List(0, 0, 0), List(1, 0, 0))) {
  val result = list match {
    case 0 :: Nil => "0" // 匹配的 List(0)
    case x :: y :: Nil => x + " " + y // 匹配的是有两个元素的 List(x,y)
    case 0 :: tail => "0 ..." // 匹配 以 0 开头的后面有任意元素的 List
    case x :: Nil => List(x)
    case _ => "something else"
  }
}
```

### 4、匹配元组

```scala
//请返回 (34, 89) => (89,34)
for (pair <- Array((0, 1), (34, 89), (1, 0), (1, 1), (1, 0, 2))) {
  val result = pair match { //
    case (0, _) => "0 ..." // 表示匹配 0 开头的二元组
    case (y, 0) => y //表示匹配 0 结尾的二元组
    case (x, y) => (y, x)
    case _ => "other" //.默认
  }
  println(result)
}
```

### 5、对象匹配

> - 1) 构建对象时 apply 会被调用 ，比如 val n1 = Square(5)
> - 2) 当将 Square(n) 写在 case 后时[case Square(n) => xxx]，会默认调用 unapply 方法(对象提取器)
> - 3) number 会被 传递给 def unapply(z: Double) 的 z 形参
> - 4) 如果返回的是 Some 集合，则 unapply 提取器返回的结果会返回给 n 这个形参
> - 5) case 中对象的 unapply 方法(提取器)返回 some 集合则为匹配成功
> - 6) 返回 None 集合则为匹配失败

```scala
def main(args: Array[String]): Unit = {

  def main(args: Array[String]): Unit = {
    // 模式匹配使用：
    val number: Double = 36.0 //Square(6.0)
    //说明
    //1. 当 number 去和 case Square(n) 时，会进行如下操作
    //2. 把 number 传递给 Square unapply(z: Double) 的 z
    //3. unapply 被调用，返回一个结果, 返回的结果和程序员的逻辑代码,比如 Some(math.sqrt(z))
    //4. 如果返回的结果是 Some 集合，则表示匹配成功 ,如果返回的是 None 则表示匹配失败
    //5. 如果匹配成功，就是将 Some(?) 的 值,赋给 case Square(n) 的 n
    //6. 这样就等价于将原来对象的构建参数，提取出来，我们将这个过程称为对象匹配, 这个使用很多.
    number match {
      case Square(n) => println(n) // 6.0
      case _ => println("nothing matched")
    }
  }
}

//说明
//1. unapply 为对象提取器
//2. apply 对象构建器
object Square { //静态性质
  def unapply(z: Double): Option[Double] = {
    println("unapply 被调用 z =" + z) // 36.0
    Some(math.sqrt(z)) // Some(6.0)
  }

  def apply(z: Double): Double = z * z
}
```

### 6、样例类

> - 1) 样例类仍然是类。
> - 2) 样例类用 case 关键字进行声明。
> - 3) 样例类是为模式匹配(对象)而优化的类。
> - 4) 构造器中的每一个参数都成为 val——除非它被显式地声明为 var（不建议这样做）。
> - 5) 在样例类对应的伴生对象中提供 apply 方法让你不用 new 关键字就能构造出相应的对象。
> - 6) 提供 unapply 方法让模式匹配可以工作。
> - 7) 将自动生成 toString、equals、hashCode 和 copy 方法(有点类似模板类，直接给生成，供程序员使用)。
> - 8) 除上述外，样例类和其他类完全一样。你可以添加方法和字段，扩展它们。

```scala
//抽象类
abstract class Amount

case class Dollar(value: Double) extends Amount

//Currency 样例类
case class Currency(value: Double, unit: String) extends Amount

//NoAmount 样例类
case object NoAmount extends Amount

//样例类依然可以有自己的方法和属性
case class Dog(name: String) {
  var age = 10

  def cry(): Unit = {
    println("小狗汪汪叫~~")
  }
}
```

### 7、密封类

> - 1) 如果想让 case 类的所有子类都必须在申明该类的相同的源文件中定义，可以将样例类的通用超类声明为`sealed`，这个超类称之为密封类。
> - 2) 密封就是不能在其他文件中定义/使用类。

## 三、函数式编程

### 1、偏函数

> 在对符合某个条件，而不是所有情况 进行逻辑操作时，使用偏函数是一个不错的选择将包在大括号内的一组 case 语句封装为函数，我们称之为偏函数，它只对会作用于指定类型的参数或指定范围值的参数实施计算，超出范围的值会忽略.
>
> 偏函数在 Scala 中是一个特质 `PartialFunction`

> - 1) 使用构建特质的实现类(使用的方式是 PartialFunction 的匿名子类)
> - 2) PartialFunction 是个特质(看源码)
> - 3) 构建偏函数时，参数形式 [Any, Int]是泛型，第一个表示传入参数类型，第二个表示返回参数
> - 4) 当使用偏函数时，会遍历集合的所有元素，编译器执行流程时先执行 isDefinedAt()如果为 true ,就会执行 apply,构建一个新的 Int 对象返回
> - 5) 执行 isDefinedAt() 为 false 就过滤掉这个元素，即不构建新的 Int 对象.
> - 6) map 函数不支持偏函数，因为 map 底层的机制就是所有循环遍历，无法过滤处理原来集合的元素
> - 7) collect 函数支持偏函数

```scala
object demo2 {
  def main(args: Array[String]): Unit = {

    val list = List(1,2,3,4,"abc")

    val function1 = new PartialFunction[Any,Int] {
      override def isDefinedAt(x: Any): Boolean = {
        //判断x是否是int类型
        x.isInstanceOf[Int]
      }

      override def apply(v1: Any): Int = {
        //将是int类型的元素值+1,然后返回相加之后的值
        v1.asInstanceOf[Int] + 1
      }
    }

    // 调用偏函数不能使用 map, 而是 collect
    val list2 = list.collect(function1)
    println(list2)

  }
}
```

**偏函数简化**

```scala
// 方式1
def f2: PartialFunction[Any, Int] = {
  case i: Int => i + 1
}
println(list.collect(f2))

//方式2
val list = List(1, 2, 3, 4, "abc")
val list3 = list.collect {
    case i: Int => i + 1
}
println(list3)
```

### 2、匿名函数

> 没有名字的函数就是匿名函数，可以通过 函数表达式，来设置匿名函数

```scala
val add = (a: Int, b: Int) => a + b
println(add(1,2))
```

### 3、高阶函数

> 能够接受函数作为参数的函数，叫做高阶函数 `(higher-order function)`。可使应用程序更加健壮。 高阶函数可以返回一个匿名函数。

```scala
object demo3 {
  def main(args: Array[String]): Unit = {
    println(minusxy(3)(4))
  }

  def minusxy(x: Int) = {
    (y: Int) => x - y
  }
}
```

### 4、闭包

> 基本介绍：闭包就是一个函数和与其相关的引用环境（变量/值）组合的一个整体(实体)。

**f就是一个闭包**

```scala
object demo3 {
  def main(args: Array[String]): Unit = {
    val f = minusxy(10)
    println(f(2))
  }

  def minusxy(x: Int) = {
    (y: Int) => x - y
  }
}
```

### 5、函数柯里化(curry)

> - 1) 函数编程中，接受多个参数的函数都可以转化为接受单个参数的函数，这个转化过程就叫柯里化。
> - 2) 柯里化就是证明了函数只需要一个参数而已。其实我们刚才的学习过程中，已经涉及到了柯里化操作。
> - 3) 不用设立柯里化存在的意义这样的命题。柯里化就是以函数为主体这种思想发展的必然产生的结果。(即：柯里化是面向函数思想的必然产生结果)
>   传统方式, 函数/方法(变量)， 对象.方法(变量)
>   集合.函数(函数).函数(函数).函数(函数) //函数链

```scala
def eq(s1: String)(s2: String):Boolean={
  s1.eq(s2)
}
```

###  6、控制抽象

> - 1) 参数是函数
> - 2) 函数参数没有输入值也没有返回值

```scala
def main(args: Array[String]): Unit = {

  def myRunThread(f1: =>Unit)={
    new Thread{
      override def run(): Unit = {
        f1
      }
    }
  }.start()
  
  myRunThread{
    println("start")
    Thread.sleep(2000)
    println("end")
  }
}
```

## 四、泛型

```scala
object demo2 {
  def main(args: Array[String]): Unit = {

    val value = new IntMessage[Int](20)
    println(value.get)

  }

  // 在 Scala 定义泛型用[T]， s 为泛型的引用
  abstract class Message[T](s:T){
    def get:T = s
  }

  //可以构建 Int 类型的 Message
  class IntMessage[Int](msg:Int) extends Message(msg)

  //String 类型的 Message
  class StringMessage[String](msg:String) extends Message(msg)

}
```

### 1、上界、下界

**scala 中上界**
在 scala 里表示某个类型是 A 类型的子类型，也称上界或上限，使用 <: 关键字，语法如下：

```scala
[T <:A] //A 是 T 的上界
//或用通配符:
[_ <:A]
```

**scala 中下界**
在 scala 的下界或下限，使用 >: 关键字，语法如下：

```scala
[T >: A] //A 是 T 的下界,下限
//或用通配符:
[_ >:A]
```

### 2、协变、逆变和不变

> Scala 的协变(+)，逆变(-)，协变 covariant、逆变 contravariant、不可变 invariant

- 1) C[+T]：如果 A 是 B 的子类，那么 C[A]是 C[B]的子类，称为协变。
- 2) C[-T]：如果 A 是 B 的子类，那么 C[B]是 C[A]的子类，称为逆变。
- 3) C[T]：无论 A 和 B 是什么关系，C[A]和 C[B]没有从属关系。称为不变。