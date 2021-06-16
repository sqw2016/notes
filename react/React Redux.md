# Redux

## Redux概念

redux是一种状态管理容器，提供可预测的状态管理。

核心概念：

view: 视图

store: 保存数据的容器，一个应用只有一个

state: store中保存的数据

action: 更新数据唯一方法

reducer: 重新计算state的过程



## Redux的工作流

![img](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)

Redux 数据流是单向的，用户视图 Components 通过 dispatch action 来触发 reducers 更新 store 中的 state。store 接收到 action 之后将 action传递给 reducer ，进行数据更新操作。然后通过 store.subscribe 订阅 store 更新，触发视图更新。



## Redux 核心API

**`Redux.createStore(reducer[, initState])`**: 创建store对象，接收两个参数：

reducer: 更新state的函数，初始化时和dispatch时都会调用该函数，返回新的state。

initState: 可选参数，用于设置初始化的 state。

```js
// 1 创建 store
const store = Redux.createStore(reducer)
// const store = Redux.createStore(reducer, {count: 0})
```

 设置 state 初始值的两种方法

1. 通过 createState 方法第二个参数设置

   ```js
   const store = Redux.createStore(reducer, {count: 0})
   ```

   

2. 通过给 reducer 函数设置默认参数来设置 state 初始值

   ```js
   const reducer = function(state = initialState, action) { return state }
   ```



**`store.dispatch(action)`**: 接收一个 action 参数，action 是一个描述 state 变化的对象，包含一个必需的 type 属性用于标识状态的变化，以及其他自定义属性。

```js
const increment = { type: 'increment' }
const decrement = { type: 'decrement' }
store.dispatch(increment)
store.dispatch(decrement)
```

对于 action 对象，社区有一个规范，将其携带的信息统一放到 payload 属性中。如果发生错误，额外添加一个 error 属性来表示错误，payload 则存放错误对象。

```js
const action = {
  type: 'ADD_TODO',
  payload: {
    text: 'Do something.'  
  }
}
const errAction = {
	type: 'ADD_TODO',
  payload: new Error(),
  error: true
}
```



**`store.getState()`**: 获取最新的 state 数据。

**`store.subscribe(callback)`**: 接收一个回到函数，当state发生改变时，将会调用callback。

```js
// 7 订阅state数据更新
store.subscribe(() => {
	// 更新视图
	document.getElementById('count').innerText = store.getState().count
})
```



**`reducer(state[, action])`**: state更新过程，返回的值将作为最新的 state。初始化时调用将会传入一个特殊type的action。

![image-20210616111605061](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210616111605061.png)

```js
const reducer = function(state = initialState, action) {
  console.log(state, action);
  switch(action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    default:
      return state
  }
}
```



## Redux实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div>
    <button id="plus">+</button>
    <span id="count">0</span>
    <button id="minus">-</button>
  </div>
  <script src="./redux.min.js"></script>
  <script>
    // 3 初始化 state
    const initialState = {
      count: 0
    }

    // 2 创建 reducer
    const reducer = function(state = initialState, action) {
      console.log(state, action);
      switch(action.type) {
        case 'increment':
          return { count: state.count + 1 }
        case 'decrement':
          return { count: state.count - 1 }
        default:
          return state
      }
    }

    // 4 定义 action
    const increment = { type: 'increment' }
    const decrement = { type: 'decrement' }

    // 6 定义点击事件
    document.getElementById('plus').onclick = function() {
      // 触发更新
      store.dispatch(increment)
    }

    document.getElementById('minus').onclick = function() {
      // 触发更新
      store.dispatch(decrement)
    }

    // 1 创建 store
    const store = Redux.createStore(reducer)
    // const store = Redux.createStore(reducer, {count: 0})

    // 7 订阅state数据更新
    store.subscribe(() => {
      document.getElementById('count').innerText = store.getState().count
    })

  </script>
</body>
</html>
```



## bindActionCreators的使用

bindActionCreators(actionCreators, dispatch): 接收actionCreator对象，返回一个action方法对象。用于分离action生成函数。

```react
// 使用 bindActionCreators生成 action
const mapDispatchToProps = dispatch => bindActionCreators({
  increment() {
    return { type: 'increment' }
  },
  decrement() {
    return { type: 'decrement' }
  }
}, dispatch)

// 未使用 bindActionCreators生成
const mapDispatchToProps = dispatch => ({
  increment: () => {
    dispatch({ type: 'increment' })
  },
  decrement: () => {
    dispatch({ type: 'decrement' })
  }
})
```

进一步将提取 actionCreators 到单独文件：

```js
// src/store/actions/counter.js
export const increment = () => ({ type: 'increment'})
export const decrement = () => ({ type: 'decrement'})
```

```react
import * as counterActions from '../store/actions/counter';

// 使用 bindActionCreators生成 action
const mapDispatchToProps = dispatch => bindActionCreators(counterActions, dispatch);

// 未使用 bindActionCreators生成
const mapDispatchToProps = dispatch => ({
  increment: () => {
    dispatch({ type: 'increment' })
  },
  decrement: () => {
    dispatch({ type: 'decrement' })
  }
})
```



## combineReducers方法的使用

combinReducers 方法主要用于合并拆分之后的 reducer。该方法接收一个对象，对象中的属性键值分别对应 state 中该 reducer 的属性名，以及子 reducer。

```react
// 合并 Reducer
import { combineReducers } from 'redux';
import counterReducer from './counter';
import modalReducer from './modal';

// 合并之后的 state
// { counter: { count: 0}, modal: { show: false }}
export default combineReducers({
  counter: counterReducer,
  modal: modalReducer
});
```

```react
// counterReducer
import { DERCREMENT, INCREMENT } from '../const/counter'

const initialState = {
  count: 0
}

export default function counterReducer(state = initialState, action) {
  let obj = {};
  switch (action.type) {
    case INCREMENT:
      obj = { count: state.count + action.payload };
      break;
    case DERCREMENT:
      obj = { count: state.count - action.payload };
      break;
    default:
  }
  return {
    ...state,
    ...obj
  }
}
```

```react
// modalReducer
import { HIDEMODAL, SHOWMODAL } from '../const/modal'

const initialState = {
  show: false
}

export default function modalReducer(state = initialState, action) {
  let obj = {}
  switch(action.type) {
    case HIDEMODAL: {
      obj = { show: false }
      break;
    }
    case SHOWMODAL: {
      obj = { show: true }
      break;
    }
    default:
  }
  return {
    ...state,
    ...obj
  }
}
```



# React中不使用Redux时的问题

组件有时需要修改祖先组件中传下来的数据时，只能通过祖先组件将数据更新的方法传递下来，组件通过调用该方法才能修改祖先组件中的数据。类似于这种跨层级数据共享的问题，在没有使用redux时，只能通过这种一层层传递数据修改函数来实现。



# 在React中使用Redux

## **安装依赖**

```bash
npm i redux react-redux
```

## **react-redux**

顾名思义react-redux是帮助在react中集成redux的。react-redux将提供一个Provider组件和一个connect方法帮助在react中使用redux。

### Provider组件

必须是最外层组件，必须传入一个store属性，该属性就是所有组件共享的state状态容器。

```react
ReactDOM.render(
  <Provider store={store}>
    <Counter />
  </Provider>,
  document.getElementById('root')
);
```

### connect方法

connect方法是一个柯里化的方法，依次接收一个mapStateToProps方法和视图组件。主要有以下功能：

1. 向props中添加一个dispatch方法，用于触发store中的state更新。
2. state改变时自动重新渲染组件。
3. 向props中映射 state 状态。

mapStateToProps 方法接收一个表示store中所有数据的state，并返回一个对象，该对象中的所有属性将被添加到props中。

```react
import { connect } from 'react-redux';

function Counter(props) {
  return (
    <div>
      <button onClick={() => { props.dispatch({ type: 'increment' }) }}>+</button>
      <span>{props.count}</span>
      <button onClick={() => { props.dispatch({ type: 'decrement' }) }}>-</button>
    </div>
  )
}

const mapStateToProps = state => ({
  count: state.count
})

export default connect(mapStateToProps)(Counter)
```

### connect第二个参数

通过向connect传递第二个参数将state更新方法预定义，并映射到 props 中，以简化组件中state更新方法的调用。

```react
import { connect } from 'react-redux';

function Counter({count, increment, decrement}) {
  return (
    <div>
      <button onClick={increment}>+</button>
      <span>{count}</span>
      <button onClick={decrement}>-</button>
    </div>
  )
}

const mapStateToProps = state => ({
  count: state.count
})

const mapDispatchToProps = dispatch => ({
  increment: () => {
    dispatch({ type: 'increment' })
  },
  decrement: () => {
    dispatch({ type: 'decrement' })
  }
})

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

# Redux中间件

redux中间件实质是一个函数，用于增强或扩展redux功能。

## 加入中间件之后Redux的工作流

![image-20210616174556121](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210616174556121.png)

store 接收到 action 之后会将 action 传递给中间件，经过中间件处理之后再传递给 reducer。

