# 1. 创建 Next 项目

通过下列命令创建 Next 项目

```bash
npm init next-app appName
```



# 2. 下载 Chakra UI 

项目根目录下，使用下面的命令行下载 Chakra UI

```base
yarn add @chakra-ui/react
```



# 3. 下载 Chakra 主题

下载 Chakra 主题，并暴露主题配置

```bash
npx chakra-cli init --theme
```

命令行运行完成之后，根目录下将多出一个 chakra 文件夹

![image-20210803153935572](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210803153935572.png)

克隆完之后，需要对 chakra 中的内容进行调整。/chakra/components/menu.js中

```js
//import { mode，copy } from '@chakra-ui/theme-tools';
// theme-tools 中没有 copy 方法，编译无法通过
import { mode } from '@chakra-ui/theme-tools';
```



# 4. 添加其他依赖

```bash
yarn add react-icons @emotion/react @emotion/styled framer-motion
```



# 5. 让react应用支持 css 属性

要让react应用支持 css 属性，需要用到一个 babel 的扩展 @emotion/babel-preset-css-prop

命令行安装插件

```bash
yarn add @emotion/babel-preset-css-prop
```

根目录下创建 .babelrc 文件，并添加下面的配置

```json
{
  "presets": ["next/babel", "@emotion/babel-preset-css-prop"]
}
```



# 6. 设置主题

应用主题样式，主题样式是全局的，所以应该在全局样式中添加

修改 pages/_app.js 中的内容如下，

```jsx
import { ChakraProvider } from '@chakra-ui/react'
import theme from '../chakra'
import '../styles/globals.css'


function MyApp({ Component, pageProps }) {
  return (
    <ChakraProvider theme={theme}>
      <Component {...pageProps} />  
    </ChakraProvider>
  )
}

export default MyApp
```

