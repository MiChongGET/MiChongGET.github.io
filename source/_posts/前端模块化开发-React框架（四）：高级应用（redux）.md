---
title: 前端模块化开发--React框架（四）：高级应用（redux）
date: 2019-05-17 13:27:23
tags: 
- React
- JavaScript
- ES6
- redux
categories:
- 前端
cover: https://file.buildworld.cn/img/91dd1f3e10ab02d27d53fce6d07396d2_react.jpg
---

[代码地址](https://github.com/MiChongGET/react-redux)

###### 安装

```
npm install --save redux
```

#### 一、 redux要点
	1. redux理解
	2. redux相关API
	3. redux核心概念(3个)
	4. redux工作流程
	5. 使用redux及相关库编码

##### 1. redux理解
	什么?: redux是专门做状态管理的独立第3方库, 不是react插件
	作用?: 对应用中状态进行集中式的管理(写/读)
	开发: 与react-redux, redux-thunk等插件配合使用

#### 二、redux相关API
> 	redux中包含: createStore(), applyMiddleware(), combineReducers()

> 	store对象: getState(), dispatch(), subscribe()
> 	react-redux: <Provider>, connect()()

##### 1、 createStore()
- 1) 作用: 
> 创建包含指定reducer的store对象
- 2) 编码:

```javascript
import {createStore} from 'redux'
import counter from './reducers/counter'
const store = createStore(counter)
```
##### 2、store对象

- 1)作用: 
  redux库最核心的管理对象
- 2)它内部维护着:
   `state`
   	`reducer`
- 3)核心方法:

```javascript
        getState()
		dispatch(action)
		subscribe(listener)
```

- 4)编码:

```javascript
        store.getState()
		store.dispatch({type:'INCREMENT', number})
		store.subscribe(render)
```
##### 3、applyMiddleware()

- 1)作用:
  `应用上基于redux的中间件(插件库)`
- 2)编码:

```javascript
import {createStore, applyMiddleware} from 'redux'
import thunk from 'redux-thunk'  // redux异步中间件
const store = createStore(
  counter,
  applyMiddleware(thunk) // 应用上异步中间件
)
```
##### 4、combineReducers()
- 1)作用:
  `合并多个reducer函数`
- 2)编码:

```javascript
export default combineReducers({
  user,
  chatUser,
  chat
})
```

#### 三、redux核心概念(3个)

	action: 
		默认是对象(同步action), {type: 'xxx', data: value}, 需要通过对应的actionCreator产生, 
		它的值也可以是函数(异步action), 需要引入redux-thunk才可以
	reducer
		根据老的state和指定的action, 返回一个新的state
		不能修改老的state
	store
		redux最核心的管理对象
		内部管理着: state和reducer
		提供方法: getState(), dispatch(action), subscribe(listener)
##### 1、action: 		
- 1)标识要执行行为的对象
- 2)包含2个方面的属性

```
    a.type: 标识属性, 值为字符串, 唯一, 必要属性
    b.xxx: 数据属性, 值类型任意, 可选属性
```

- 3)例子:

```javascript
        const action = {
			type: 'INCREMENT',
			data: 2
		}
```

- 4)Action Creator(创建Action的工厂函数)
```javascript
const increment = (number) => ({type: 'INCREMENT', data: number})
```

##### 2、reducer
- 1)根据老的state和action, 产生新的state的纯函数
- 2)样例

```javascript
    export default function counter(state = 0, action) {
		  switch (action.type) {
		    case 'INCREMENT':
		      return state + action.data
		    case 'DECREMENT':
		      return state - action.data
		    default:
		      return state
		  }
		}
```

- 3)注意

```
a.返回一个新的状态
b.不要修改原来的状态
```

##### 4、store
- 1)将state,action与reducer联系在一起的对象
- 2)如何得到此对象?
  	
```javascript
        import {createStore} from 'redux'
		import reducer from './reducers'
		const store = createStore(reducer)
```

- 3)此对象的功能?
  	
```javascript
        getState(): 得到state
		dispatch(action): 分发action, 触发reducer调用, 产生新的state
		subscribe(listener): 注册监听, 当产生了新的state时, 自动调用
```



#### 四、redux工作流程
![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)
![](D:\File\GitFile\MiChongGET.github.io\source\_posts\2R5G8bG.png)
		

#### 五、 使用redux及相关库编码
	需要引入的库: 
		redux
		react-redux
		redux-thunk
		redux-devtools-extension(这个只在开发时需要)
	redux文件夹: 
		action-types.js
		actions.js
		reducers.js
		store.js
	组件分2类: 
		ui组件(components): 不使用redux相关PAI
		容器组件(containers): 使用redux相关API

##### 1、 react-redux
###### 下载依赖包

```
npm install --save react-redux
```

###### 理解
- 1)一个react插件库
- 2)专门用来简化react应用中使用redux

###### React-Redux将所有组件分成两大类

- 1)UI组件

```
a.只负责 UI 的呈现，不带有任何业务逻辑
b.通过props接收数据(一般数据和函数)
c.不使用任何 Redux 的 API
d.一般保存在components文件夹下
```

- 2)容器组件

```
a.负责管理数据和业务逻辑，不负责UI的呈现
b.使用 Redux 的 API
c.一般保存在containers文件夹下
```

###### 相关API
- 1)Provider
    让所有组件都可以得到state数据

```javascript
<Provider store={store}>
<App />
</Provider>
```

- 2)connect()
    用于包装 UI 组件生成容器组件

```javascript
import { connect } from 'react-redux'
      connect(
        mapStateToprops,
        mapDispatchToProps
      )(Counter)
```
- 3)mapStateToprops()
    将外部的数据（即state对象）转换为UI组件的标签属性
```javascript
const mapStateToprops = function (state) {
   return {
     value: state
   }
  }
```

- 4)mapDispatchToProps()

        将分发action的函数转换为UI组件的标签属性
        简洁语法可以直接指定为actions对象或包含多个action方法的对象


##### 2、redux异步编程
###### 下载redux插件(异步中间件)

```shell
npm install --save redux-thunk
```