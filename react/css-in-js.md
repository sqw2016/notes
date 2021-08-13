css-in-js是一种解决方案，将css代码集成到 js 文件中

# 1. css-in-js 为什么产生

css-in-js 解决方案的产生是为了解决 css 的局限性

## 1. css 缺乏动态功能

css 无法动态增减样式。集成到 js 中之后，可以利用 js 特性来动态生成样式属性。

## 2. css 没有作用域的概念

css 都是通过 link 引入的，所有的 css 样式都是混杂在一起的，容易混淆并覆盖样式。使用 css-in-js 解决方案之后，利用 js 来模拟作用域

## 3. 可移植性差

在以组件为单位的开发模式中，组件的 js 和 html 在一个文件，css 在另一个文件中，在组件移植过程中容易遗漏某些文件，使用 css-in-js 解决方案之后组件的所有内容都存放在 js 文件中，移植起来就很方便了。



# 2. css-in-js 优缺点

## 优点

### 1. 让 css 具有作用域

防止 css 代码泄漏到组件外部

### 2. 让组件具有可移植性

实现开箱即用，轻松创建松耦合的应用

### 3. 让组件更具重用性

只需编写一次，即可在任何地方运行，不限于同一项目，只要是在相同框架构建的应用中都能运行

### 4. 让样式具有动态功能

可以将复杂的逻辑应用于样式规则

## 缺点

### 1. 为项目增加了额外的复杂性

### 2. 自动生成的选择器大大降低了代码的可读性



# 3. Emotion

emotion 是一个使用 js 编写 css 的库

## 1. 安装依赖

| 依赖            | 描述 |
| --------------- | ---- |
| @emotion/react  |      |
| @emotion/styled |      |



## 2. 配置 babel 支持 css 属性

emotion 通过 css 属性来设置样式规则，但 babel 默认将 jsx 语法转换成 React.createElement 方法的调用，React.createElement 方法无法正确识别 css 属性。 所以需要将 jsx 语法转换成 @emotion/react 提供的 jsx 语法的调用。

|            | input          | output                                   |
| ---------- | -------------- | ---------------------------------------- |
| **before** | <img src="" /> | `React.createElement('img', { src: ''})` |
| **after**  | <img src="" /> | `jsx('img', { src: ''})`                 |

解决方法有两种：

### 1. 通过注释

```js
/** @jsx jsx */
import { jsx } from '@emotion/react'
```

### 2. 通过babel 插件

安装 @emotion/babel-preset-css-prop 插件，并在 preset 中配置

```json
"babel": {
  "presets": [
    "react-app",
    "@emotion/babel-preset-css-prop"
  ]
}
```



## 3. css 方法的使用

css 方法会返回一个对象，将返回的对象赋值给 css 属性就可以定义元素的样式。

![image-20210715114803544](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210715114803544.png)

最终展示的页面源码:

```html
<div class="css-13x8gvz-style">appwork</div>
```

css 方法有两种使用方式：

### 1. 作为标签函数使用

css 作为标签函数使用，可以在模板字符串中定义样式规则，样式规则的写法和在 css 文件中的写法一致，更符合我们的编码习惯。

```jsx
import { css } from '@emotion/react'

const style = css`
  width: 200px;
  height: 200px;
  background: orange;
`

function App() {
  return (
    <div css={style}>
      appwork
    </div>
  );
}

export default App;
```

### 2. 作为普通方法使用

css 作为方法使用时，需要传递一个对象，对象和传给 style 属性的对象是一样的。

```jsx
import { css } from '@emotion/react'

const style = css({
  width: 200,
  height: 200,
  background: 'pink'
})

function App() {
  return (
    <div css={style}>
      appwork
    </div>
  );
}

export default App;
```



## 4. css 属性的优先级

props 对象中的 css 属性会覆盖组件内部的 css 属性。

```jsx
const Css = (props) => {
  const style = css({
    width: 200,
    height: 200,
    background: 'orange'
  })
	// props的css属性会覆盖组件内部的css属性
  return (
    <div {...props} a={5} css={style}>Csss</div>
  )
}

const style = css({
  width: 200,
  height: 200,
  background: 'pink'
})


function App() {
  return (
    <div>
      <Css a={6} css={style} />
    </div>
  );
}
```

Css 最终的背景颜色为 pink。

![image-20210715135849704](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210715135849704.png)

## 5. 样式化组件

创建带有默认样式的组件

### 1. 样式化组件的创建

和css方法的使用一样，样式化组件的创建也有两种方式：

#### 1. 通过模板字符串来定义样式

```jsx
const Button = styled.button`
  width: 150px;
  height: 40px;
  background: green;
`
function App() {
  return (
    <Container>
      <Button>隐匿</Button>
    </Container>
  );
}
```



#### 2. 通过对象来定义样式

```jsx
const Container = styled.div({
  width: 1000,
  margin: '0 auto',
  background: 'red'
})

function App() {
  return (
    <Container>
      <Button>隐匿</Button>
    </Container>
  );
}
```



### 2. 样式化组件默认样式覆盖

通过 props 来覆盖默认样式。

```jsx
// 模板字符串中样式的覆盖
const Button = styled.button`
  width: 150px;
  height: 40px;
  background: ${props => props.bgColor || 'green'};
`

// 对象中样式的覆盖
const Container = styled.div(props => ({
  width: 1000,
  margin: '0 auto',
  background: props.bgColor || 'red'
}))

// 对象中样式覆盖的另一种方式
const Ul = styled.ul({
  width: 200,
  background: 'blue'
}, props => ({
  width: props.w,
  background: props.bgColor
}))

function App() {
  return (
    <Container bgColor="orange">
      <Button bgColor="pink">隐匿</Button>
      <Ul w={100} bgColor="yellow">
        <li>123</li>
        <li>456</li>
        <li>789</li>
      </Ul>
    </Container>
  );
}
```



## 6. 为组件添加样式

使用 styled 方法处理组件，styled 方法接收组件和样式规则，返回一个新的组件，新的组件中会多出一个 className 属性，可以赋值给标签的 className 属性用于设置样式。

```jsx
import styled from '@emotion/styled'

const Demo = ({className}) => {
  return (
    <div className={className}>Demo</div>
  )
}

const RedDemo = styled(Demo)`
  width: 100px;
  height: 200px;
  background: red;
`

const BlueDemo = styled(Demo)({
  width: 100,
  height: 100,
  background: 'blue'
}) 

function App() {
  return (
    <div>
      <RedDemo />
      <BlueDemo />
    </div>
  );
}

export default App;
```



## 7. 为特定父组件的子组件添加样式

### 1. 通过模板字符串

```jsx
import styled from '@emotion/styled'

const Child = styled.div`
  color: orange;
`

const Parent = styled.div`
  ${Child} {
    color: red;
  }
`

function App() {
  return (
    <div>
      <Child>child</Child>
      <Parent><Child>parent child</Child></Parent>
    </div>
  );
}

export default App;
```

### 2. 通过对象

```jsx
import styled from '@emotion/styled'

const Child = styled.div({
  color: 'orange'
})

const Parent = styled.div({
  [Child]: {
    color: 'red'
  }
})

function App() {
  return (
    <div>
      <Child>child</Child>
      <Parent><Child>parent child</Child></Parent>
    </div>
  );
}

export default App;
```



## 8. css 选择器 &

css 选择器 & 表示元素本身

```jsx
// & 表示元素自身
const Container = styled.div`
  width: 600px;
  height: 400px;
  background: orange;
  color: white;
  &:hover {
    background: pink;
  }
  & > span {
    color: gold;
  }
`

function App() {
  return (
    <div>
      <Container>
        hello
        <span>Span</span>
      </Container>
    </div>
  );
}
```



## 9. as 属性的使用

as 属性可以将组件内部所渲染的元素标签，渲染为 as 属性指定的元素标签。

```jsx
import styled from '@emotion/styled'

const Button = styled.button`
  color: red
`

function App() {
  return (
    <div>
      <Button as="a" href="#">sure</Button>
    </div>
  );
}

export default App;
```



## 10. 样式组合

css 可以接收一个数组，在数组中后面的样式会覆盖前面的样式。

```jsx
import { css } from '@emotion/react';

const warn = css`
  width: 100px;
  background: black;
  color: orange;
`

const danger = css`
  width: 200px;
  background: orange;
  color: black;
`


function App() {
  return (
    <div>
      <div css={[warn, danger]}>Compose</div>
    </div>
  );
}
```

## 11. 全局样式

在任意组件中使用 Global 组件，并向组件的 styles 属性传递样式规则。

```jsx
import { css, Global } from '@emotion/react';

const styles = css`
  body {
    margin: 0;
  }
  a {
    text-decoration: none;
    color: red
  }
`

const Demo = () => {
  return (
    <div>
      <Global styles={styles} />
      <a href="www">子元素a标签</a>
    </div>
  )
}

function App() {
  return (
    <div>
      <a href="#">a标签</a>
      <Demo />
    </div>
  );
}

export default App;
```



## 12. 关键帧动画的定义

```jsx
import { css, keyframes } from '@emotion/react';

const move = keyframes`
  0% {
    left: 0;
    top: 0;
    background: red;
  }
  100% {
    left: 500px;
    top: 500px;
    background: pink;
  }
`

const box = css`
  width: 50px;
  height: 50px;
  color: white;
  position: absolute;
  animation: ${move} 2s ease infinite alternate;
`

function App() {
  return (
    <div>
      <div css={box}>12312</div>
    </div>
  );
}

export default App;
```



## 13. 使用主题功能

### 1. 定义主题样式，并通过 ThemeProvider 共享主题对象

ThemeProvider 需要作为根组件。组件包含一个 theme 属性，接收一个主题对象。

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { ThemeProvider } from '@emotion/react'

const theme = {
  colors: {
    primary: 'tomato'
  }
}

ReactDOM.render(
  <ThemeProvider theme={theme}>
    <App />
  </ThemeProvider>
  ,
  document.getElementById('root')
);
```

### 2. 在组件中使用主题

有两种方式可以在组件中使用主题:

#### 1.  通过 props 获取

向 css 属性传递一个方法， 方法接收 props 作为参数，props中包含主题对象中的属性。

```jsx
import { css,  } from '@emotion/react';

const primaryColor = props => css`
  color: ${props.colors.primary}
`

function App() {
  return (
    <div>
      <div css={primaryColor}>12312</div>
    </div>
  );
}

export default App;
```



#### 2. 使用 useTheme 获取主题对象

```jsx
import { css, useTheme } from '@emotion/react';

function App() {
  // 通过 useTheme 获取主题对象
  const theme = useTheme()

  const box = css`
    color: ${theme.colors.primary}
  `

  return (
    <div>
      <div css={box}>12312</div>
    </div>
  );
}

export default App;
```

