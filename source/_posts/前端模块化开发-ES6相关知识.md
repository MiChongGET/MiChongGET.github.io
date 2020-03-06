---
title: 前端模块化开发--ES6相关知识
date: 2019-05-08 16:00:11
tags:
- JavaScript
- ES6
categories:
- 前端
cover: https://file.buildworld.cn/img/61882d96d0f62c1a2f7fb45eb57a2828_es6.jpg

description: ES6是最新标准，目标是使JS可以编写复杂的大型应用程序，成为企业级开发语言。
---

![ES6](http://myfile.buildworld.cn/es6.jpg)
#### 一、简介

- ES全名：ECMAScript 
- ES由ECMA进行标准化的一套规范
- ES涵盖各种环境中JS使用场景，无论是浏览器环境还是类似node.js的非浏览器环境
- ES版本：1、2、3、5、6
- ES6是最新标准，目标是使JS可以编写复杂的大型应用程序，成为企业级开发语言。

#### 二、新特性
##### 1、变量声明 let
###### 使用var关键字，意味着a变量是全局的，打印结果是abc
```javascript
    function info(bol) {
        if (bol) {
            var a  = 'abc';
            console.log(a)
        }else{
            console.log(a);
        }
    }
    info(true)
```
###### 使用let关键字，此时就会报错
- ES6之前，声明变量使用var，该关键字声明的变量会在函数最顶部（不在函数内的即在全局作用域的最顶部）
- ES6声明变量使用let，常量使用const，它们声明的变量都属于块级作用域，即在声明的{}中有效

```javascript
    function info(bol) {
        if (bol) {
            let a  = 'abc';
        }else{
            console.log(a);
        }
        console.log(a)
    }
    info(true)
```
##### 2、常量声明：const
> 关键字const声明的常量只能赋值一次

##### 3、模版字符串：
> 单行字符串拼接：`${}`

```javascript
    let name = 'michong';
    console.log(`你好，欢迎${name}`);
```
##### 4、参数默认值：
> ES6为函数参数提供了默认值

```javascript
    function getAge(age = 24) {
        console.log(age);
    }
    getAge();
```
##### 5、箭头函数
> 箭头代替函数，简化函数定义，箭头函数最直观的三个特点。
- 不需要function关键字来创建函数
- 省略return关键字
- 继承当前上下文的 this 关键字

```javascript
    getName = (name) => {
        console.log(name);
    }
    getName('米虫');
```
##### 6、对象初始化简写

> ES5我们对于对象都是以键值对的形式书写，是有可能出现键值对重名的


```javascript
    People = (name, age) => {
        return {
            name,
            age
        }
    }
    console.log(People('米虫',24))
```
==返回值==

```json
{ name: '米虫', age: 24 }
```
##### 7、解构
> 数组和对象是JS中最常用也是最重要表示形式。为了简化提取信息，ES6新增了解构，这是将一个数据结构分解为更小的部分的过程

```javascript
    //对象
    People = (name, age) => {
        return {
            name,
            age
        }
    }
    let {name,age} = People('米虫',24)
    console.log(`姓名：${name}======年龄：${age}`)
    
    //数组
    let nums = [1,2];
    let [one,second] = nums;
    console.log(one+second)
```
##### 8、Spread Operator

```javascript
    //数组
    const color = ['red', 'yellow']
    const colorful = [...color, 'green', 'pink']
    console.log(colorful) //[red, yellow, green, pink]
   
    //对象
    const alp = { fist: 'a', second: 'b'}
    const alphabets = { ...alp, third: 'c' }
    console.log(alphabets) //{ "fist": "a", "second": "b", "third": "c"
```
##### 9、NodeJS对ES6支持
- 1)在项目根目录添加.babelrc文件，配置es2015插件

```json
    {
      "presets": ["es2015"]
    }
    注：
    es2015 === es6
    es2016 === es7
    es2017 === es8
```
- 2)安装es2015插件
  运行安装命令
```shell
cnpm install babel-preset-es2015 --save-dev
```
> babel-preset-es2015: 可以将es2015即es6的js代码编译为es5

- 3)全局安装命令行工具

```shell
cnpm install babel-cli -g
```
- 4)使用
  babel-node js文件名

##### 10、Import和Export（Node中不支持这个，所有参考上面第九条的内容）
> ES6中的新语法，类似于exports和require，可以实现函数跨文件使用

test.js

```javascript
    //对象
    let People = function(name, age){
        return {
            name,
            age
        }
    }
    //导出模块
    export{People}
```
test2.js

```javascript
    import {People} from './test'
    let {name,age} = People('米虫',24)
    console.log(`姓名：${name}======年龄：${age}`)
```
##### 11、Promise 对象(异步处理，Ajax等)

`Promise` 是异步编程的一种解决方案，避免了传统的回调函数的层层嵌套，也就是常说的“回调地狱”。

`Promise` 一旦新建就会立即执行，无法中途取消。


```javascript
    let promise = new Promise(function (resolve, reject) {
        // 异步操作成功
        if (true) {
            resolve('执行成功');
        } else {
            reject('执行失败');
        }
    });
    
    //可以获取上面异步操作结果得到数据，并打印出来
    promise.then(function (value) {
        //成功的
        console.log(value);
    }).catch(function (error) {
        //失败的
        console.log(error);
    });
```
##### 补充 `async` `await`
> Async/await建立于Promise之上，很多人认为的最高境界，就是根本不用关心它是不是异步。async await就是异步操作的终极解决方案。


```javascript
getJSON = () => {
    return new Promise((reslove, rejcet) => {
        setTimeout(() => {
            reslove('success')
        }, 2000);
    });
}

//使用promise
let makeRequest2 = () => {
    getJSON().then(data => {
        console.log(data);
        return 'done';
    });
    //先打印出来
    console.log('成功');
}

//使用async
let makeRequest = async () => {
    //先等待promise中的resolve()方法执行完成之后才会接着执行下面的语句
    console.log(await getJSON());
    //等待上一步执行完成之后才会执行
    console.log('成功');
    return 'done';
}
console.log(makeRequest2());
```