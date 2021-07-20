react 性能优化的核心是减少真实DOM渲染频率，减少 virtualDOM 对比频率。

# 1. 组件卸载前执行清理操作

当组件中存在定时器，全局绑定的事件时，在组件卸载前需要执行清理操作。



# 2. 通过纯组件提升组件性能

纯组件会对组件的输入数据进行浅层比较，避免重复的渲染。

普通组件每次setState之后都会重新渲染

**浅层比较** 基本数据类型比较值，引用数据类型比较引用。

浅比较和 diff 比较相比，浅比较将消耗更小的性能。diff 比较会遍历整个 virtualDom 树。浅比较只比较当前组件的state 和 props。

## 1. 类组件实现纯组件

```jsx
class Name extends PureComponent {
}
```

## 2. 函数组件实现纯组件

通过 memo 包裹组件来实现

```jsx
const Func = () => ()
const PureComponent = memo(Func)
```



# 3. 通过 shouldComponentUpdate 提升性能

纯组件只浅层比较组件的输入数据，要对组件的输入数据进行深层次的比较，需要通过 shouldComponentUpdate 来完成。

shouldComponentUpdate  返回 true 表示重新渲染，返回 false 不重新渲染

shouldComponentUpdate 接收两个参数，第一个参数为 nextProps，第二个参数为 nextState。



# 4. 通过 memo 方法第二个参数提升性能

memo 内部还是通过浅层比较来对比输入数据的，如果想要深层次的比较数据，需要用到 memo 的第二个参数。

memo 的第二个参数接收一个 compare 方法。

```jsx
memo(Func, compare)
```

compare 接收两个参数，prevProps 和 nextProps。返回一个布尔值，true 表示比较结果相等，不需要渲染， false 表示不相等，需要重新渲染。

```tsx
function compare (prevProps: props, nextProps: props): boolean
```



# 5. 使用懒加载

懒加载能减小单个 bundle.js 的体积，加快首屏文件加载速度。

通过 lazy 和 Suspense 标签来实现

lazy 函数接收一个函数，函数需要返回一个 promise，表示加载文件。

```jsx
const Home = lazy(() => import('./Home.js'))
```

懒加载的模块会打包独立的 bundle 文件，如果想要修改 bundle 文件的前缀，可以通过注释来修改。

```jsx
const Home = lazy(() => import(/* webpackChunkName: "Home" */'./Home.js'))
```

lazy 需要和 Suspense 组件一起使用， Suspense 组件包裹懒加载的组件

```jsx
const Home = lazy(() => import(/* webpackChunkName: "Home" */'./Home.js'))
const Abouot = lazy(() => import(/* webpackChunkName: "About" */'./About.js'))

<Suspense>
	<Route path="/" component={Home} />
	<Route path="/about" component={About} />
</Suspense>
```



# 6. 使用 Fragment 避免额外标记

Fragment 组件不会渲染成任何标签。



# 7. 不要使用内联函数定义

假如使用内联函数定义，每次 render 时都要重新生成新的函数，旧的函数则需要垃圾回收器进行回收，加大了资源消耗。

内联函数定义

```jsx
class extends Component{
  render() {
    return (
    	<input
      	onChange={e => this.setState(e.target.value)}  
      />
    )
  }
}
```

```jsx
class extends Component {
	handleChange(e) {
    this.setState(e.target.value)
  }
  render() {
    return (
    	<input
      	onChange={this.handleChange}  
      />
    )
  }
}
```



# 8. 在构造函数中更正函数的 this 指向

构造函数只会执行一次，所以在构造函数中更正函数的 this 指向也只会执行一次。如果在使用函数时更正函数的this指向，则每次 render 时都会执行 bind 重新生成一个新的函数，造成资源浪费。

在绑定视图时更正函数this指向

```jsx
class extends Component {
	handleChange(e) {
    this.setState(e.target.value)
  }
  render() {
    return (
    	<input
      	onChange={this.handleChange.bind(this)}  
      />
    )
  }
}
```

在构造函数中更正this指向

```jsx
class extends Component {
  constructor() {
    super()
    this.handleChange = this.handleChange.bind(this)
  }
	handleChange(e) {
    this.setState(e.target.value)
  }
  render() {
    return (
    	<input
      	onChange={this.handleChange}  
      />
    )
  }
}
```



# 9. 不要在类组件中使用箭头函数

箭头函数不会存在 this 指向问题。但是箭头函数会被添加为类的实例对象属性，而不是原型对象属性。如果组件被多次重用，每个组件实例对象中都将会有一个相同的函数实例，降低了函数实例的可重用性，造成了资源浪费。

```jsx
class extends Component	{
	handleChange: e => this.setState(e.target.value),
	render() {
    return (
    	<input
      	onChange={this.handleChange.bind}  
      />
    )
  }
}
```



# 10. 避免使用内联样式属性

内联样式属性 style 会被编译成 js 代码。通过 js 代码将样式规则映射到元素上，这涉及到了脚本的执行，增加了 js 代码的执行时间。

```jsx
<div style={{background: 'skyblue'}}>sjjfs</div>
```



# 11. 优化条件渲染

```jsx
if (true) {
  return (
  	<>
    	<Admin />
    	<Header />
    	<Footer />
    </>
  )
} else {
  return (
  	<>
    	<Header />
    	<Footer />
    </>
  )
}
```

上面的代码，在第二次渲染下面的内容是，react  会依次对比每个组件的 Admin 和 Header 对比， Header 和 Footer对比， 发现组件类型不相同时，会先卸载 Admin、Header 和 Footer 组件，然后在渲染 Header 和 Footer 组件，进行了不必要的挂载和卸载。

应该精确控制要进行条件渲染的内容。

```jsx
<>
	{ true && <Admin /> }
  <Header />
  <Footer />
</>
```



# 12. 避免重复无限渲染

不要在render、componentWillUpdate 和 ComponentDidUpdate 生命周期中调用 setState。



# 13. 为组件创建错误边界

默认情况下，组件渲染错误会导致整个程序中断。为组件创建错误边界可以确保组件在发生错误时程序不中断。

错误边界是一个 React 组件，可以捕获子组件在渲染时发生的错误。记录错误，并渲染备用 UI 界面。

错误边界涉及到两个生命周期函数，getDerivedStateFromError 和 componentDidCatch

**getDerivedStateFromError**  是一个静态方法，返回一个对象，该对象将和 state 进行合并，用于更改应用程序状态。当子组件发生错误时，将调用该生命周期函数，参数为错误对象 error。

**componentDidCatch** 用于记录应用程序的错误信息。当子组件发生错误时，将调用该生命周期函数，参数为错误对象 error。

**错误边界不能捕获异步错误**

```jsx
class ErrorBoundaries extends Component{
  constructor() {
    super()
    this.state = {
      hasError: false
    }
  }
  componentDidCatch(error) {
    console.log('发生错误', error);
  }
  static getDerivedStateFromError() {
    return {
      hasError: true
    }
  }
  render() {
    if (this.state.hasError) {
      return <div>发生了错误</div>
    }
    return <App />
  }
}


ReactDOM.render(
  <ErrorBoundaries /> 
  ,
  document.getElementById('root')
);

```



# 14. 避免数据结构突变

props 和 state 中的数据应该保持结构一致，否则会导致输出不一致。



# 15. 依赖优化

使用第三方包时，不想引入包中所有代码，只想要使用哪些引入哪些。可以使用插件对依赖项进行优化。

