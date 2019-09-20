---
title: Python开发之Pandas的使用
date: 2019-09-20 23:38:41
tags:
- Python
- Pandas
categories:
- 人工智能
- 编程语言
---

#### 一、简介
- Pandas 是 Python 中的数据操纵和分析软件包，它是基于Numpy去开发的，所以Pandas的数据处理速度也很快，而且Numpy中的有些函数在Pandas中也能使用，方法也类似。

- Pandas 为 Python 带来了两个新的数据结构，即 Pandas Series(可类比于表格中的某一列)和 Pandas DataFrame(可类比于表格)。借助这两个数据结构，我们能够轻松直观地处理带标签数据和关系数据。


#### 二、创建Pandas Series
> 可以使用 pd.Series(data, index) 命令创建 Pandas Series，其中data表示输入数据， index 为对应数据的索引，除此之外，我们还可以添加参数dtype来设置该列的数据类型。


```Python
import pandas as pd #约定俗成的简称
pd.Series(data = [30, 6, 7, 5], index = ['eggs', 'apples', 'milk', 'bread'],dtype=float)

out:
eggs      30.0
apples     6.0
milk       7.0
bread      5.0
dtype: float64
```
> data除了可以输入列表之外，还可以输入字典，或者是直接一个标量。


```Python
pd.Series(data={'name':'michong','age':18})

out:
name    michong
age          18
dtype: object
```
#### 三、访问和删除Series中的元素

##### 1、访问
> 一种类似于从列表中按照索引访问数据，一种类似于从字典中按照key来访问value。


```Python
s = pd.Series(data=8,index=['apple','milk','bread'])

s[0]
out:
    8

s['apple']
out:
    8
    
s.loc['apple']
s.iloc[1]
```
##### 2、修改
>修改完记得重新赋值即可

##### 3、删除

```Python
s.drop(['apple'])
out:
    milk     8
    bread    8
    dtype: int64
```
> .drop()函数并不会修改原来的数据，如果你想要修改原数据的话，可以选择添加参数inplace = True或者是用原数据替换s = s.drop(label)

```Python
s.drop(['apple'],inplace=True)
```

#### 四、DataFrame的使用
##### 1、创建DataFrame

> pd.DataFrame(data, index, columns)


```Python
data是数据，可以输入ndarray，或者是字典（字典中可以包含Series或arrays或），或者是DataFrame；

index是索引，输入列表，如果没有设置该参数，会默认以0开始往下计数；

columns是列名，输入列表，如果没有设置该参数，会默认以0开始往右计数；
```

```
d = [[1,2],[3,4]]
df = pd.DataFrame(data=d,index=['a','b'],columns=['one','two'])
df

out:
    	one	two
    a	1	2
    b	3	4
```
##### 2、访问DataFrame中的元素

- 访问单行
```Python
df.loc['a']
df.iloc[0]

out:
    one    1
    two    2
    Name: a, dtype: int64
    
```
- 访问多行

```Python
df.loc[['a','b']]
df.iloc[[0,1]]

out:
    	one	two
    a	1	2
    b	3	4
```

- 访问某一列

```Python
df.one
df['one']
df.iloc[:,0]

out：
    a    1
    b    3
    Name: one, dtype: int64
```
- 访问多列

```Python
df[['one','two']]
df.iloc[:,0:2] #0-2,不包含2，即第三列

out:
    	one	two
    a	1	2
    b	3	4
```
- 访问某一个元素

```Python
df.iloc[0,1]    #先访问行再访问列
df['two']['a']  #先访问列再访问行

out:
    2
```
##### 3、删除、增加元素
> 使用.drop函数删除元素，默认为删除行，添加参数axis = 1来删除列。
- 删除行

```Python
df.drop(['a'])
    out:
        one	two
    b	3	4
```
- 删除列

```Python
df.drop('one',axis=1)

out:
    	two
    a	2
    b	4
```
==值得注意的是，drop函数不会修改原数据，如果想直接对原数据进行修改的话，可以选择添加参数inplace = True或用原变量名重新赋值替换。==

- 增加元素
>一种是append()，另外一种是insert()


```Python
df.insert(2,'T',8) #新生成一个列，列名称是T

out:
    	one	two	T
    a	1	2	8
    b	3	4	8
    

df.insert(2,'F',[9,10]) #设定F列下的每一行的值
out：
        one	two	F	T
    a	1	2	9	8
    b	3	4	10	8
```

```
data2 = pd.DataFrame([[8,9,10,11],[6,7,8,9]],
                     columns=['one','two','F','T']
                    ,index=['c','d'])
df.append(data2,ignore_index=True)

out:
    	one	two	F	T
    0	1	2	9	8
    1	3	4	10	8
    2	8	9	10	11
    3	6	7	8	9
```
##### 4、重命名
- 修改列的名称
```Python
df.rename(columns={'one':'第一列'})
out:
        第一列	two	F	T
    a	1   	2	9	8
    b	3   	4	10	8
```
- 修改行的名称

```Python
df.rename(index={'a':'第一行'})
out:
            	one	two	   F   T
    第一行	 1	  2	   9 	8
    b	    	3	 4	   10	8
```
##### 5、更改索引

```
可以使用函数set_index(index_label)，将数据集的index设置为index_label。

除此之外，还可以使用函数reset_index()重置数据集的index为0开始计数的数列。
```
##### 6、缺失值(NaN)处理
- 查找NaN

> 可以使用isnull()和notnull()函数来查看数据集中是否存在缺失数据，在该函数后面添加sum()函数来对缺失数量进行统计。除此之外，还可以使用count()函数对非NaN数据进行统计计数。
- 删除NaN -- df.dropna()
> dropna()函数还有一个参数是how，当how = all时，只会删除全部数据都为NaN的列或行。

==不修改原来的数据==


- 替换NaN

```Python
df.fillna(0)
out:
        0	   1 	  F 	T	 one    two
a	0.0	 0.0	9.0	 8.0	1.0	 2.0
b	0.0	 0.0    10.0    8.0	 3.0    4.0
0	5.0	 6.0	0.0	 0.0	0.0	0.0
```

```
使用fillna()函数可以替换NaN为某一值。其参数如下：
    value：用来替换NaN的值
    
    method：常用有两种，一种是ffill前向填充，一种是backfill后向填充
    
    axis：0为行，1为列
    
    inplace：是否替换原数据，默认为False
    
    limit：接受int类型的输入，可以限定替换前多少个NaN
```
#### 五、数据分析流程及Pandas应用
##### 1、打开文件

```Python
    #打开csv文件
    pd.read_csv('filename')
    #打开excel文件
    pd.read_excel('filename')
    #处理中文字符的tsv文件
    pd.read_csv('filename',sep = '\t',encoding = 'utf-8')
```
##### 2、查看数据

```Python
    #查看前五行
    df.head()
    #查看尾五行
    df.tail()
    #查看随机一行
    df.sample()
```
##### 3、查看数据信息

```Python
    #查看数据集行数和列数
    df.shape
    #查看数据集信息（列名、数据类型、每列的数据量——可以看出数据缺失情况）
    df.info()
    #查看数据集基本统计信息
    df.describe()
    #查看数据集列名
    df.columns
    #查看数据集数据缺失情况 
    df.isnull().sum()
    #查看缺失列数据
    df[df['col_name'].isnull()]
    #查看数据集数据重复情况
    sum(df.duplicated())
    #查看重复数据
    df[df.duplicated()]
    #查看某列分类统计情况
    df['col_name'].value_counts()
    #查看某列唯一值
    df['col_name'].unique()
    #查看某列唯一值数量
    df['col_name'].nunique()
    #以某列对数据集进行排序
    df.sort_values(by = 'col_name',ascending = False)#False为由大至小
```
##### 4、数据筛选

```Python
    #提取某行
    df.iloc[row_index]
    df.loc['row_name']
    #提取某几行
    df.iloc[row_index_1:row_index_2]
    #提取某列
    df['col_name']
    #提取某几列
    df[['col_name_1','col_name_2']]
    #提取某行某列的值
    df.iloc[row_index,col_index]
    df.loc['row_name','col_name']
    #筛选某列中满足某条件的数据
    df[df['col_name'] == value]#等于某值的数据，同理满足所有比较运算符
    df.query('col_name == value')#代码效果同上
    df[(df['col_name_1'] >= value_1) & (df['col_name_2'] != value_2)]#与&，或|
    df.query('(col_name_1 >= value_lower) & (col_name_2 <= value_upper)')
    df.groupby('col_name').groups #按col_name列进行分组，聚类
```
##### 5、数据清理

```Python
    #删除某行
    df.drop(['row_name'],inplace = True)#若添加inplace = True，修改后的数据会覆盖原始数据
    #删除某列
    df.drop(['col_name'],axis = 1)
    #缺失值的处理
    df.fillna(mean_value)#替换缺失值
    df.dropna()#删除包含缺失值的行
    df.dropna(axis = 1, how = 'all')#只删除所有数据缺失的列
    #删除重复值
    drop_duplicates(inplace = True)
    #更改某行/列/位置数据
    用iloc或者loc直接替换修改即可
    #更改数据类型
    df['datetime_col'] = pd.to_datetime(df['datetime_col'])
    df['col_name'].astype(str)#还可以是int/float...
    #更改列名
    df.rename(columns={'A':'a', 'C':'c'}, inplace = True)
    #apply函数
    #讲function应用在col_name列，此方法比用for循环快得多得多
    df['col_name'].apply(function)
```