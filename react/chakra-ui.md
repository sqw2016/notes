Chakra-ui 是一个简单的，模块化的易于理解的 React UI 组件库。



# 1. 为什么要使用 Chakra-UI

## 1. chakra ui 内置了 emotion

chakra ui 很好的实践了 css-in-js 解决方案。

## 2. chakra ui 是基于样式系统的

样式系统， 

[styled system]: https://styled-system.com/

是一种遵循一定规则的 props 样式属性快速构建 UI 组件的方式。

## 3. 支持开箱即用的主题功能

chakra ui 默认支持白天和黑夜两种主题功能模式

## 4. 拥有大量功能丰富且非常有用的组件

## 5. 使响应式设计轻而易举

## 6. 文档丰富，api查找简单，易于学习

## 7. 适用于构建用于展示给用户的界面

## 8. 持续完善



# 2. chakra-ui 快速集成

## 1. 安装 chakra-ui 最新版本

```bash
yarn add @chakra-ui/react @emotion/react@^11 @emotion/styled@^11 framer-motion@^4
```

## 2. 使用 ChakraProvider 包裹所有组件

即使用 ChakraProvider 作为根组件

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { ChakraProvider } from '@chakra-ui/react'

ReactDOM.render(
  <ChakraProvider>
    <App /> 
  </ChakraProvider>
  ,
  document.getElementById('root')
);
```



# 3. 样式属性

sytle props，适用于更改组件样式的，通过为组件传递属性的方式实现。通过传递简化的样式属性来提升开发效率。并且不再需要关注样式的优先级。

| Prop                  | CSS Property                                  | Theme Key |
| --------------------- | --------------------------------------------- | --------- |
| `m`, `margin`         | `margin`                                      | `space`   |
| `mt`, `marginTop`     | `margin-top`                                  | `space`   |
| `mr`, `marginRight`   | `margin-right`                                | `space`   |
| `me`, `marginEnd`     | `margin-inline-end`                           | `space`   |
| `mb`, `marginBottom`  | `margin-bottom`                               | `space`   |
| `ml`, `marginLeft`    | `margin-left`                                 | `space`   |
| `ms`, `marginStart`   | `margin-inline-start`                         | `space`   |
| `mx`                  | `margin-inline-start` + `margin-inline-end`   | `space`   |
| `my`                  | `margin-top` + `margin-bottom`                | `space`   |
| `p`, `padding`        | `padding`                                     | `space`   |
| `pt`, `paddingTop`    | `padding-top`                                 | `space`   |
| `pr`, `paddingRight`  | `padding-right`                               | `space`   |
| `pe`, `paddingEnd`    | `padding-inline-end`                          | `space`   |
| `pb`, `paddingBottom` | `padding-bottom`                              | `space`   |
| `pl`, `paddingLeft`   | `padding-left`                                | `space`   |
| `ps`, `paddingStart`  | `padding-inline-start`                        | `space`   |
| `px`                  | `padding-inline-start` + `padding-inline-end` | `space`   |
| `py`                  | `padding-top` + `padding-bottom`              | `space`   |
| `color`               | `color`                                       | `colors`  |
| `bg`, `background`    | `background`                                  | `colors`  |
| `bgColor`             | `background-color`                            | `colors`  |
| `opacity`             | `opacity`                                     | none      |



# 4. 切换模式

chakra ui 的主题模式是持久化的，页面刷新也会保持主题模式。因为 chakra ui 将主题存储在了 localStorage 中。同时使用了 类名策略 来确保颜色模式是持久的。

主题模式默认为浅色模式。

```jsx
import { Box, Text, Button, useColorMode } from '@chakra-ui/react'

function App() {
  const {colorMode, toggleColorMode} = useColorMode()
  return (
    <Box>
      <Text>当前模式 {colorMode}</Text>
      <Button w={300} onClick={toggleColorMode}>切换颜色模式</Button>
    </Box>
  );
}

export default App;
```



# 5. 根据模式改变值

通过 useColorModeValue 来设置不同模式对应的值。第一个参数为浅色模式下的值，第二个参数为深色模式下的值。

```jsx
const bgColor = useColorModeValue('lightBlue', 'orange')

return (
<Box w={400} h={300} bgColor={bgColor}>
  <Text>当前模式 {colorMode}</Text>
  <Button w={300} onClick={toggleColorMode}>切换颜色模式</Button>
</Box>
);
```



# 6. 强制组件保持某个颜色模式

需要用到 LightMode 和 DarkMode 两个组件，希望组件保持那个模式，就将组件包裹在哪个组件下面。

```jsx
<LightMode>
  <Button w={300}>保持浅色模式</Button>
</LightMode>
```



# 7. 主题的全局配置

为 ChakraProvider 提供一个 theme 来配合主题，通过 extendTheme 来修改主题的默认配置，并返回一个新的主题。

```jsx
const theme = extendTheme({
  config: {
    useSystemColorMode: true,
    initialColorMode: 'dark',
  }
})


ReactDOM.render(
  <ChakraProvider theme={theme}>
    <App /> 
  </ChakraProvider>
  ,
  document.getElementById('root')
);
```

`initialColorMode` 默认的颜色主题，light / dark

`useSystemColorMode` 使用系统的颜色主题，会随着系统的颜色主题变化而变化。优先级高于 `initialColorMode`



# 8. 主题对象

主题中包含了多个样式对象，如 colors 对象用于设置颜色，space 对象用于设置间距，size对象用于设置大小，radii对象用于设置圆角等。

![image-20210716142146279](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210716142146279.png)

## 1. colors

chakra ui 主题中包含了一些颜色对象，用于设置样式颜色。解析样式属性的值时，首先会从颜色对象中查找，如果找到了就使用颜色对象中的颜色。

![image-20210716140813708](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210716140813708.png)

一种颜色可能有深浅不一的多种情况，在颜色对象中表现为一个个属性，属性的键名为数值，表示颜色的深度。数值越大颜色越深。

使用的通过 color.num 的形式来使用。

```js
  const bgColor = useColorModeValue('red.300', 'orange.700')
```



## 2. space

spaces 对象主要应用于 `padding`, `margin`,  `top`, `left`, `right`, `bottom` 样式.

![image-20210716142212953](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210716142212953.png)

```jsx
<Box m="10" p="5"></Box>
```



## 3. sizes 

sizes 对象主要应用于 `width`, `height`,  `maxWidth`, `minWidth`, `maxHeight`, `minHeight` 样式设置

![image-20210716142252784](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210716142252784.png)

```jsx
<Box w="5" h="4" bgColor={bgColor}></Box>
```

1 表示 `0.25rem`, 之后的数字都是在此基础上按倍值计算。



## 4. breakpoints

breakpoints 对象定义响应式断点，表示页面宽度的一组值。可以为样式属性设置一组值来对应 breakpoints 中的宽度进行显示，表示某个宽度是属性取某个值，从小到大依次匹配。

![image-20210716144749955](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210716144749955.png)

```jsx
<Box m="10" p="5" w={['100px', '200px', '300px', '400px', '500px']}></Box>
```

第一个值是默认值，即没有匹配到 breakpoints 时的值。



# 9. 自定义组件

## 1. 定义组件的 styles

```jsx
const MyButton = {
  baseStyle: {
    borderRadius: 'lg',
  },
  sizes: {
    sm: {
      px: '3',
      py: '1',
      fontSize: '12px'
    },
    md: {
      px: '5',
      py: '2',
      fontSize: '14px'
    }
  },
  variants: {
    danger: {
      bgColor: 'red.400',
      color: 'white'
    },
    primary: {
      bgColor: 'blue.400',
      color: 'white'
    },
  },
  defaultProps: {
    size: 'md',
    variant: 'danger'
  }
}
```

## 2. 在 theme 中注册组件

```jsx
const theme = extendTheme({
  config: {
    useSystemColorMode: true,
    initialColorMode: 'dark',
  },
  // 注册组件
  components: {
    MyButton
  }
})
```

## 3. 定义组件

```jsx
const MyButton = function (props) {

  const { variant, size, children, ...rest } = props
  // 使用注册的组件生成组件样式
  const styles = useStyleConfig('MyButton', {variant, size})
	// __css 具有更低的优先级，可以让用户传入的参数覆盖组件定义的参数
  return <Button __css={styles} {...rest}>{ children }</Button>
}
```

## 4. 使用组件

```jsx
<MyButton size="sm" variant="primary">保持浅色模式</MyButton>
```

