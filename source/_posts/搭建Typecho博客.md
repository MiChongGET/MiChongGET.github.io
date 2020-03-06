---
title: 搭建Typecho博客
date: 2020-03-04 17:07:11
tags:
- Typecho
- 博客开发 
top_img: https://ae01.alicdn.com/kf/H3aa6d2d529db4a42894510d605c8dc69a.png
cover: https://ae01.alicdn.com/kf/H3aa6d2d529db4a42894510d605c8dc69a.png
description: 本条博客记录围绕typecho建站的各种资料，希望帮到有需要的人。💪
top: True
categories:
- 后端
---

### 一、介绍
[typecho官网][1]
[开发文档][2]

>其实对于大多数人来说，我们不必关系typecho网站的开发，主要是项目的部署和主题的更换。

> 对于我来说，我大学的时候就开始写博客了，开始的CSDN,wordpress到后来的hexo,再到gridea，因为手头有服务器和域名，所以就干脆整一个typecho了  ，WordPress主要太大了，使用起来感觉很臃肿，就抛弃了。


### 二、使用
#### 1、部署
>如果手头有闲置的服务器，环境也没有配置的话，我建议使用phpstudy环境，一键搭建Nginx+Php+MySQL环境

[phpstudy官网地址][3]
>Centos一键安装，注意系统要求没有安装过docker环境  
```shell
yum install -y wget && wget -O install.sh https://download.xp.cn/install.sh && sh install.sh
```
`简直傻瓜式有木有`

![](https://ae01.alicdn.com/kf/H563e8c7e6111451f95264075fe157751K.png#vwid=1915&vhei=959)

#### 2、主题美化
>接下来就是主题的修改，推荐下面的网址，不仅有各种主题，还有好用的插件

[typecho主题网站](https://typecho.me/ "typecho主题网站")

>本文的主题来自https://blog.imalan.cn/archives/247/ ，强烈推荐 👍   github地址：https://github.com/AlanDecode/Typecho-Theme-VOID 

>另外推荐几个网站

[https://qqdie.com/](https://qqdie.com/)

#### 3、常用插件
| 名称  | 描述  |  项目地址 |
| ------------ | ------------ | ------------ |
|  APlayer for Typecho(Meting) |  	在 Typecho 中使用 APlayer 播放在线音乐吧～ |https://github.com/MoePlayer/APlayer-Typecho   |
|  DoubanBoard | 在博客上展示你的豆瓣书单与豆瓣影单  | 详细介绍 https://blog.imalan.cn/archives/168/  |
|  EditorMD | 增强Markdown编写  | 详细介绍 https://blog.imalan.cn/archives/168/  |
|  Qiniu File | 将 Typecho 的附件上传至七牛云存储中。  | https://lichaoxi.com/  |
|  DPlayer | 将 Typecho 的附件上传至七牛云存储中。  | https://plugins.typecho.me/plugins/qiniu-file.html  |
|  KaTeX4Typecho | 数学公式展示 | https://github.com/vc12345679/KaTeX4Typecho |
|  YoduPlayer | 一款清爽的BGM播放器,需要您的主题支持pjax或者instantclick才能保证页面切换依旧播放 |https://qqdie.com/archives/typecho-yoduplayer.html |

>更多插件请看下面网址

https://qqdie.com/plugins/
https://plugins.typecho.me/

### 三、常见的问题
#### 1、网站开启SSL
- 首先在网站根目录找到`config.inc.php`，添加代码

```php
/** 开启HTTPS */
define('__TYPECHO_SECURE__',true);
```
- 其次在phpstudy站点设置中开启SSL,SSL证书需要自己去申请，但是phpstudy中有免费的SSL，还可以自动设置脚本在SSL过期前自动申请SSL

![](https://ae01.alicdn.com/kf/Hcd540c702407448986313199dd5e859dq.png#vwid=825&vhei=702)


#### 2、typecho如果使用MySQL，默认不支持emoji的
- 首先将typecho使用的数据格式设置成 `utf8mb4_unicode_ci`
- 然后进入该数据执行下面的sql语句
```sql
alter table typecho_comments convert to character set utf8mb4 collate utf8mb4_general_ci;
alter table typecho_contents convert to character set utf8mb4 collate utf8mb4_general_ci;
alter table typecho_fields convert to character set utf8mb4 collate utf8mb4_general_ci;
alter table typecho_metas convert to character set utf8mb4 collate utf8mb4_general_ci;
alter table typecho_options convert to character set utf8mb4 collate utf8mb4_general_ci;
alter table typecho_relationships convert to character set utf8mb4 collate utf8mb4_general_ci;
alter table typecho_users convert to character set utf8mb4 collate utf8mb4_general_ci;
```
- 网站根目录数据库配置文件`config.inc.php`,修改一下

```php
/** 定义数据库参数 */
$db = new Typecho_Db('Pdo_Mysql', 'typecho_');
$db->addServer(array (
  ...
  'charset' => 'utf8mb4',  // 修改编码为 utf8mb4
  ...
), Typecho_Db::READ | Typecho_Db::WRITE);
Typecho_Db::set($db);
```
- OK👌

  [1]: http://typecho.org/
  [2]: http://lab.qqdie.com/docs/#/
  [3]: https://www.xp.cn/linux.html