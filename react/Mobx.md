# 概述

Mobx 是一个简单的可扩展的状态管理库，没有样板代码，风格简约

Mobx6 不同于 之前的版本，不在默认支持装饰器语法，因为装饰器语法不是ES标准。但仍可以通过配置开启装饰器语法。

Mobx可以在任何支持ES语法的环境运行，因此也支持ssr。

Mobx通常与react一起使用，但是 Angular 和 Vue 中同样可以使用 Mobx，通过引入对应的绑定库即可使用。



# 核心概念

## observable

表示被 Mobx 跟踪的状态，只有被跟踪的状态改变时才会通知 React 进行重新渲染

## action

允许修改状态的方法，在严格模式下只有 action 方法才能修改状态

## computed

衍生状态标记，和 Vue 的计算属性类似。其标记的是一个 get 方法。

```js
import { makeObservable, observable, action, computed } from 'mobx'

class CounterStore {
  constructor () {
    this.count = 0
    // 标记类中的成员
    makeObservable(this, {
      count: observable, // 标记为被跟踪的状态
      decrement: action.bound, // 标记为更改状态的方法
      increment: action.bound, // 标记为更改状态的方法
      countIsNegative: computed // 标记为衍生状态
    })
  }

  decrement () {
    this.count--
  }

  increment () {
    this.count++
  }

  get countIsNegative () {
    return this.count < 0
  }
}

export default CounterStore
```

然后像使用属性一样使用衍生状态。

```jsx
import React from 'react'
import { observer } from 'mobx-react-lite'
import { useRootStore } from '../store'

function Counter () {
  const { counterStore } = useRootStore()
  return (
    <div>
      <button onClick={() => { counterStore.increment() }}>+</button>
      <span>{counterStore.count}</span>
      <button onClick={() => { counterStore.decrement() }}>-</button>
      <div>{ counterStore.countIsNegative ? '是' : '不是' }负数</div>
    </div>
  )
}

export default observer(Counter)
```

这里状态标记貌似有点多余，因为不做标记的话这里也能正常使用。

## flow

执行副作用的方法的标记，是一个 generator 函数，只有标记为 flow 的方法才能在执行完副作用之后修改状态



# 工作流程

![image-20210701144716034](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210701144716034.png)

1. 视图通过触发事件调用 action
2. 调用 action 之后回去更新状态
3. 状态改变会通知衍生状态改变
4. 最后触发视图重新渲染



# 依赖库

mobx：mobx核心库

mobx-react-lite：react 函数组件绑定库，只支持函数组件

mobx-react：react 绑定库，即支持函数组件也支持类组件



# Mobx 的使用

## 1. 定义数据类

```js
export default class CounterStore {
  constructor () {
    this.count = 0
  }

  decrement () {
    this.count--
  }

  increment () {
    this.count++
  }
}
```

## 2. 使用Mobx提供的方法对类中的属性和方法进行标记

```js
import { makeObservable, observable, action } from 'mobx'

export default class CounterStore {
  constructor () {
    this.count = 0
    // 标记类中的成员
    makeObservable(this, {
      count: observable, // 标记为被跟踪的状态
      decrement: action, // 标记为更改状态的方法
      increment: action // 标记为更改状态的方法
    })
  }

  decrement () {
    this.count--
  }

  increment () {
    this.count++
  }
}
```

## 3. 创建数据类的实例，在组件中引用实例

可以通过props，也可以直接在组件内部状态引用。如果多个组件引用了同一个实例，则一处修改，关联的组件视图都会刷新。

```jsx
import React from 'react'
import { observer } from 'mobx-react-lite'
import counterStore from '../store/CounterStore'

function Counter () {
  return (
    <div>
      <button onClick={() => { counterStore.increment() }}>+</button>
      <span>{counterStore.count}</span>
      <button onClick={() => { counterStore.decrement() }}>-</button>
    </div>
  )
}
// 使用 observer 方法包裹组件
export default observer(Counter)
```

## 4. 使用 observer 方法包裹组件

observer 方法将返回一个新的组件，在跟踪状态改变时，更新视图。

## 5. 强制绑定 action 方法的 this 指向

action 没有绑定 this 指向时，如果从实例中结构出 action 标记的方法，方法的 this 将不会指向实例。通过action.bound将 action 标记的方法的 this 指向 实例。

```js
makeObservable(this, {
  count: observable, // 标记为被跟踪的状态
  decrement: action.bound, // 标记为更改状态的方法
  increment: action.bound // 标记为更改状态的方法
})
```



# 全局共享状态

## 1. 通过React Context 实现

```js
import React, { createContext, useContext } from 'react'

import CounterStore from './CounterStore'

// 1. 创建 RootStore
class RootStore {
  constructor () {
    // 2. CounterStore 作为 RootStore 的一个成员
    this.counterStore = new CounterStore()
  }
}

// 3. 创建 RootStore 实例
const rootStore = new RootStore()

// 4. 创建 Context
const RootStoreContext = createContext()

// 5. 创建 Context 对应的 Provider 组件，并将 rootStore 作为共享状态向下传递。 Provider 组件需要作为根组件
export const RootStoreProvider = ({ children }) => (
  <RootStoreContext.Provider value={rootStore}>
    { children }
  </RootStoreContext.Provider>
)

// 7. 提供使用引入 rootStore 的方法
export const useRootStore = () => useContext(RootStoreContext)
```

将根组件替换为 RootStoreProvider 组件

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

import Counter from './components/Counter'
import Todos from './components/Todos'
import { RootStoreProvider } from './store'

ReactDOM.render(<RootStoreProvider>
  <Counter />
  <Todos />
</RootStoreProvider>,
document.getElementById('root')
)
```

在组件中通过 useRootStore 来使用状态数据。

```jsx
import React from 'react'
import { observer } from 'mobx-react-lite'
import { useRootStore } from '../store'

function Counter () {
  const { counterStore } = useRootStore()
  return (
    <div>
      <button onClick={() => { counterStore.increment() }}>+</button>
      <span>{counterStore.count}</span>
      <button onClick={() => { counterStore.decrement() }}>-</button>
    </div>
  )
}

export default observer(Counter)
```

## 2. 直接引用

不通过 Context， 直接在组件中引入 rootStore

```jsx
import CounterStore from './CounterStore'

// 1. 创建 RootStore
class RootStore {
  constructor () {
    // 2. CounterStore 作为 RootStore 的一个成员
    this.counterStore = new CounterStore()
  }
}

// 3. 创建 RootStore 实例
const rootStore = new RootStore()

export default rootStore
```

```jsx
import React from 'react'
import { observer } from 'mobx-react-lite'
import rootStore from '../store'

function Counter () {
  const { counterStore } = rootStore
  return (
    <div>
      <button onClick={() => { counterStore.increment() }}>+</button>
      <span>{counterStore.count}</span>
      <button onClick={() => { counterStore.decrement() }}>-</button>
    </div>
  )
}

export default observer(Counter)
```

# Mobx5.0的使用	

## 1.create-react-app 项目启用装饰器语法的支持

### 1. 弹射出配置，并修改

#### 1. 执行 eject 命令，弹射出配置

```bash
yarn eject
```



#### 2. 安装 babel 插件 @babel/plugin-proposal-decorators

```bash
yarn add @babel/plugin-proposal-decorators -D
```



#### 3. 在 webpack 配置中配置 babel 插件

```json
babel: [
  {
    test: /\.(js|mjs|jsx|ts|tsx)$/,
    include: paths.appSrc,
    loader: require.resolve('babel-loader'),
    plugins: [
      [
        require.resolve('@babel/plugin-proposal-decorators'),
        {
          legacy: true
        }
      ],
    ]
  }
]
```

#### 4. vscode 装饰器语法报错解决

setting 中搜索 decorator，勾选 Implicit Project Config: Experimental Decorators 项

#### 5. vscode eslint 语法检查报错解决

##### 1. class 语法提示出错

下列 class 语法不能通过 eslint 语法检查

```js
class Store {
	count = 0
}
```

原因是 eslint 默认的编译器不能识别这种写法，在 eslint 配置文件中指定编译器为 @babel/eslint-parser，并增减 babel 配置文件，将 preset 配置定义在配置文件中。

eslint 配置

```json
{
    "env": {
        "browser": true,
        "es2021": true
    },
    "parser": "@babel/eslint-parser",
}
```

babel 配置

```json
{
  "presets": [
    "react-app"
  ]
}
```

##### 2. eslint 装饰器语法检查提示错误

在 babel 中配置转化装饰器语法的插件 @babel/plugin-proposal-decorators

babel 配置

```json
{
  "presets": [
    "react-app"
  ],
  "plugins": [
    ["@babel/plugin-proposal-decorators", { "legacy": true }]
  ]
}
```

### 2. 覆盖默认的配置



## 2. Mobx5.0 绑定库版本

mobx 的版本应该和 mobx-react 的版本配套，mobx5.x 不能和 mobx-react6.x 一起使用，否则将存在版本兼容问题。

```json
"mobx": "5.15.4",
"mobx-react": "5.4.4",
```

