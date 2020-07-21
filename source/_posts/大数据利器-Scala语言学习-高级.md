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