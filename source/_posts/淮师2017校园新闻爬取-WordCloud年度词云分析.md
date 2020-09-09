---
title: 淮师2017校园新闻爬取&&WordCloud年度词云分析
date: 2017-12-07 22:08:38
tags:
- Python
- WordCloud
categories:
- 校园
description: 前言：最近一直想做数据采集这块，想到年底了，来个年终总结什么的。所以就想到了爬取学校2017年的校内新闻。
top_img: https://ae01.alicdn.com/kf/He5c0d011cf8d49fe8056a63e72b32afeC.jpg
cover: https://file.buildworld.cn/img/w882gr.jpg
---
>前言：最近一直想做数据采集这块，想到年底了，来个年终总结什么的。所以就想到了爬取学校2017年的校内新闻。基于采集的五百多篇新闻结合Python的WordCloud做出来个词云，可视化新闻图片，根据关键词出现次数自动设置大小。效果图如下:

![](https://file.buildworld.cn/img/20171207202848015)

## 一、爬虫模块：
>爬虫模块使用了Java的开源爬虫框架jsoup。通过对页面的批量获取以及对网页进行分析从而得到新闻内容。
因为学校的网站过于简单，没有使用现在流行的json接口，也没有严格的反爬虫验证，所以爬取新闻没什么技术难度，那就不需要去找接口了，比网易那个简单多了，有兴趣可以看看我那篇博客 网易云音乐API获取分析。

![](https://ae01.alicdn.com/kf/H6c12bdfea3544428a196f5ede7a26c80Q.jpg)
![](https://ae01.alicdn.com/kf/Hcf71889edc1e48398f73c37b599dc699g.jpg)

>从上面的图片可以看出，新闻列表是写在table中的，每一个标题就是对应一个链接，也就是新闻正文部分。所以我们第一步就是要先通过新闻列表获取新闻详情。
首先批量获取新闻的URL。使用get方式请求链接获取网页内容，返回来是一个完整的网页内容。我们该如何通过从一个复杂的网页获取我们想要的内容呢？引出jsoup框架，一代神器！使用jsoup框架的时候你感觉你在使用正则表达式，但是比正则容易多了。jsoup的官网：http://www.open-open.com/jsoup/。

### 1、获取新闻链接
**先贴上代码看看：**
```java
public static void getArticleList(){
		
		for (int i = 1; i <38; i++) {
		
			String list = HttpUtils.doGet("http://www.hnnu.edu.cn/s/21/t/148/p/11/i/"+i+"/list.htm");
			Document doc = Jsoup.parse(list);
			Elements div_content = doc.select("a[href][target][style]");
			for (Element element : div_content) {
				String href = element.attr("href");
				getArticle(href);
			}
		}
	}
```
- 1）分析链接，通过查阅可以看出2017年的新闻总共占了37页，通过for循环就可以获取2017年的列表页。
- 2）通过工具类获取网页内容。
- 3）先获取document对象，再输入指定的过滤规则就可以得到a标签，但是我们需要获得a标签里面的href属性。
- 4）使用Element的attr方法获得标签里面的属性

### 2、爬取新闻内容
![](https://ae01.alicdn.com/kf/H11060439a90c4f2787c1c4b9a6e9e6ffX.jpg)

```java
public static void getArticle(String url ){
		//先获取新闻列表
		count++;
		System.out.println("第"+count+"篇文章");
		String getUrl = "http://www.hnnu.edu.cn"+url;
		String list = HttpUtils.doGet(getUrl);
		Document doc = Jsoup.parse(list);
		Elements div_content = doc.select("span[style]");
		//将爬取的新闻转化为集合
		List<String> eachText = div_content.eachText();
		StringBuffer content = new StringBuffer();
		for (int i = 0; i < eachText.size(); i++) {
			//去除作者等信息
			if(i>2){
				content.append(eachText.get(i));
			}
		}
		//System.out.println(content.toString());
		//以Mybatis方式插入数据库
		ArticleService as = new ArticleService();
		Article article = new Article(content.toString(), getUrl);
		System.out.println(article);
		
		as.insert(article);
		
	}
```
- 1）先请求获取新闻详情网页
- 2）发现新闻的内容都是写在了span标签里面，通过指定获取span标签的内容
- 3）再使用span[style]近一步过滤内容
- 4）为了后面的数据分析的准确度，所以此处去除作者之类的内容
- 5）使用MyBatis框架将数据写到数据库中。如下图：

## 二、词云模块：
**词云模块使用了Python语言。**
- 1、首先安装WorlCloud模块。当然不是一次就能安装成功的，遇到了很多坑！放弃Python2使用了Python3，无奈，Python2安装插件安装了一晚上加一上午，还没搞定！果断换了Python3。抛出链接，自己总结了一下http://blog.csdn.net/qq_31673689/article/details/78745155。
- 2、使用Python的PyMysql框架读取数据库，关于pymysql的安装和使用请看我的另一篇博客：http://blog.csdn.net/qq_31673689/article/details/78745207
- 3、将新闻内容取出写入到本地的一个TXT文件中，贴出代码：

```python
# -*- coding: utf-8 -*-
import pymysql.cursors
#数据库连接设置
connection = pymysql.connect(host='127.0.0.1',
                             port=3306,
                             user='root',
                             password='root',
                             db='mybatisdemo',
                             charset='utf8mb4',
                             cursorclass=pymysql.cursors.DictCursor)
cursor = connection.cursor()
sql = "select * from article"
#执行sql查询
cursor.execute(sql)
#查询所有，得到结果集
result = cursor.fetchall()
#循环遍历获取结果
 
#以追加的方式打开文件,设置为utf-8
f = open('D:\\PythonStudio\\WORK\\Demo1\\test.txt','a',encoding='utf-8')
 
for data in result:
    id = data["id"]
    #获取新闻内容
    content = data["content"]
    url = data["url"]
    f.write(content)
    print(id,content,url)
 
connection.close()
f.close()
```
- 4、使用WordCloud模块对TXT文件读取自动分析，并自动生成结果图片
```python
from wordcloud import WordCloud,STOPWORDS,ImageColorGenerator
import matplotlib.pyplot as plt
import numpy as np
from PIL import Image
from os import path
from scipy.misc import imread
 
d = path.dirname(__file__)
#两种读取背景图片方式，注意路径不能使用\
pic = np.array(Image.open('D:/PythonStudio/WORK/Demo1/Ciyun/github4.png'))
bg_pic = imread(path.join(d,'github.png'))
 
#读取收集文章的TXT文件需要使用utf-8
f = open(u'D:/PythonStudio/WORK/Demo1/test.txt','r',encoding='utf-8').read()
# 你可以通过font_path参数来设置字体集
# width,height,margin可以设置图片属性
wordcloud = WordCloud(font_path = r'C:\\Windows\\Fonts\\STFANGSO.ttf',
                      background_color="white",
                      max_words=1000,
                      max_font_size=100,
                      random_state=50,
                      scale=5,
                      mask=pic).generate(f)
#显示结果
plt.imshow(wordcloud)
plt.axis("off")
plt.show()
#保存到本地图片中
wordcloud.to_file('test.png')
```
![](https://ae01.alicdn.com/kf/H6c21b61389d848f8a10d10d52ccdc5ddZ.jpg)

![](https://ae01.alicdn.com/kf/H5ee1d6be6ddc4631a6077c275ba2f93bV.jpg)

>总结：本次小项目使用到了Java和Python两种语言，（其实Python也适合爬虫，但是现在Java比较顺手，所以将就了就使用了Java）新闻爬取模块没什么难点，就是细心一点分析一下网页就行了。Python的模块就比较坑了！各种不兼容，插件安装不上去！最后还是靠着耐心解决了