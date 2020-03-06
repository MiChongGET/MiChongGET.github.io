---
title: '前端模块化开发--React框架（一）: 入门和面向组件编程'
date: 2019-05-10 10:35:27
tags: 
- React
- JavaScript
- ES6
categories:
- 前端
cover: https://file.buildworld.cn/img/91dd1f3e10ab02d27d53fce6d07396d2_react.jpg
---
![React](http://myfile.buildworld.cn/react.jpg)

[React中文官网](https://react.docschina.org)

#### 一、简介
##### 1、特点
- 1)Declarative(声明式编码)
- 2)Component-Based(组件化编码)
- 3)Learn Once, Write Anywhere(支持客户端与服务器渲染)
- 4)高效
- 5)单向数据流

##### 2、React高效的原因
- 1)虚拟(virtual)DOM, 不总是直接操作DOM
- 2)DOM Diff算法, 最小化页面重绘

##### 3、相关的js核心库
- 1)react.js: React的核心库
- 2)react-dom.js: 提供操作DOM的react扩展库
- 3)babel.min.js: 解析JSX语法代码转为纯JS语法代码的库

##### 4、简单的例子

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>

    <div id="test"></div>

</body>

    <script type="text/javascript" src="../js/react.development.js"></script>
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <script type="text/babel"> //声明babel
        //创建虚拟dom元素
        const dom = <h1>Hello React</h1>;
        //将虚拟DOM渲染到真实的DOM中
        ReactDOM.render(dom, document.getElementById('test'));
    </script>
</html>
```

##### 5、虚拟DOM

```
1)React提供了一些API来创建一种 `特别` 的一般js对象
    a.var element = React.createElement('h1', {id:'myTitle'},'hello')
    b.上面创建的就是一个简单的虚拟DOM对象
2)虚拟DOM对象最终都会被React转换为真实的DOM
3)我们编码时基本只需要操作react的虚拟DOM相关数据, react会转换为真实DOM变化而更新界面
```

```javascript
<script type="text/babel"> //声明babel
        //创建虚拟dom元素
        let id = 'qjzxzxd';
     
        //三种创建dom元素的方法
        const dom = <h1>Hello React</h1>;
        const dom2 = React.createElement('h2',{id:'name'},'MiChong')
        const test3 =<h3 id={id.toUpperCase()}>{id.toUpperCase()}</h3>
        
        //将虚拟DOM渲染到真实的DOM中
        ReactDOM.render(dom, document.getElementById('test'));
        ReactDOM.render(dom2,document.getElementById('name'));
        ReactDOM.render(test3,document.getElementById('test3'));
</script>
```
#### 二、JSX（JavaScript XML）介绍和使用

##### 1、介绍
- 1)全称:  `JavaScript XML`
- 2)react定义的一种类似于XML的JS扩展语法: XML+JS
- 3)作用: 用来创建react虚拟DOM(元素)对象

```
    a.var ele = <h1>Hello JSX!</h1>
    b.注意1: 它不是字符串, 也不是HTML/XML标签
    c.注意2: 它最终产生的就是一个JS对象
```
- 4)标签名任意: HTML标签或其它标签
- 5)标签属性任意: HTML标签属性或其它
- 6)基本语法规则
```
    a.遇到 <开头的代码, 以标签的语法解析: html同名标签转换为html同名元素, 其它标签需要特别解析
    b.遇到以 { 开头的代码，以JS语法解析: 标签中的js代码必须用{ }包含
```

- 7)babel.js的作用
```
    a.浏览器不能直接解析JSX代码, 需要babel转译为纯JS的代码才能运行
    b.只要用了JSX，都要加上type="text/babel", 声明需要babel来处理
```
##### 2、使用
###### 将数据的数组转为标签的数组

```javascript
<script type="text/babel">

    //数组集合
    let names = ['java','vue','React','Angular']

    //新建DOM元素
    let ul = (<ul>
        {
            names.map((name,index)=><li key={index}>{name}</li>)
        }
    </ul>);

    //渲染DOM
    ReactDOM.render(ul,document.getElementById('names'));

</script>
```
#### 三、模块化

##### 1.模块
- 1)理解: 向外提供特定功能的js程序, 一般就是一个js文件
- 2)为什么:  js代码更多更复杂
- 3)作用: 复用js, 简化js的编写, 提高js运行效率
##### 2.组件
- 1)理解: 用来实现特定(局部)功能效果的代码集合(html/css/js)
- 2)为什么: 一个界面的功能更复杂
- 3)作用: 复用编码, 简化项目编码, 提高运行效率
##### 3.模块化
- 当应用的js都以模块来编写的, 这个应用就是一个模块化的应用

#### 四、React面向组件编程
##### 1、自定义组件(Component)

```javascript
<script type="text/babel">

    //1、定义组件
    //方式1：工厂函数组件（简单组件）
    function MyComponent() {
        return <h1>MiChong</h1>;
    }

    //方式2、ES6类组件（复杂组件）
    class MyComponent2 extends React.Component {
        render() {
            return <h2>qjzxzxd</h2>;
        }
    }

    //2、渲染组件标签
    ReactDOM.render(<MyComponent />, document.getElementById('test1'));
    ReactDOM.render(<MyComponent2 />, document.getElementById('test2'));

</script>

```
##### 2、组件三大属性
==`state`==
- 1)state是组件对象最重要的属性, 值是对象(可以包含多个数据)
- 2)组件被称为"状态机", 通过更新组件的state来更新对应的页面显示(重新渲染组件)

```javascript
<script type="text/babel">

    /**
     * 需求：自定义组件
     * 1、显示h2标题，初始文本为：你喜欢我
     * 2、点击标题更新为：我喜欢你
     */
        //1、定义组件
    class Like extends React.Component {

        constructor(props) {
            super(props);
            //初始化状态
            this.state = {
                isLikeMe: false
            };

            //将新增方法中的this强制绑定为组件对象
            this.handleClick = this.handleClick.bind(this);
        }

        //新添加的方法：内部的this默认不是组件对象
        //设置点击事件处理
        handleClick() {
            //得到状态
            const isLikeMe = !this.state.isLikeMe;
            //更新状态
            this.setState ({isLikeMe});
        }

        //重写组件类方法
        render() {
            //读取状态
            const {isLikeMe} = this.state;
            return <h2 onClick={this.handleClick}>{isLikeMe ? '你喜欢我' : '我喜欢你'}</h2>;
        }
    }

    //2、渲染组件标签
    ReactDOM.render(<Like/>, document.getElementById('test'));
</script>

```
==`props`==

- 1)每个组件对象都会有props(properties的简写)属性
- 2)组件标签的所有属性都保存在props中
- 3)通过标签属性从组件外向组件内传递变化的数据
- 4)注意: 组件内部不要修改props数据

```javascript
<script type="text/babel">
    /**
     * 需求: 自定义用来显示一个人员信息的组件
     1). 姓名必须指定
     2). 如果性别没有指定, 默认为男
     3). 如果年龄没有指定, 默认为18
     */
    //1、定义组件
    //方式一、使用工厂
    function Person(props) {
        return (
            <ul>
                <li>姓名：{props.name}</li>
                <li>年龄：{props.age}</li>
                <li>性别：{props.sex}</li>
            </ul>
        )
    }
    //方式二、使用ES6类组件
    class Person extends React.Component{
        render(){
            return (
                <ul>
                    <li>姓名：{this.props.name}</li>
                    <li>年龄：{this.props.age}</li>
                    <li>性别：{this.props.sex}</li>
                </ul>
            )
        }
    }

    //设置自定义标签的默认值
    Person.defaultProps = {
        name: '米虫',
        age: 18,
        sex: '男'
    };
    //设置自定义组件属性值的类型和必要性(要导入prop-types.js)
    Person.propTypes = {
        name: PropTypes.string.isRequired,
        age: PropTypes.number
    };

    //2、渲染组件标签
    const p1 = {
        name: 'MiChong',
        age: 18,
        sex: '男'
    };
    //ReactDOM.render(<Person name={p1.name} age={p1.age} sex={p1.sex}/>, document.getElementById('test'));
    ReactDOM.render(<Person {...p1}/>, document.getElementById('test'));
    ReactDOM.render(<Person age={p1.age} sex={p1.sex}/>, document.getElementById('test2'));

</script>

```
==`refs`==
- 1)组件内的标签都可以定义ref属性来标识自己

```
a.<input type="text" ref={input => this.msgInput = input}/>
b.回调函数在组件初始化渲染完或卸载时自动调用
```

- 2)在组件中可以通过this.msgInput来得到对应的真实DOM元素
- 3)作用: 通过ref获取组件内容特定标签对象, 进行读取其相关数据
###### 事件处理
- 1)通过onXxx属性指定组件的事件处理函数(注意大小写)

```
a.React使用的是自定义(合成)事件, 而不是使用的原生DOM事件
b.React中的事件是通过事件委托方式处理的(委托给组件最外层的元素)
```

- 2)通过event.target得到发生事件的DOM元素对象
######  例子
```javascript
<input onFocus={this.handleClick}/>
handleFocus(event) {
    event.target  //返回input对象
}
```

```javascript
<script type="text/babel">

    //1、定义组件
    class MyComponent extends React.Component {

        //固定格式
        constructor(props){
            super(props);
            //添加自定义方法
            this.showInput = this.showInput.bind(this);
            this.handleBlur = this.handleBlur.bind(this);
        }

        //自定义事件方法
        showInput(){

            let input = this.refs.content;
            //第一种写法的输出
            //alert(input.value);
            //第二种写法的输出
            alert(this.input.value);
        };
        handleBlur(event){
            alert(event.target.value);
        }


        render() {
            return (
                <div>
                    //第一种写法
                    <input type="text" ref="content"/>
                    //第二种写法
                    <input type="text" ref={input=>this.input = input}/>
                    <button onClick={this.showInput}>提示输入</button>&nbsp;&nbsp;&nbsp;
                    <input type="text" placeholder="失去焦点提示内容" onBlur={this.handleBlur}/>
                </div>
            )
        }
    }

    ReactDOM.render(<MyComponent/>, document.getElementById('test'));

</script>
```
##### 3、组件的组合
代码
```javascript
<script type="text/babel">

    //1、自定义组件
    class App extends React.Component {

        //初始化
        constructor(props) {
            super(props);
            this.state = {
                todos: ['java', 'html', 'go']
            }
            this.addTodos = this.addTodos.bind(this);
        }

        addTodos(todo) {
            const {todos} = this.state;
            todos.unshift(todo)
            //更新状态
            this.setState({todos})
        }

        render() {
            const {todos} = this.state;
            return (
                <div>
                    <h1>Sample TODO Add</h1>
                    <Add count={todos.length} addTodos={this.addTodos}/>
                    <List todos={todos}/>
                </div>
            )
        }
    }

    class Add extends React.Component {
        constructor(props) {
            super(props);
            this.add = this.add.bind(this)
        }

        //自定义方法
        add() {
            //1、读取输入的数据
            const inputValue = this.addInput.value.trim();
            //2、列表添加数据
            this.props.addTodos(inputValue)
            //3、清除输入
            this.addInput.value = ''
        }

        render() {
            return (
                <div>
                    <input type="text" ref={input => this.addInput = input}/>
                    <button onClick={this.add}>添加{this.props.count}</button>
                </div>


            )
        }
    }

    class List extends React.Component {
        render() {
            const {todos} = this.props;

            return (
                <ul>
                    {todos.map((todo, index) => <li key={index}>{todo}</li>)}
                </ul>
            )
        }
    }

    //2、渲染到真实的DOM中去
    ReactDOM.render(<App/>, document.getElementById('sample'))

</script>
```
##### 4、收集表单数据
- 1)问题: 在react应用中, 如何收集表单输入数据
- 2)包含表单的组件分类

```
a.受控组件: 表单项输入数据能自动收集成状态
b.非受控组件: 需要时才手动读取表单输入框中的数据
```
示意代码

```javascript
<script type="text/babel">

    /**
     * 需求: 自定义包含表单的组件
     1. 输入用户名密码后, 点击登陆提示输入信息
     2. 不提交表单
     */
        //1、自定义组件
    class LoginForm extends React.Component {

        constructor(props) {
            super(props);

            //初始化状态
            this.state = {
                pwd: ''
            }
            this.handleSubmit = this.handleSubmit.bind(this);
            this.handleChange = this.handleChange.bind(this);
        }

        handleSubmit(event) {

            const name = this.inputName.value;
            const {pwd} = this.state;

            //阻止时间的默认行为（提交）
            event.preventDefault()
            alert(`用户名${name},密码${pwd}`)
        }

        handleChange(event) {
            //读取密码
            const pwd = event.target.value;

            //更新状态
            this.setState({pwd})

        }

        render() {
            return (
                <form action="/test" onSubmit={this.handleSubmit}>
                    用户名：<input type="text" ref={input => this.inputName = input}/>
                    密码： <input type="password" value={this.state.pwd} onChange={this.handleChange}/>
                    <input type="submit" value="登录"/>
                </form>
            )
        }
    }

    //2、渲染到真实的DOM中去
    ReactDOM.render(<LoginForm/>, document.getElementById('sample'))

</script>
```

##### 5、组件的生命周期
![react生命周期](http://myfile.buildworld.cn/react生命周期.png)

###### 1、生命周期流程:
```
a.第一次初始化渲染显示: ReactDOM.render()
      * constructor(): 创建对象初始化state
      * componentWillMount() : 将要插入回调
      * render() : 用于插入虚拟DOM回调
      * componentDidMount() : 已经插入回调
b.每次更新state: this.setSate()
      * componentWillUpdate() : 将要更新回调
      * render() : 更新(重新渲染)
      * componentDidUpdate() : 已经更新回调
c.移除组件: ReactDOM.unmountComponentAtNode(containerDom)
      * componentWillUnmount() : 组件将要被移除回调
```
###### 2、 重要的勾子

```
1)render(): 初始化渲染或更新渲染调用
2)componentDidMount(): 开启监听, 发送ajax请求
3)componentWillUnmount(): 做一些收尾工作, 如: 清理定时器
4)componentWillReceiveProps(): 后面需要时讲
```
###### 代码
```javascript
<script type="text/babel">

    /**
     * 需求: 自定义组件
     1. 让指定的文本做显示/隐藏的渐变动画
     2. 切换持续时间为2S
     3. 点击按钮从界面中移除组件界面
     */
        //1、自定义组件
    class Life extends React.Component {

        constructor(props) {
            super(props);
            this.state = {
                opacity: 1
            };
            this.destroyComponent = this.destroyComponent.bind(this);
        }

        destroyComponent(){
            ReactDOM.unmountComponentAtNode(document.getElementById('sample'))
        }

        //设置循环定时器
        componentDidMount() {
            this.intervalId = setInterval(function () {
                let {opacity} = this.state
                opacity -= 0.1
                if (opacity <= 0) {
                    opacity = 1
                }
                // 更新状态
                this.setState({opacity})
                console.log(opacity)
            }.bind(this), 200)
        }

        //组件将要被移除回调
        componentWillUnmount(){
            clearInterval(this.intervalId);
        }

        render() {

            const {opacity} = this.state;
            return (
                <div>
                    <h1 style={{opacity: opacity}}>{this.props.content}</h1>
                    <button onClick={this.destroyComponent}>change</button>
                </div>
            )
        }

    }

    //2、渲染到真实的DOM中去
    ReactDOM.render(<Life content="准时下班"/>, document.getElementById('sample'))

</script>
```
##### 6、虚拟DOM与DOM Diff算法
![image](http://myfile.buildworld.cn/diff算法示意图.png)

```javascript
<script type="text/babel">
  /*
  验证:
  虚拟DOM+DOM Diff算法: 最小化页面重绘
  */

  class HelloWorld extends React.Component {
    constructor(props) {
      super(props)
      this.state = {
          date: new Date()
      }
    }

    componentDidMount () {
      setInterval(() => {
        this.setState({
            date: new Date()
        })
      }, 1000)
    }

    render () {
      console.log('render()')
      return (
        <p>
          Hello, <input type="text" placeholder="Your name here"/>!&nbsp;
          <span>It is {this.state.date.toTimeString()}</span>
        </p>
      )
    }
  }

  ReactDOM.render(
    <HelloWorld/>,
    document.getElementById('example')
  )
</script>
```