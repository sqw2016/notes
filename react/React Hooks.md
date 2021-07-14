React Hooks是 react16.8 新增的特性，主要是对函数功能进行增强。

Hooks 实际是钩子函数。

# React Hooks解决的问题

## 1. 类组件的不足

### 1. 类组件缺少逻辑复用的机制

在类组件中要实现逻辑复用，需要通过渲染属性props或高阶组件来实现。但是这两者都比较复杂，都是在组件外部嵌套一层没有渲染效果的组件，增加了组件的复杂度，降低了运行效率。

### 2. 类组件经常会变得很复杂

这一点体现在生命周期函数中，一个业务逻辑会被分散到多个生命周期函数中，同时一个生命周期函数会有多个业务逻辑。

### 3. 类成员方法不能保证 this 指向的正确性



# React Hooks 功能介绍

## 1. useState

让函数型组件保存状态

```jsx
function App() {

  const [count, setCount] = useState(0)

  return (
    <div className="App">
      <div>
        <span>{count}</span>
        <button onClick={() => { setCount(count+1) }}>+1</button>
      </div>
    </div>
  );
}
```



#### useState使用细节

1. useState 接收唯一参数，用于设置状态的初始值，可以是任意类型。如果是函数，则函数的返回值为初始状态。函数只会被调用一次。

2. useState 返回一个数组，数组中包含一个状态，和改变状态的方法，方法约定以set开头。方法接收的参数将替换旧的状态。

3. useState 可以被调用多次，用于保存不同的状态值。

4. 改变状态 的 set 方法可以接收一个函数作为参数。函数接收一个旧的状态为参数，返回值将作为新的状态。参数函数的执行是异步的。

   ```jsx
   function App() {
   
     const [count, setCount] = useState(0)
   
     const handlerCountChange = () => {
       setCount(count => count+2)
       document.title = count
     }
   
     return (
       <div className="App">
      		<div>
        		<span>{count}</span>
        		<button onClick={handlerCountChange}>+1</button>
        	</div>
       </div>
     );
   }
   ```

   

## 2. useReducer

让函数组件保存状态的另一种方式。 和 redux 的使用类似。

```jsx
const reducer = function (state, action) {
  switch (action.type) {
    case 'inscrement':
      return state + 1
    case 'decrement':
      return state - 1
    default:
      return state
  }
}

function App() {

  const [count, dispatch] = useReducer(reducer, 0)

  const handlerCountChange = () => {
    dispatch({type: 'inscrement'})
    document.title = count
  }

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <div>
          <span>{count}</span>
          <button onClick={handlerCountChange}>+1</button>
        </div>
      </header>
    </div>
  );
}
```

### useReducer使用细节

1. useReducer 接收一个 reducer 函数，以及一个表示初始值的参数。不同于 useState 的初始参数，这里的初始值参数将原样保存到 useRenducer 返回的状态中。以上面的例子为例，假如第二个参数为一个函数，则则会原样保存到 count 中。

   ```jsx
   const initialValue = () => {
     return 0
   }
   
   function App() {
   
     const [count, dispatch] = useReducer(reducer, initialValue)
   
     const handlerCountChange = () => {
       dispatch({type: 'inscrement'})
       document.title = count
     }
   
     console.log(count === initialValue); // true
   
     return (
       <div className="App">
         <header className="App-header">
           <img src={logo} className="App-logo" alt="logo" />
           <p>
             Edit <code>src/App.js</code> and save to reload.
           </p>
           <div>
             <span>{count === initialValue}</span>
             <button onClick={handlerCountChange}>+1</button>
           </div>
         </header>
       </div>
     );
   }
   ```

2. useReducer 返回一个保存状态的变量和一个触发状态改变的 dispatch 函数。

3. 和 useState 相比的好处是可以向下传递一个 dispatch 方法来触发状态的更新，只要触发 action 就可以更具需求来更新部分状态，而不用关心整体的结构。

## 3. useContext 

使用 createContext 跨组件传递数据时，简化获取数据的代码。

使用 useContext 之前

```jsx
const counterContext = createContext()

function App() {

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <counterContext.Provider value={100}>
          <Foo />
        </counterContext.Provider>
      </header>
    </div>
  );
}

function Foo() {
  return (
    <counterContext.Consumer>
      {
        val => {
          return <div>{ val }</div>
        }
      }
    </counterContext.Consumer>
  )
}
```

使用 useContext 之后，主要简化 Foo 组件中 context 的获取

```jsx
function Foo() {
  const val = useContext(counterContext)
  return <div>{ val }</div>
}
```

## 4. useEffect

让函数组件拥有处理副作用的能力，类似生命周期函数

### 1. useEffect 可以看做是 3 个生命周期函数的结合

#### 1. componentDidMount 和 componentDidUpdate

```jsx
// 在组件挂载之后以及（state）更新之后执行
useEffect(() => {
	console.log(111);
}, [state])
```

#### 2. componentDidMount

```jsx
// 只在组件挂载之后执行一次
useEffect(() => {
  console.log(111);
}, [])
```

#### 3. componentWillUnmount

```jsx
// 返回的函数将在组件将要卸载时执行
useEffect(() => {
  return () => {

  }
}, [])
```

### 2. useEffect 好处

#### 1. 可以按用途将代码分类

相干的业务逻辑可以放在一个副作用函数中，比如要实现搜索功能，条件改变之后重新查询数据

#### 2. 简化重复代码

一般而言组件挂载时和状态更新时执行的操作时一样的，useEffect 将两者结合起来，只需要写一遍，简化了代码。

### 3. useEffect 函数中执行异步操作

useEffect 的返回值只能是一个函数或 undefined，不能是其他类型的值，这意味着下面的写法就不正确了

```jsx
useEffect(async () => {
  const data = await getData()
})
```

上面的写法将报错，async 方法默认会返回一个 Promise 函数，这改变了副作用函数的返回值类型。

要想使用 async 来执行异步操作，可以将 async 函数包裹在只执行函数中。

```jsx
useEffect(() => {
  (async () => {
    const data = await getData()
  })()
})
```



## 5. useMome

useMemo 的行为类似于 Vue 的计算属性，可以监测某个值的变化，根据变化值计算新值，新值可以参与视图渲染。

```jsx
const twiceCount = useMemo(() => {
  return 2 * count
}, [count])
```

### 1. useMemo 使用要点

1. useMemo 接收一个函数和依赖项数组作为参数
2. 函数的返回值将作为 useMemo 的返回值，可以参与视图渲染
3. 只有在依赖项数组中的状态改变之后，才会重新计算 useMemo 的值

### 2. useMemo 能缓存计算结果

只要依赖的状态没有发生改变，useMemo 传入的函数不会重新执行，依旧会使用上一次的计算结果。**即使组件重新渲染**。因此 useMemo 能够避免不必要的计算，提高组件性能。



## 6. memo方法

主要用于性能优化，如果本组件的数据没有发生变化，阻止组件更新。类似与类组件中的 PureComponent 和 ShouldComponentUpdate。

memo 方法接收一个组件，并返回一个新的组件

```jsx
export default memo(App)
```



### 1. 默认情况下子组件在上层组件重新渲染时也会重新渲染

```jsx
function App() {

  const [count, setCount] = useState(0)

  const handlerCountChange = () => {
    setCount(count + 1)
  }

  return (
    <div className="App">
      <div>
        <span>{count}</span>
        <button onClick={handlerCountChange}>+1</button>
      </div>
      <Foo />
    </div>
  );
}

function Foo() {
  console.log('foo');
  return  <div>123123</div>
}
```

### 2. 使用 mome 方法之后，子组件不会重新渲染

```jsx
const Foo = memo(function() {
  console.log('foo');
  return  <div>123123</div>
})
```

## 7. useCallback

缓存函数，使组件重新渲染的时候能得到相同的函数实例，提高性能

```jsx
const reset = useCallback(() => {
  setCount(0)
}, [setCount])
```



### 1. 使用要点

1. useCallback 接收一个函数以及依赖项数组作为参数
2. 返回一个函数，当依赖项没有发生变化时，函数不会改变，**即使组件重新渲染**

### 2. 使用场景

当向子组件传递自定义函数时，即使使用 memo 包裹子组件，在组件重新渲染时，子组件也会重新渲染。因为每次都重新生成了不同的函数实例。实例不同，子组件接收到的数据自然是发生了变化。

```jsx
function App() {

  const [count, setCount] = useState(0)

  const handlerCountChange = () => {
    setCount(count + 1)
  }

  const reset = () => {
    setCount(0)
  }

  return (
    <div className="App">
      <div>
        <span>{count}</span>
        <button onClick={handlerCountChange}>+1</button>
      </div>
      <Foo reset={ reset }/>
    </div>
  );
}

const Foo = memo(function(props) {
  console.log('foo');
  return <div>
    123123
    <button onClick={props.reset}>reset</button>
  </div>
})
```

使用 useCallback 定义的函数，在依赖项没有发生变化时，组件重新渲染不会生成新的函数实例，传入子组件的值没有改变，所以不会重新渲染。

```jsx
function App() {

  const [count, setCount] = useState(0)

  const handlerCountChange = () => {
    setCount(count + 1)
  }

  const reset = useCallback(() => {
    setCount(0)
  }, [setCount])

  return (
    <div className="App">
      <div>
        <span>{count}</span>
        <button onClick={handlerCountChange}>+1</button>
      </div>
      <Foo reset={ reset }/>
    </div>
  );
}

const Foo = memo(function(props) {
  console.log('foo');
  return <div>
    123123
    <button onClick={props.reset}>reset</button>
  </div>
})
```



## 8. useRef

获取 DOM 对象

### 1. 基本使用

```jsx
function App() {

  const [count, setCount] = useState(0)

  const handlerCountChange = () => {
    setCount(count + 1)
    console.log(fooRef);
  }

  const fooRef = useRef()

  return (
    <div className="App">
      <div ref={fooRef}>
        <span>{count}</span>
        <button onClick={handlerCountChange}>+1</button>
      </div>
    </div>
  );
}
```

![image-20210709182937155](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210709182937155.png)

### 2. 使用 useRef 保存数据

将数据挂载到 useRef 的结果上，数据能跨组件周期保存，组件重新渲染不会改变数据。但只能将数据挂载到current 属性中，useRef 的结果是不可扩展的。

```jsx
function App() {

  const [count, setCount] = useState(0)

  const fooRef = useRef()

  useEffect(() => {
    // 数据只能挂载到 current 属性中
    fooRef.current = setInterval(() => {
      setCount(count => count + 1)
    }, 1000)
  }, [])

  const stop = () => {
    clearInterval(fooRef.current)
  }

  return (
    <div className="App">
      <div ref={fooRef}>
        <span>{count}</span>
        <button onClick={handlerCountChange}>+1</button>
        <button onClick={stop}>停止</button>
      </div>
    </div>
  );
}
```



# 自定义HOOK

自定义 hook 是为了在组件之间实现逻辑共享

自定义 hook 是标准的封装和共享逻辑的方式

自定义 hook 是一个以 use 开头的函数

自定义 hook 是逻辑与内置 hook 的组合

```jsx
// 自定义hook，公共逻辑提取
const useInput = initialValue => {
  const [value, setValue] = useState(initialValue)
  return {
    value,
    onChange: e => setValue(e.target.value)
  }
} 

function App() {
  const textInput = useInput('')
  const passwordInput = useInput('')
  const numberInput = useInput(0)

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <div>
          <input type="text" {...textInput} />
          <input type="password" {...passwordInput} />
          <input type="number" {...numberInput} />
        </div>
      </header>
    </div>
  );
}
```



# 路由钩子函数

路由对应的渲染视图的 props 会注入 history、location 以及 match 属性，为了方便获取这些属性，react-router-dom 提供了一些钩子函数来获取这些属性。

![image-20210712110457158](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210712110457158.png)

```jsx
<div>
  <Link to="/home">Home</Link>
  <Link to="/about/张三">About</Link>
</div>
<div>
  <Route path="/home" component={Home} />
  <Route path="/about/:name" component={About} />
</div>
```



```jsx
import { useHistory, useLocation, useParams, useRouteMatch } from 'react-router-dom';

export default function About(props) {
  console.log(useHistory());
  console.log(useLocation());
  console.log(useRouteMatch());
  console.log(useParams());
  return (
    <div>
      About
    </div>
 )
}
```

![image-20210712110758911](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210712110758911.png)



# useState 实现

## 1. 基础 useState 实现

实现 useState 接收初始值，并返回状态及修改状态的方法。状态改变时更新视图

```js
import ReactDOM from 'react-dom'
import App from '../App'
let state
export default function useState(initialState) {
  state = state ? state : initialState
  function setState(newState) {
    state = newState
    // 更新视图
    ReactDOM.render(<App />, document.getElementById('root'))
  }
  return [
    state,
    setState
  ]
}
```

## 2. 多次调用 useState，返回不同的状态

上面的实现多次调用 useState 时，会返回同一个状态。实际的情况是 **useState 每次调用会生成一组新的状态以及状态修改方法**。

使用数组来缓存 state 和 setState 方法，setState 修改状态时，会利用下标来查找对应的状态，所以需要利用闭包缓存当前下标，不然 setState 运行时下标的值会固定为最后一个下标的值。

重新渲染时，由于需要重新执行 useState 方法，所以下标要从 0 开始。

```js
import ReactDOM from 'react-dom'
import App from '../App'

function render() {
  // 重新渲染时，要重新执行 useState，index 要从0开始
  index = 0
  // 更新视图
  ReactDOM.render(<App />, document.getElementById('root'))
}

// 利用闭包缓存 index
function renderSetter(index) {
  return function (newState) {
    states[index] = newState
    render()
  }
}

const states = []
const setStates = []
let index = 0
export default function useState(initialState) {
  // 重新 render 时，避免 state 重复设置为初始值
  if (!states[index]) states[index] = initialState
  if (!setStates[index]) setStates[index] = renderSetter(index)
  const r = [
    states[index],
    setStates[index]
  ]
  index++
  return r
}
```



## 3. 精确重新渲染

useState 更新状态时，只渲染使用状态的组件。



# useEffect 实现

useEffect 接收一个方法以及可选参数依赖数组，当依赖数组中的项发生改变时，执行方法

1. 没有传入依赖项数组时，初始化渲染时和重新渲染时都会执行传入的方法
2. 传入了一个空的依赖项数组时，只有初始化渲染时执行传入的方法
3. 传入了非空的依赖项数组时，当初始化渲染以及依赖项数组中的值发生改变时，都会执行传入的方法

## 1. 基本功能实现

 校验参数，第一个参数必须是函数，如果传入了第二个参数，第二个参数必须是数组

如果没有传入第二个参数，每次都会执行方法

如果传入了第二个参数，则需要和前一次的依赖数组对比，如果发生变化，则要执行参数

```js
let prevDeps = []
export function useEffect(func, deps) {
  if (Object.prototype.toString.call(func) !== '[object Function]') throw new Error('useEffect第一个参数必须是函数')
  if (deps === undefined) {
    func()
  } else {
    if (Object.prototype.toString.call(deps) !== '[object Array]') throw new Error('useEffect第而然个参数必须是数组')
    // 对比依赖是否发生变化，如果变化，则执行 func
    const noChanged = deps.every((dep, i) => dep === prevDeps[i])
    // 依赖向发生变化，执行func函数
    if (!noChanged) {
      func()
    }
    prevDeps = deps
  }
}
```



## 2. 多次执行 useEffect 

多次执行 useEffect 需要使用一个下标记录上一次调用 useEffect 传入的依赖项数组。同时如果重新渲染时，需要重置下标。

```js
let prevDeps = []
export function useEffect(func, deps) {
  if (Object.prototype.toString.call(func) !== '[object Function]') throw new Error('useEffect第一个参数必须是函数')
  if (deps === undefined) {
    func()
  } else {
    if (Object.prototype.toString.call(deps) !== '[object Array]') throw new Error('useEffect第而然个参数必须是数组')
    // 获取上一次的依赖
    const prevDep = prevDeps[effectIndex]
    // 对比依赖是否发生变化，如果变化，则执行 func
    const noChanged = prevDep ? deps.every((dep, i) => dep === prevDep[i]) : false
    // 依赖向发生变化，执行func函数
    if (!noChanged) {
      func()
    }
    prevDeps[effectIndex] = deps
    effectIndex++
  }
}
```

下标的重置要放在 useState 的 render 函数中。



# useReducer 实现

useReducer 的实现可以基于上面的 useState 来实现。只是修改状态是通过 dispatch 方法来修改的。

```js
export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState)
  function dispatch(action) {
    // 生成新的状态
    const newState = reducer(state, action)
    // 更新状态
    setState(newState)
  }
  return [state, dispatch]
}
```

