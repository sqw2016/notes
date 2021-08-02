# Next.js 介绍

Next.js 是 react 服务端渲染框架，构建利于 SEO 的SPA应用



功能特性：

1. 支持两种预渲染方式：静态生成和服务端渲染
2. 提供基于页面的路由系统，路由零配置
3. 自动代码拆分，优化页面的加载速度
4. 支持静态导出，可将应用导出为静态网站，部署到 CND 服务器
5. 内置 css-in-js 库 styled-jsx，也支持使用其他 的 css-in-js 库，如emotion
6. 方案成熟，可用于生成环境，许多公司都在使用，如腾讯新闻手机端
7. 应用部署简单，拥有专属的部署环境Vercel，也支持在其他拥有 node 的环境下部署



# Next.js 项目创建

有两个命令可以创建 next 项目

第一种官方推荐: 提供初始项目模板

```base
npx create-next-app nextjs-blog --use-npm --example "https://github.com/vercel/next-learn-starter/tree/master/learn-starter"
```

第二种：最少依赖，最初始最干净的初始项目

```bash
npm init next-app appName
```



# 基于页面的路由系统

## 1. 创建页面

在 next.js 中，页面都是放在 pages 文件夹中的 React 组件

组件需要默认被导出

组件文件中不需要导入 React

页面地址与文件地址是对应的关系



## 2. 页面访问

pages/list.js  ----> /list

**pages/index.js  ----> /**

**pages/index.js  ----> /index**

pages/a/b.js  ----> /a/b



## 3. 页面跳转

页面跳转需要使用 <Link> 组件

```jsx
import Link from 'next/link'
import styles from '../styles/Home.module.css'

export default function Home() {
  return (
    <div className={styles.container}>
      <h1>Home Page</h1>
      <Link href="/list">
        <a title="list page">list</a>
      </Link>
    </div>
  )
}
```

Link 组件默认使用 js 跳转，如果 js 被禁用时，将使用 a 标签的连接跳转。

Link 组件只识别 href 属性，添加其他属性将没有任何效果。

如果想要添加属性，需要在 Link 组件中包裹一个 a 标签。

```jsx
import Link from 'next/link'
import styles from '../styles/Home.module.css'

export default function Home() {
  return (
    <div className={styles.container}>
      <h1>Home Page</h1>
      <Link href="/list">
        <a title="list page">list</a>
      </Link>
    </div>
  )
}
```

Link 组件在生成环境下会通过**预取**功能自动优化应用程序以获取最优功能。next 会在浏览器空闲时间下载 Link 目标页面的资源。在打开页面时，能够顾直接看到页面内容。



# 静态资源访问

next 中的静态资默认存在于根目录下的 public 文件夹下。访问路由和页面访问路径规则一致。

public/images/1.jpg  ---->  /images/1.jpg

public/css/base.css  ----> /css/base.css

```jsx
import Image from 'next/image'
import styles from '../styles/Home.module.css'

export default function Home() {
  return (
    <div className={styles.container}>
      <Image alt="test" layout="fill" src="/images/1.jpg" />
    </div>
  )
}
```



# 修改页面元数据

通过在 Head 组件包裹元数据内容来修改页面元数据

```jsx
import Head from 'next/head'
import Link from 'next/link'
import styles from '../styles/Home.module.css'

export default function Home() {
  return (
    <div className={styles.container}>
      <Head>
        <title>首页</title>
        <meta charset="uft-8" />
      </Head>
      <Link href="/list">
        <a title="list page">list</a>
      </Link>
    </div>
  )
}
```



# Next应用中添加样式

## 1. 通过内置的 styled-jsx 实现

```jsx
<Link href="/list">
  <a className="a_color" title="list page">list</a>
</Link>
<style jsx>{`
  .a_color {
  	color: tomato;
  }
`}</style>
<Image alt="test" width="200" height="200" src="/images/1.jpg" />
```



## 2. 通过 css module的方式

Next 也支持 React 默认的 css module 的方式来使用 css。css文件的命名需要遵循 pageName.module.css 这样的规则。css 文件可以是在 pages 和 styles 文件夹下。

```jsx
import Head from 'next/head'
import Image from 'next/image'
import styles from '../styles/Home.module.css'

export default function Home() {
  return (
    <div className={styles.container}>
      <Head>
        <title>首页</title>
        <meta charSet="uft-8" />
      </Head>
      <Image alt="test" width="200" height="200" src="/images/1.jpg" />
    </div>
  )
}
```



## 3. 全局样式添加

pages 中添加 _app.js , 名字是固定。并添加以下内容：

```jsx
import '../styles/globals.css'

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}

export default MyApp
```

在文件中通过 import 导入全局的 css 样式，全局 css 文件可以存在于任何文件夹中。

global.css 示例

```css
html,
body {
  padding: 0;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen,
    Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
  background: orange;
}

a {
  color: inherit;
  text-decoration: none;
}

* {
  box-sizing: border-box;
}
```



# 预渲染

## 1. 概念

预渲染就是数据和HTML在服务端结合生成页面内容并返回

预渲染的好处：有利于 SEO，无需运行 js 即可渲染UI，带来更好的用户体验



## 2. 预渲染的方式

Next 提供了两种预渲染的方式，选择哪种预渲染方式应该根据业务场景进行选择。

### 1. 静态生成

静态生成是在构建时生成 HMTL内容，以后每次请求都公用提前生成好的 HTML。适用于数据更新不频繁的场景，如营销页面，博客文章，电子商务产品列表，帮助文档等。

### 2. 服务端渲染

服务端渲染是在服务端接收到请求时生成HTML内容，每次请求时会生成新的HTML内容。适用于数据更新频繁或内容随请求变化而变化的场景，如商品列表，商品详情等。



# 预渲染实现

## 1. 静态生成的实现

静态生成分为无数据的静态生成和有数据的静态生成。

**Next 默认会进行无数据的静态生成。**

有数据的静态生成，Next 在构建时会先去获取数据，然后根据数据进行静态生成。有数据的静态生成的实现需要通过 getStaticProps 方法来实现。

**getStaticProps** 的作用就是获取静态生成所需的数据，然后通过 props 传递给组件。该方法是一个异步函数，需要在组件内部进行导出。

**getStaticProps 在只在构建时执行一次。**

```jsx
import { readFile } from 'fs'
import Head from 'next/head'
import { join } from 'path'
import { promisify } from 'util'

// 将 readFile 转化成 promise
const read = promisify(readFile)

export default function List({data}) {
 return (
    <div>
      <Head>
        <title>列表</title>
     </Head>
     <h1>list</h1>
     <p>{data}</p>
    </div>
 )
}

export async function getStaticProps() {
  const data = await read(join(process.cwd(), 'pages', '_app.js'), {
    encoding: 'utf-8'
  })
  console.log(data);
  return {
    props: {
      data
    }
  }
}
```



## 2. 服务端渲染实现

服务端渲染的实现需要通过 getServerSideProps 函数实现，和 getStaticProps 类似，都需要从组件中导出。

**getServerSideProps** 在每次请收到请求时都会执行一次。

```jsx
import { readFile } from 'fs'
import Head from 'next/head'
import { join } from 'path'
import { promisify } from 'util'

// 将 readFile 转化成 promise
const read = promisify(readFile)

export default function List({data}) {
 return (
    <div>
      <Head>
        <title>列表</title>
     </Head>
     <h1>list</h1>
     <p>{data}</p>
    </div>
 )
}

export async function getServerSideProps(context) {
  const data = await read(join(process.cwd(), 'pages', '_app.js'), {
    encoding: 'utf-8'
  })
  // 从 context 的 query 获取请求参数
  console.log(context.query);
  return {
    props: {
      data
    }
  }
}
```



# 基于动态路由的静态生成

## 1. 原理介绍

主要是用于显示文章内容，根据文章的id显示不同的文章内容。

基于参数为页面组件生成HTML页面，有多少参数就会生成多少HTML页面。在构建应用时，先获取用户可以访问的所有路由参数，在根据路由参数获取具体数据，然后根据数据生成HTML页面。



## 2. 实现

首先需要创建一个动态路由页面。

基于动态路由的静态生成的实现需要使用到 getStaticPaths 和 getStaticProps 两个方法。 Next 在构建时首先通过 getStaticPaths 获取页面参数列表，遍历参数列表，逐个调用 getStaticProps 方法，获取页面数据传递给组件，静态生成HTML。

```jsx
export default function Post ({data}) {
 return (
    <div>
     <h1>Post</h1>
     <span>{data.id}</span>
     <span>{data.title}</span>
    </div>
 )
}

export async function getStaticPaths() {
  const paths = [
    {
      params: {id: '1'} // 参数的key(即id)需要和动态路由参数对应
    },
    {
      params: {id: '2'}
    }
  ]
  return {
    paths, // 页面路由参数
    fallback: false
  }
}
// params 对应页面路由参数中的 params
export async function getStaticProps({ params }) {
  return {
    props: {
      data: {
        id: params.id,
        title: 'POST' + params.id
      }
    }
  }
}
```

参数的key(即id)需要和动态路由参数对应。

![image-20210802160645225](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210802160645225.png)



## 3. getStaticPaths 方法的返回值中 fallback 的作用

fallback 有两个值，true/false。

fallback 为 false 时，如果路由参数不在 paths 中，则显示 404 页面。

fallback 为 true 时，如果路由参数不在 paths 时，则调用 getStaticProps 方法获取数据，生成 HTML 内容。**同时需要在组件中定义生成页面时展示的内容，通过 router 对象的 isFallback 属性来判断是否是正在生成页面。**如果没有定义，打包将失败。

```jsx
import { useRouter } from 'next/dist/client/router'

export default function Post({ data }) {
  const router = useRouter()
  // 判断是否正在生成页面，true 表示正在生成页面，false 表示页面生成完毕
  if (router.isFallback) {
    return <div>loading...</div>
  }
 return (
    <div>
     <h1>Post</h1>
     <span>{data.id}</span>
     <span>{data.title}</span>
    </div>
 )
}

export async function getStaticPaths() {
  const paths = [
    {
      params: {id: '1'} // 参数的key(即id)需要和动态路由参数对应
    },
    {
      params: {id: '2'}
    }
  ]
  return {
    paths, // 页面路由参数
    // 当页面参数不在 paths 中时，指定行为。 false: 直接返回 404；true: 为参数生成一个对应的页面。
    fallback: true
  }
}

export async function getStaticProps({ params }) {
  return {
    props: {
      data: {
        id: params.id,
        title: 'post' + params.id
      }
    }
  }
}
```



# 自定义 404 页面

只需要在 pages 文件夹下，创建 404.js 文件，默认导出的组件即为404页面展示的内容。



# API Routes

api routes 就是客户端调用的服务端的数据接口。api routes 定义在 pages/api 文件夹下。一个文件对应一个接口，文件默认导出一个方法，接收req请求对象和res返回对象两个参数，并通过res对象返回数据。

![image-20210802172839116](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210802172839116.png)

