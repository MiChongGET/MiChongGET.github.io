---
title: 前端模块化开发--React框架（二）：脚手架&&网络请求框架
date: 2019-05-14 14:03:16
tags: 
- React
- JavaScript
- ES6
- 脚手架
- axios
- fetch
categories:
- 前端
cover: https://file.buildworld.cn/img/91dd1f3e10ab02d27d53fce6d07396d2_react.jpg
---

[GitHub地址](https://github.com/MiChongGET/react-app)

#### 一、React脚手架
##### 1、react脚手架说明
- 1)xxx脚手架: 用来帮助程序员快速创建一个基于xxx库的模板项目
```
- a.包含了所有需要的配置
- b.指定好了所有的依赖
- c.可以直接安装/编译/运行一个简单效果
```
- 2)react提供了一个用于创建react项目的脚手架库: create-react-app
- 3)项目的整体技术架构为:  react + webpack + es6 + eslint
- 4)使用脚手架开发的项目的特点: 模块化, 组件化, 工程化

##### 2、使用命令

```shell
//设置安装全局
npm install -g create-react-app
//创建名称为hello-react的脚手架
create-react-app hello-react
//进入到项目的目录
cd hello-react
//运行项目
npm start
```
##### 3、react脚手架项目结构

```
ReactNews
	|--node_modules---第三方依赖模块文件夹
	|--public
		|-- index.html-----------------主页面
	|--scripts
		|-- build.js-------------------build打包引用配置
	|-- start.js-------------------start运行引用配置
	|--src------------源码文件夹
		|--components-----------------react组件
		|--index.js-------------------应用入口js
	|--.gitignore------git版本管制忽略的配置
	|--package.json----应用包配置文件 
	|--README.md-------应用描述说明的readme文件
```

##### 4、WebStorm配置代码模板

```javascript
import React, {Component} from 'react'

export default class $className$ extends Component {

    render() {
        return(
            <div>

            </div>
        )
    }
}
```
#### 二、react ajax
##### 1、说明
- 1)React本身只关注于界面, 并不包含发送ajax请求的代码
- 2)前端应用需要通过ajax请求与后台进行交互(json数据)
- 3)react应用中需要集成第三方ajax库(或自己封装)

##### 2、常用的ajax库
- 1)jQuery: 比较重, 如果需要另外引入不建议使用
- 2)axios: 轻量级, 建议使用

```
- a.封装XmlHttpRequest对象的ajax
- b. promise风格
- c.可以用在浏览器端和node服务器端
```

- 3)fetch: 原生函数, 但老版本浏览器不支持

```
- a.不再使用XmlHttpRequest对象提交ajax请求
- b.为了兼容低版本的浏览器, 可以引入兼容库fetch.js
```
##### 3、axios

[GitHub](https://github.com/axios/axios)
###### 安装

```shell
$ npm install axios
```

###### 使用
- GET方式
```javascript
    //使用axios发送异步的ajax请求
            const url = 'https://api.github.com/search/repositories';

            axios.get(url,{
                params:{
                    q:'r',
                    sort:'starts'
                }
            }).then(response => {
                const result = response.data
                const {owner={}} = result.items[0]
                this.setState({avatar_url: owner.avatar_url,repoName:'logo'});
            }).catch(error=>{
                console.log(error)
            })
            
    // Want to use async/await? Add the `async` keyword to your outer function/method.
            async function getUser() {
              try {
                const response = await axios.get('/user?ID=12345');
                console.log(response);
              } catch (error) {
                console.error(error);
              }
            }
            
```
- POST方式

```javascript
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```
##### 4、fetch

[GitHub](https://github.github.io/fetch/)
###### GET请求
```javascript
        fetchRequire(){
            const url = 'https://api.github12.com/search/repositories?q=r&sort=stars';
            fetch(url).then(response=>{
                return response.json()
            }).then(data=>{
                const {owner={}} = data.items[0]
                this.setState({avatar_url: owner.avatar_url,repoName:'logo'})
            }).catch(e=>{
                //请求错误的时候
                console.log(e+'==>请求错误')
            })
        }
```
###### POST请求

```javascript
        fetch(url, {
          method: "POST",
          body: JSON.stringify(data),
        }).then(function(data) {
          console.log(data)
        }).catch(function(e) {
          console.log(e)
        })
```

#### 三、重要总结
##### 1、组件间通信
###### 方式一: 通过props传递
- 1)共同的数据放在父组件上, 特有的数据放在自己组件内部(state)
- 2)通过props可以传递一般数据和函数数据, 只能一层一层传递
- 3)一般数据-->父组件传递数据给子组件-->子组件读取数据
- 4)函数数据-->子组件传递数据给父组件-->子组件调用函数

###### 方式二: 使用消息订阅(subscribe)-发布(publish)机制
- 1)工具库: PubSubJS
- 2)下载: npm install pubsub-js --save
- 3)使用: 

```javascript
import PubSub from 'pubsub-js' //引入
componentDidMount() {
  PubSub.subscribe('delete', (msg,data)=>{ }); //订阅  
}
PubSub.publish('delete', data) //发布消息
```

###### 方式三: redux

暂时不介绍



##### 2、事件监听理解
###### 原生DOM事件
- 1)绑定事件监听

```
a.事件名(类型): 只有有限的几个, 不能随便写
b.回调函数
```

- 2)触发事件

```
a.用户操作界面
b.事件名(类型)
c.数据()
```

###### 自定义事件(消息机制)
- 1)绑定事件监听

```
a.事件名(类型): 任意
b.回调函数: 通过形参接收数据, 在函数体处理事件
```

- 2)触发事件(编码)

```
a.事件名(类型): 与绑定的事件监听的事件名一致
b.数据: 会自动传递给回调函数
```

##### 3、ES6常用新语法
- 1)定义常量/变量:  const/let
- 2)解构赋值: let {a, b} = this.props   import {aa} from 'xxx'
- 3)对象的简洁表达: {a, b}
- 4)箭头函数: 

```
a.常用场景
    * 组件的自定义方法: xxx = () => {}
    * 参数匿名函数
b.优点:
    * 简洁
    * 没有自己的this,使用引用this查找的是外部this
```

- 5)扩展(三点)运算符: 拆解对象(const MyProps = {}, <Xxx {...MyProps}>)
- 6)类:  class/extends/constructor/super
- 7)ES6模块化:  export default | import