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

**`Redux.createStore(reducer,[initState],[enhancer])`**: 创建store对象，接收两个参数：

reducer: 更新state的函数，初始化时和dispatch时都会调用该函数，返回新的state。

initState: any, 可选参数，用于设置初始化的 state。

enhancer: function, 增强函数，使用 applyMiddleware 注册的中间件

```js
// 1 创建 store
const store = Redux.createStore(reducer)
// const store = Redux.createStore(reducer, {count: 0})
// const store = createStore(reducer, applyMiddleware(loggerMiddleware, testMiddleware))
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

## Redux中间件本质

Redux 中间件的本质是一个函数，模板如下：

```js
const reduxMiddleware = store => next => action => { 
  //中间件逻辑 
}
```

store 是 redux store 的简化对象，包含 2 个方法，getState 获取 store state 以及 dispatch 触发数据更新。

![image-20210618165442750](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210618165442750.png)

next 是一个方法，只有调用 next 之后才会将 action 向后传递给其他的中间件或 Reducer。

action 就是调用 dispatch 传入的 action。



## 中间件的使用

首先通过 redux 提供的 applyMiddleware 方法注册中间件，然后将 applyMiddleware 的返回，作为第二个或第三个参数传递给 createStore 方法。

```js
const store = createStore(reducer, applyMiddleware(loggerMiddleware, testMiddleware))
```

使用中间件之后，store 会将接收到的 action 传递给中间件，在中间件中通过调用 next 将 action 传递给其他中间件或 reducer。



## 中间件的执行顺序

如果有多个中间件，中间件会按照注册时的顺序依次执行。



## redux-thunk 中间件处理异步

redux-thunk 中间件认为 action 如果是一个方法则表示要进行异步处理，调用 action 方法进行异步处理，然后通过 dispatch 来触发同步的 action 更新 store state。

```js
// thunkMiddleware
export default ({ dispatch, getState }) => next => action => {
  if (typeof action === 'function') { // action 如果是一个方法，则认为是一个异步 action
    return action(dispatch, getState)
  }
  return next(action)
}
```



```js
// 同步 action
export const increment = payload => ({ type: INCREMENT, payload })
// 异步 action
export const incrementSync = payload => dispatch => {
  setTimeout(() => {
    dispatch(increment(payload))
  }, 2000)
}

```

## redux-saga

redux-saga 和 redux-thunk 一样允许在redux工作流中加入异步处理，不同的是，redux-saga 允许将异步处理抽离出来单独管理。

### redux-saga的基本使用

安装saga

```bash
yarn add redux-saga
```

编写saga函数

```js
import { takeEvery, put, delay } from '@redux-saga/core/effects'
import { increment } from '../actions/counter'
import { INCREMENT_SAGA } from '../const/counter'

function * syncIncrement () {
  yield delay(2000)
  yield put(increment(10))
}

export const counterIncrementSaga = function * () {
  // 接收 action
  yield takeEvery(INCREMENT_SAGA, syncIncrement)
}
```

这里通过 takeEvery 来声明 action 对应的实际逻辑处理。最终通过 put 来触发一个同步的 action 更新 state。takeEvery 允许同时触发多个 INCREMENT_SAGA action，会依次执行。如果想要同一时间只允许执行一个 action，可以使用 takeLatest。

注册 redux-saga 中间件， 并启动 counterIncrementSaga。

```js
// app.js中
import { createStore, applyMiddleware } from 'redux'
import reducer from './reducers'
import createSagaMiddleware from '@redux-saga/core'
import { counterIncrementSaga } from './saga/counter'

const sagaMiddleware = createSagaMiddleware()

// 注册 redux-saga
const store = createStore(reducer, applyMiddleware(sagaMiddleware))

export default store

// 启动 counterIncrementSaga
sagaMiddleware.run(counterIncrementSaga)
```

在 action 生成器中定义 INCREMENT_SAGA  action。

```js
import { DERCREMENT, INCREMENT, INCREMENT_SAGA } from '../const/counter'

export const increment = payload => ({ type: INCREMENT, payload })
export const decrement = payload => ({ type: DERCREMENT, payload })

// 触发 saga 的 action
export const incrementSaga = () => ({ type: INCREMENT_SAGA })

```

最后在视图中触发 incrementSaga 。

```jsx
import React from 'react'
import { connect } from 'react-redux'
import { bindActionCreators } from 'redux'
import * as counterActions from '../store/actions/counter'

function Counter ({ count, incrementSync, incrementSaga, decrement }) {
  return (
    <div>
      <!-- 触发saga -->
      <button onClick={() => { incrementSaga(3) }}>+</button>
      <span>{count}</span>
      <button onClick={() => { decrement(5) }}>-</button>
    </div>
  )
}

const mapStateToProps = state => ({
  count: state.counter.count
})

const mapDispatchToProps = dispatch => bindActionCreators(counterActions, dispatch)

export default connect(mapStateToProps, mapDispatchToProps)(Counter)

```

### redux-saga 传参

首先 redux-saga 的参数是通过视图中触发的 action 传入的。参数会记录到 action 中并向下传递。

```js
// 触发 saga 的 action
export const incrementSaga = () => ({ type: INCREMENT_SAGA })
```

之后 saga 接收到 action 然后去触发 action 对应的处理函数，并将 action 传递给处理函数 syncIncrement。syncIncrement  接收到 action，拿到 action 中的参数，最终将参数传递给同步 action，更新 state。

```js
function * syncIncrement (action) {
  yield delay(2000)
  // 传递参数给同步 action
  yield put(increment(action.payload))
}
```

### saga 文件的拆分与合并

按模块将 saga 文件拆分，假如现在有两个模块，counter 和 modal。将这两个模块的 saga 拆分放到单独的文件中管理。

```js
// modal.saga.js
import { takeEvery, put, delay } from 'redux-saga/effects'
import { show } from '../actions/modal'
import { SHOWMODAL_SAGA } from '../const/modal'

function * asyncShow (action) {
  yield delay(2000)
  yield put(show())
}

export const modalShowSaga = function * () {
  yield takeEvery(SHOWMODAL_SAGA, asyncShow)
}
```

```js
// counter.saga.js
import { takeEvery, put, delay } from 'redux-saga/effects'
import { increment } from '../actions/counter'
import { INCREMENT_SAGA } from '../const/counter'

function * asyncIncrement (action) {
  yield delay(2000)
  yield put(increment(action.payload))
}

export const counterIncrementSaga = function * () {
  yield takeEvery(INCREMENT_SAGA, asyncIncrement)
}
```

然后在 index.js 中组合 saga，并统一对外导出。

```js
// index.js
import { all } from 'redux-saga/effects'
import { counterIncrementSaga } from './counter'
import { modalShowSaga } from './modal'

export default function * rootSaga () {
  yield all([
    counterIncrementSaga(),
    modalShowSaga()
  ])
}
```

all 方法用来组合 saga，接收的是 effects 数组，所以需要调用子 saga 模块导出的方法来得到结果。

## redux-actions

redux-actions的引入时为了简化 action 和 reducer 。因为 action 和 reducer 的创建包含大量样板代码。

安装

```bash
yarn add redux-actions
```

以 counter 模块为例，修改 action 和 reducer。

```js
// counter.actions.js
import { createAction } from 'redux-actions'
import { DECREMENT, INCREMENT, INCREMENT_SAGA } from '../const/counter'

export const increment = createAction(INCREMENT)
export const decrement = createAction(DECREMENT)

export const incrementSaga = createAction(INCREMENT_SAGA)
```

```js
// counter.reducer.js
import { handleActions } from 'redux-actions'
import { decrement, increment } from '../actions/counter'

const initialState = {
  count: 0
}

export default handleActions({
  [increment]: (state, action) => ({ count: state.count + action.payload }),
  [decrement]: (state, action) => ({ count: state.count - action.payload })
}, initialState)
```

在 saga 中也可以直接使用 createAction 返回的结果作为 takeEvery 方法的第一个参数。

```js
import { takeEvery, put, delay } from 'redux-saga/effects'
import { increment, incrementSaga } from '../actions/counter'

function * asyncIncrement (action) {
  yield delay(2000)
  yield put(increment(action.payload))
}

export const counterIncrementSaga = function * () {
	// 使用 createAction 的结果作为第一个参数
  yield takeEvery(incrementSaga, asyncIncrement)
}
```

# redux 的实现

redux 的基础功能都主要是能够创建 store ，即 createStore 函数的实现。

## createStore 函数实现



createStore[reducer, preloadState, enhancer]:{getState, dispatch, subscribe}

createStore接收三个参数：

reducer：state 更新函数，reducer必须是函数，否则提示错误

```js
const reducer = function(state, action) {
  switch(action.type) {
    case 'increment': 
      return state + 1
    case 'decrement':
      return state - 1
    default:
      return state
  }
}
```

preloadState：初始化的state

enhancer：增强函数，增强主要是增强 dispatch 方法。接收一个 createState 参数并返回一个新的函数，新函数接收reducer, preloadState两个参数，返回一个增强 store 对象。

```js
const enhancer = function(createStore) {
  return function(reducer, preloadState) {
    // 创建基础的 store
    const store = createStore(reducer, preloadState)
    const dispatch = store.dispatch
    function _dispatch(action) {
      if (typeof action === 'function') {
        return action(dispatch)
      }
      return dispatch(action)
    }

    return {
      ...store,
      dispatch: _dispatch
    }
  }
}
```

然后返回一个包含三个方法的store对象：

getState：返回当前的 state

dispatch：接收一个 action，触发 state 更新。action 必须是一个包含 type 属性的对象。

subscribe：接收一个回调函数，监听 state 更新，state 更新之后会依次调用回调函数。

```js
/**
 * redux 核心：
 * createStore(reducer, preloadState, enhancer): { getState, dispatch, subscribe }
 * reducer: state更新函数
 * preloadState: 初始化的状态
 * enhancer: 增强函数
 */
function createStore(reducer, preloadState, enhancer) {
  if (isNotFunction(reducer)) throw new Error('reducer 必须是一个函数')
  const listeners = []
  let currentState = preloadState
  // 判断 enhancer 是否存在
  if (enhancer) {
    if (typeof enhancer !== 'function') throw new Error('enhancer 必须是一个函数')
    return enhancer(createStore)(reducer, preloadState)
  }
  // 获取当前状态
  function getState() {
    return currentState
  }
  // 触发状态更新
  function dispatch(action) {
    if (!isPlainObject(action)) throw new Error('action必须是一个对象')
    if(typeof action.type === 'undefined') throw new Error('action必须具有type属性')
    // 进行状态更新
    currentState = reducer(currentState, action)
    // 通知订阅
    for (let index = 0, len = listeners.length; index < len; index++) {
      const listener = listeners[index];
      listener()
    }
  }
  function subscribe(fn) {
    if (typeof fn !== 'function') throw new Error('subscribe只能接受函数作为参数')
    listeners.push(fn)
  }
  return {
    getState,
    dispatch,
    subscribe
  }
}

function isNotFunction(arg) {
  return typeof arg !== 'function'
}

function isPlainObject(obj) {
  if (typeof obj !== 'object' || obj === null) return false
  let proto = obj
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto)
  }
  return proto === Object.getPrototypeOf(obj)
}
```

createStore 的使用：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Redux</title>
</head>
<body>
  <div>
    <button id="plus">+</button>
    <span id="count">0</span>
    <button id="minus">-</button>
  </div>
  <script src="myRedux.js"></script>
  <script>
    const enhancer = function(createStore) {
      return function(reducer, preloadState) {
        // 创建基础的 store
        const store = createStore(reducer, preloadState)
        const dispatch = store.dispatch
        function _dispatch(action) {
          if (typeof action === 'function') {
            return action(dispatch)
          }
          return dispatch(action)
        }

        return {
          ...store,
          dispatch: _dispatch
        }
      }
    }
    const reducer = function(state, action) {
      switch(action.type) {
        case 'increment': 
          return state + 1
        case 'decrement':
          return state - 1
        default:
          return state
      }
    }
    const store = createStore(reducer, 0, enhancer)
    console.log(store);
    store.subscribe(() => {
      document.getElementById('count').innerHTML = store.getState()
    })
    document.getElementById('plus').onclick = () => {
      store.dispatch(dispatch => {
        setTimeout(() => {
          dispatch({type: 'increment'})
        }, 2000)
      })
    }
    document.getElementById('minus').onclick = () => {
      store.dispatch({type: 'decrement'})
    }

  </script>
</body>
</html>
```

## applyMiddleware函数的实现

applyMiddleware实际是一个增强函数，所以必须有增强函数的基本形式。

```js
function applyMiddleware(...middlewares) {
  return function(createStore) {
    return function(reducer, preloadState) {
      const store = createStore(reducer, preloadState)
      // applyMiddleware 的具体实现
      return {
        ...store,
        dispatch: dispatch
      }
    }
  }
}
```

 action会依次经过中间件的处理，然后通过 next 向下传递。先来看一个中间件的示例

```js
const logger = function (store) { // 第一层函数
  return function (next) {  // 第二层函数
    return function (action) { // 第三层函数
      console.log('logger')
      // 向下传递action，即调用下一个中间件的方法，并传入action
      next(action)
    }
  }
}
```

首先调用中间件的第一层函数，将简化的store传入。 并将外层函数的执行结果，即第二层函数收集到一个数组中。最后通过数组的reduceRight方法将第二层函数的执行结果，也就是第三层函数串联到一起。

```js
function applyMiddleware(...middlewares) {
  return function (createStore) {
    return function (reducer, preloadState) {
      const store = createStore(reducer, preloadState)
      // 传递给中间件函数第一层函数的参数 store，只包含 getState 和 dispatch 方法
      const postStore = {
        getState: store.getState,
        dispatch: store.dispatch
      }
      // 执行第一层函数，并传入简化版的 store
      const chain = middlewares.map(middleware => middleware(postStore))
      // 第二层函数的 next 参数是下一0个中间件的最里层函数，所以从最后一个中间件开始执行
      // 最后一个中间件执行时传入的 next 参数是 dispatch 函数
      const dispatch = chain.reduceRight((r, c) => c(r), store.dispatch)
      return {
        ...store,
        dispatch: dispatch
      }
    }
  }
}
```

## bindActionCreators 实现

bindActionCreators(actionCreators, dispatch) 方法接收两个参数：

actionCreators：生成 action 的函数

```js
const actions = bindActionCreator({
  increment() {
    return { type: 'increment'}
  },
  decrement() {
    return { type: 'decrement'}
  },
}, store.dispatch)
```

dispatch: 派发 action 的 dispatch 函数

最后返回一个相同属性的对象，对象属性对应的值为一个内部执行 dispatch(action) 的函数。

```js
{
  increment: () => {
    dispatch({ type: 'increment' })
  },
  decrement: () => {
    dispatch({ type: 'decrement' })
  }
}
```

```js
function bindActionCreators(actionCreators, dispatch) {
  const actions = {}
  // 遍历 actionCreators，为每个key生成一个新的函数，函数内部执行对应的 action
  Object.keys(actionCreators).forEach(key => {
    actions[key] = function () {
      dispatch(actionCreators[key]())
    }
  })
  return actions
}
```

具体使用：

```js
const actions = bindActionCreator({
  increment() {
    return { type: 'increment'}
  },
  decrement() {
    return { type: 'decrement'}
  },
}, store.dispatch)
document.getElementById('plus').onclick = () => {
  // store.dispatch({type: 'increment'})
  actions.increment()
  // store.dispatch(dispatch => {
  //   setTimeout(() => {
  //     dispatch({type: 'increment'})
  //   }, 2000)
  // })
}
document.getElementById('minus').onclick = () => {
  actions.decrement()
}
```

## combineReducers 实现

combineReducers(reducers) 接收一个参数：

reducers：一个包含多个子 reducer 的对象

返回一个 reducer 函数更新state，当函数执行时，逐个调用子 reducer 获取新的 state，最后将新的 state 合并返回。

```js
function combineReducers(reducers) {
  // reducers中的每个属性必须是函数
  for (const key in reducers) {
    const reducer = reducers[key]
    if (typeof reducer !== 'function') throw new Error('reducer 必须是函数')
  }
  // 返回一个新的 reducer 方法，方法执行时，通过调用 reducers 中的 reducer 来获取新的 state，最后合并新的 state 并返回。
  return function (state, action) {
    const nextState = {}
    // 遍历 reducers 获取新的 state
    for (const key in reducers) {
      const reducer = reducers[key]
      // 调用 reducer 获取新的 state
      nextState[key] = reducer(state[key], action)
    }
    return nextState
  }
}
```

# Redux Toolkit

redux 的工具集，是对 redux 的二次封装，为了是 redux 开发变得更加简单。

安装依赖

```bash
yarn add @reduxjs/toolkit react-redux
```



## 创建状态切片

状态切片可以看做是 action 和 reducer 的集合， 不用在去手动创建 action 和 reducer 函数。

createSlice(options): 创建切片，接收一个参数对象 options, 结构如下：

```js
function createSlice({
    // A name, used in action types
    name: string,
    // The initial state for the reducer
    initialState: any,
    // An object of "case reducers". Key names will be used to generate actions.
    reducers: Object<string, ReducerFunction | ReducerAndPrepareObject>
    // A "builder callback" function used to add more reducers, or
    // an additional object of "case reducers", where the keys should be other
    // action types
    extraReducers?:
    | Object<string, ReducerFunction>
    | ((builder: ActionReducerMapBuilder<State>) => void)
})
```

并返回一个对象，包含下列属性：

```js
{
    name : string,
    reducer : ReducerFunction,
    // actionCreator 函数对象
    actions : Record<string, ActionCreator>,
    caseReducers: Record<string, CaseReducer>
}
```

一个真实的切片实例:

```js
import { createSlice } from '@reduxjs/toolkit'

// 会被多处使用，因此提取为一个常量
export const TODO_FEATURE_KEY = 'todos'

// reducer 函数， actions -> actionCreator 函数
const { reducer: todoReducers, actions } = createSlice({
  name: TODO_FEATURE_KEY,
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
        // 可以直接操作 state，不用返回
        state.push(action.payload)
    }
  }
})

export const { addTodo } = actions

export default todoReducers
```

## action预处理

在创建切片时，reducers 定义reducer时，将 reducer 定义为对象，添加 prepare 方法来对 action 进行预处理。prepare 方法接收 patch 时传入的 payload，需要返回一个包含 payload 属性的对象，作为新的 payload，传递给 reducer 函数。

reducers 对象中的 key 将作为 action 生成的 type。

```js
import { createSlice } from '@reduxjs/toolkit'

// 会被多处使用，因此提取为一个常量
export const TODO_FEATURE_KEY = 'todos'

// reducer 函数， actions -> actionCreator 函数
const { reducer: todoReducers, actions } = createSlice({
  name: TODO_FEATURE_KEY,
  initialState: [],
  reducers: {
    addTodo: {
      reducer: (state, action) => {
        // 可以直接操作 state，不用返回
        state.push(action.payload)
      },
      prepare: todo => { // action 预处理
        return {
          payload: {
            ...todo,
            title: 'todo'
          }
        }
      }
    }
  }
})

export const { addTodo } = actions
// 导出给创建 store 时使用
export default todoReducers
```

## 创建 store

通过 configStore 方法创建 store。configStore 方法接收一个参数对象，并返回一个 store 对象。

```js
type ConfigureEnhancersCallback = (
  defaultEnhancers: StoreEnhancer[]
) => StoreEnhancer[]

interface ConfigureStoreOptions<
  S = any,
  A extends Action = AnyAction,
  M extends Middlewares<S> = Middlewares<S>
> {
  /**
   * A single reducer function that will be used as the root reducer, or an
   * object of slice reducers that will be passed to `combineReducers()`.
   * 传递给 combineReducers 的参数
   */
  reducer: Reducer<S, A> | ReducersMapObject<S, A>

  /**
   * An array of Redux middleware to install. If not supplied, defaults to
   * the set of middleware returned by `getDefaultMiddleware()`.
   */
  middleware?: ((getDefaultMiddleware: CurriedGetDefaultMiddleware<S>) => M) | M

  /**
   * Whether to enable Redux DevTools integration. Defaults to `true`.
   *
   * Additional configuration can be done by passing Redux DevTools options
   */
  devTools?: boolean | DevToolsOptions

  /**
   * The initial state, same as Redux's createStore.
   * You may optionally specify it to hydrate the state
   * from the server in universal apps, or to restore a previously serialized
   * user session. If you use `combineReducers()` to produce the root reducer
   * function (either directly or indirectly by passing an object as `reducer`),
   * this must be an object with the same shape as the reducer map keys.
   */
  preloadedState?: DeepPartial<S extends any ? S : S>

  /**
   * The store enhancers to apply. See Redux's `createStore()`.
   * All enhancers will be included before the DevTools Extension enhancer.
   * If you need to customize the order of enhancers, supply a callback
   * function that will receive the original array (ie, `[applyMiddleware]`),
   * and should return a new array (such as `[applyMiddleware, offline]`).
   * If you only need to add middleware, you can use the `middleware` parameter instead.
   */
  enhancers?: StoreEnhancer[] | ConfigureEnhancersCallback
}

function configureStore<S = any, A extends Action = AnyAction>(
  options: ConfigureStoreOptions<S, A>
): EnhancedStore<S, A>
```

实例

```js
import { configureStore } from '@reduxjs/toolkit'
import logger from './middlewares/logger'
import thunk from './middlewares/thunk'
import todoReducers, { TODO_FEATURE_KEY } from './todo'

export default configureStore({
  reducer: { // combineReducer 的参数，合并 reducer
    [TODO_FEATURE_KEY]: todoReducers
  },
  middleware: [thunk, logger],
  // 是否开启调试工具
  devTools: process.env.NODE_ENV !== 'production'
})
```

## 异步操作

reduxToolkit 内置了 redux-thunk ，并对 redux-thunk 进行了二次封装。

异步操作的 actionCreator 通过 createAsyncThunk 来创建异步 action。

createAsyncThunk(type, payloadCreator, options) 接收三个参数

type：action 的type，但是添加了子模块名前缀。如`todo/loadTodos`

payloadCreator：异步执行逻辑函数，函数接收两个参数：

​	payload：dispatch action时传入的参数

​	thunkAPI：redux-thunk的api

options： 包含一些配置选项用来跳过payloadCreator 和 dispatch 的执行

```js
export const asyncAddTodos = createAsyncThunk('todo/asyncAddTodos', (payload, thunkAPI) => {
  setTimeout(() => {
    thunkAPI.dispatch(addTodo(payload))
  }, 2000)
})
```

使用时可以通过mapDispatchToProps将方法映射到props中。

```js
const mapDispatchToProps = dispatch => bindActionCreators({
  addTodo,
  asyncAddTodo
}, dispatch)
```

或者直接使用 dispatch 调用

```js
const dispatch = useDispatch()
const add = () => {
  const val = document.getElementById('input').value
  // asyncAddTodo 是一个 actionCreator 函数
  dispatch(asyncAddTodo(val))
}
```

## 实体适配器

放置数据的容器。将状态放入实体适配器中，实体适配器会提供各种操作状态的方法，简化操作。

### 生成实体适配器

createEntityAdapter(selectId, sortCompare) 接收两个参数

selectedId：将使用该方法的返回值作为实体的id

sortCompare：排序方法，通sort方法的参数。如果传入了，则将对 ids 数组进行排序。

```js
const todoAdapter = createEntityAdapter({
  selectId: instance => instance.id,
  sortComparer: (a, b) => a.id - b.id > 0 ? 1 : -1
})
```

返回一个对象，包含一系列操作数据的方法：

![image-20210630161849229](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210630161849229.png)

数据格式：adapter 会改变原有的数据格式

![image-20210630162002717](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210630162002717.png)

entities：存储具体的数据，以id为key，数据对象为值

ids：存放 id 的数组

adapter.add* 会自动查找第二个参数中payload。因此可以简化

```js
reducers: {
  addTodo: {
    reducer: (state, action) => {
      todoAdapter.addOne(state, action.payload)
    }
}
// 简化
reducers: {
  addTodo: {
    reducer: todoAdapter.addOne
  }
}
```

### 状态选择器

使用实体适配器管理数据时，会改变数据的结构，要想正常使用数据，需要过滤之后才能使用。

```jsx
// 首先过滤要使用的数据，即entities中的内容
const mapStateToProps = (state) => ({
  todos: state[TODO_FEATURE_KEY].entities
})
// 使用数据时，由于 entities 是对象，需要通过 Object.values 遍历访问数据内容
return (
	<ul>
    {
      todoList.map(item => (
        <li key={item.id}>{ item.title }</li>
      ))
		}
  </ul>
)
```

redux 提供了 createSelector 来对状态数据进行过滤

```js
// selectAll 是 adapter中提供的选择器，用于选择所有的实体，并存放到一个数组中
const { selectAll } = todoAdapter.getSelectors()

// state 是store中的state
export const todoSelector = createSelector(state => state[TODO_FEATURE_KEY], selectAll)

// 使用选择器，对数据进行过滤
const todoList = useSelector(todoSelector)
```

