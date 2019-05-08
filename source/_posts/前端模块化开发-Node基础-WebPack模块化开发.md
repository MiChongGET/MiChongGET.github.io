---
title: 前端模块化开发--Node基础&&WebPack模块化开发
date: 2019-05-07 17:19:32
tags: 
- Node
categories:
- 前端
---
![Node](http://myfile.buildworld.cn/node.jpg)
#### 一、Node 开发

##### 1、模块化开发
###### 定义统一的方法：function.js

```javascript
exports.sum = function sum(a, b) {
    return a + b;
}
```
###### 导入方法：use.js

```javascript
var fun = require('./function')
console.log(fun.sum(1, 2))
```
##### 2、服务器

```javascript
//创建服务器
var http = require('http');
http.createServer(function (request, response) {
    // 发送 HTTP 头部
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, { 'Content-Type': 'text/plain' });
    //页面显示内容
    response.write('hi michong\n') //可以调用多次
    response.end('hello world!')    //只可以调用一次
}).listen(8888);
```
##### 3、接收参数

```javascript
//接收参数(http://localhost:8888/?name=michong)
var http = require('http');
var url = require('url')
http.createServer(function (request, response) {
    // 发送 HTTP 头部
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, { 'Content-Type': 'text/plain' });
    //解析url参数
    var params = url.parse(request.url,true).query;
    response.end('姓名：'+params.name);
}).listen(8888);
```
#### 二、 包资源管理器NPM

> npm全称Node Package Manager，他是node包管理和分发工具。其实我们可以把NPM
> 理解为前端的Maven

###### npm的镜像替换成淘宝
```shell
npm config set registry http://registry.npm.taobao.org/
```
###### 全局下载

```shell
npm install xxx -g
```
###### 运行工程


```
如果我们想运行某个工程，则使用run命令
如果package.json中定义的脚本如下
dev是开发阶段测试运行
build是构建编译工程
lint 是运行js代码检测
```
#### 三、Webpack
> Webpack 是一个前端资源加载/打包工具。它将根据模块的依赖关系进行静态分
> 析，然后将这些模块按照指定的规则生成对应的静态资源。

![webpack示意图](http://myfile.buildworld.cn/webpack.png)
###### 全局安装

```shell
npm install webpack ‐g
npm install webpack‐cli ‐g
```
##### 快速入门
##### 1、js打包
- 创建src文件夹，创建bar.js

```javascript
exports.info = function (str) {
    document.write(str);
}
```
- src下创建logic.js

```javascript
exports.add = function (a, b) {
    return a + b;
}
```
- src下创建main.js

```javascript
var bar = require('./bar');
var logic = require('./logic');
bar.info('hello world!' + logic.add(1, 2));
```
- 创建配置文件webpack.config.js ，该文件与src处于同级目录

```javascript
var path = require('path');
module.exports = {
    entry: './src/main.js',
    output:{
        path: path.resolve(__dirname,'./dist'),
        filename: 'bundle.js'
    }
};
```
> 以上代码的意思是：读取当前目录下src文件夹中的main.js（入口文件）内容，把对应的
> js文件打包，打包后的文件放入当前目录的dist文件夹下，打包后的js文件名为bundle.js

- 执行编译命令

```
webpack
```
- 创建index.html ,引用bundle.js

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <script src="dist/bundle.js"></script>
</body>
</html>
```

##### 2、css打包
> Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用
> loader 进行转换。
> Loader 可以理解为是模块和资源的转换器，它本身是一个函数，接受源文件作为参数，
> 返回转换的结果。这样，我们就可以通过 require 来加载任何类型的模块或文件，比如
> CoffeeScript、 JSX、 LESS 或图片。首先我们需要安装相关Loader插件，css-loader 是
> 将 css 装载到 javascript；style-loader 是让 javascript 认识css

```
npm install style-loader css-loader --save-dev
```
```注：--save和--save-dev区别
npm install --save moduleName 命令
1.安装模块到项目node_modules目录下。
2.会将模块依赖写入dependencies 节点。
3.运行 npm install 初始化项目时，会将模块下载到项目目录下。
4.运行npm install --production或者注明NODE_ENV变量值为production时，会自动下载模块到node_modules目录中。
npm install --save-dev moduleName 命令
1.安装模块到项目node_modules目录下。
2.会将模块依赖写入devDependencies 节点。
3.运行 npm install 初始化项目时，会将模块下载到项目目录下。
4.运行npm install --production或者注明NODE_ENV变量值为production时，不会自动下载模块到node_modules目录中。
```
- 修改webpack.config.js

```javascript
var path = require('path');
module.exports = {
    entry: './src/main.js',
    output:{
        path: path.resolve(__dirname,'./dist'),
        filename: 'bundle.js'
    },
    module:{
		rules:[
			{
				test:/\.css$/,
				use:['style-loader','css-loader']
			}
		]
	}
};
```
- 在src文件夹创建css文件夹,css文件夹下创建css1

```css
body{
    background: blue;
}
```
- 修改main.js ，引入css1.css

```javascript
require('./css/css1.css')
```
- 重新webpack一下

#### 其它
##### vscode Chrome-debug插件
> 在launch.json中添加

```json
    , {
            "name": "使用本机 Chrome 调试",
            "type": "chrome",
            "request": "launch",
             "file": "${workspaceRoot}/index.html",
        //  "url": "http://mysite.com/index.html", //使用外部服务器时,请注释掉 file, 改用 url, 并将 useBuildInServer 设置为 false "http://mysite.com/index.html
            "runtimeExecutable": "C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe", // 改成您的 Chrome 安装路径
            "sourceMaps": true,
            "webRoot": "${workspaceRoot}",
        //  "preLaunchTask":"build",
            "userDataDir":"${tmpdir}",
            "port":5433
        }
```