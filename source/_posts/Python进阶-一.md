---
title: Python进阶(一)
date: 2020-03-14 10:58:27
tags:
- Python
categories:
- 编程语言
---
#### 1、*args 和 **kwargs
##### *args
>  *args 是⽤来发送⼀个⾮键值对的可变数量的参数列表给⼀个函数.

```Python
    def test_var_args(f_arg, *argv):
        print("first normal arg:", f_arg)
        for arg in argv:
            print("another arg through *argv:", arg)
    test_var_args('yasoob', 'python', 'eggs', 'test')
    
    这会产⽣如下输出:
    first normal arg: yasoob
    another arg through *argv: python
    another arg through *argv: eggs
    another arg through *argv: test
```
##### **kwargs
>  ****kwargs 允许你将不定长度的键值对, 作为参数传递给⼀个函数。 如果你想要在⼀个函数⾥处理带名字的参数, 你应该使⽤** **kwargs。比如参数是字典


```
def greet_me(**kwargs):
    for key,value in kwargs.items():
        print ("{0} == {1}".format(key,value))

greet_me(name='michong',age=10)        

输出
name == michong
age == 10
```
> 标准参数与*args、**kwargs在使⽤时的顺序

```
demo_func(fargs, *args, **kwargs)
```
#### 2、调试(Debugging)
##### pdb.set_trace()方法
> 这个方法在jupter Notebook中也可以使用，这个方法使用的时候直接放在需要打断点的地方


```Python
import pdb
def make_bread():
    pdb.set_trace()
    return "I don't have time"
print(make_bread())
```
##### debugger模式下的命令
- c: 继续执⾏
- w: 显⽰当前正在执⾏的代码⾏的上下⽂信息
- a: 打印当前函数的参数列表
- s: 执⾏当前代码⾏，并停在第⼀个能停的地⽅（相当于单步进⼊）
- n: 继续执⾏到当前函数的下⼀⾏，或者当前⾏直接返回（单步跳过）


#### 3、Generator 生成器
##### 可迭代对象(Iterable)
> 对象中定义了可以返回一个迭代器的__iter__方法，或者定义了可以⽀持下标索引的__getitem__⽅法，它就是一个可迭代对象

##### 迭代器(Iterator)
> 任意定义了next或者__next__发放，它就是一个迭代器

##### 迭代(Iteration)
> 循环遍历的过程叫迭代

##### ⽣成器(Generators)
> 它也是一中迭代器，使用yield生成一个值

```
def generator_function():
    for i in range(10):
        yield i
for item in generator_function():
    print(item)
    
输出：
    # Output: 0
    # 1
    # 2
    # 3
    # 4
    # 5
    # 6
    # 7
    # 8
    # 9
```
#### 4、Map，Filter 和 Reduce
##### Map
###### 遍历元素

```
items = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, items))
```

###### 遍历方法

```
def multiply(x):
    return (x*x)
def add(x):
    return (x+x)
funcs = [multiply, add]
for i in range(5):
    value = map(lambda x: x(i), funcs)
    print(list(value))
# 译者注：上⾯print时，加了list转换，是为了python2/3的兼容性
# 在python2中map直接返回列表，但在python3中返回迭代器
# 因此为了兼容python3, 需要list转换⼀下

# Output:
# [0, 0]
# [1, 2]
# [4, 4]
# [9, 6]
# [16, 8]
```
##### Filter
>filter过滤列表中的元素，并且返回⼀个由所有符合要求的元素所构成的列
表，符合要求即函数映射到该元素时返回值为True. 

```
number_list = range(-5,5)
list(filter(lambda x:x%2==0 , number_list))

output
    [-4, -2, 0, 2, 4]
```
##### Reduce
> 对一个列表进行计算并返回结果，可以使用Reduce函数

```
#下面执行的是列表里面所有的元素相互相加的功能
from functools import reduce
reduce((lambda x, y:x+y),[1,2,3,4])

output：
    10
```
#### 5、set数据结构

##### 集合

```
some_list = ['a', 'b', 'c', 'b', 'd', 'm', 'n', 'n']
duplicates = set([x for x in some_list if some_list.count(x) > 1])
print(duplicates)

##输出: set(['b', 'n'])
```
##### 交集

```
valid = set(['yellow','red','blue','green','black'])
input_set = set(['red','brown'])
print(input_set.intersection(valid))

output:
{'red'}
```
##### 差集

```
valid = set(['yellow','red','blue','green','black'])
input_set = set(['red','brown'])
print(input_set.difference(valid))


output：
{'brown'}
```

##### 基本操作
###### 添加元素

```
s.add( x )

s.update( x ) ##也可以添加元素，且参数可以是列表
```
###### 移除元素

```
s.remove( x ) ##将元素 x 从集合 s 中移除，如果元素不存在，则会发生错误。
s.discard( x ) ##此外还有一个方法也是移除集合中的元素，且如果元素不存在，不会发生错误。


s.discard( x ) ##此外还有一个方法也是移除集合中的元素，且如果元素不存在，不会发生错误。

s.pop()  ##随机删除集合中的一个元素
```
###### 计算集合元素个数

```
len(s)
```

###### 清空集合

```
s.clear()
```

###### 集合内置方法完整列表
![集合内置方法完整列表](http://myfile.buildworld.cn/360截图18430709535842.png)

