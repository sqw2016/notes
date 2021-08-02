# 1. 基本概念



## 1. 客户端渲染

CSR: Client Side Rendering

服务端返回数据，data 和 html 在客户端渲染



## 2. 服务端渲染

SSR: Server Side Rendering

数据端获取数据，并将数据和HTML拼接并返回给客户端



## 3. React SSR 同构

同构指的是代码复用，即实现客户端和服务器端最大程度的代码复用



# 2. 客户端渲染存在的问题

## 1. 首屏等待时间长

![image-20210728151516691](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210728151516691.png)

客户端渲染时，用户请求网页的过程：

1. 请求空白页面
2. 加载js脚本
3. 执行脚本
4. 请求数据
5. 渲染页面

## 2. 不利于SEO

客户端渲染时页面结构是空的，搜索引擎的爬取页面时获得的页面内容也是空的。



# 3. SSR 解决的问题

![image-20210728151652754](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210728151652754.png)

解决了首屏空白以及不利于SEO的问题



# 4. ReactSSR 的实现过程



## 1. 创建项目基本结构

创建项目的基本目录结构如下：

![image-20210730101021662](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730101021662.png)

### 1. 依赖

| 依赖                               | 描述                  |
| ---------------------------------- | --------------------- |
| "@babel/cli": "^7.10.3",           | babel命令行工具       |
| "@babel/core": "^7.10.3",          | babel核心库           |
| "@babel/polyfill": "^7.10.4",      | polyfill 浏览器兼容库 |
| "@babel/preset-env": "^7.10.3",    | 兼容 es6 最新特性     |
| "@babel/preset-react": "^7.10.1",  | 兼容 react 语法       |
| "axios": "^0.19.2",                | 请求库                |
| "babel-loader": "^8.1.0",          | 转换 js 文件          |
| "express": "^4.17.1",              | 创建web服务器         |
| "nodemon": "^2.0.4",               | 以watch状态执行 node  |
| "npm-run-all": "^4.1.5",           | 同时执行多个npm命令   |
| "react": "^16.13.1",               | react核心库           |
| "react-dom": "^16.13.1",           | react dom 操作库      |
| "react-helmet": "^6.1.0",          |                       |
| "react-redux": "^7.2.0",           |                       |
| "react-router-config": "^5.1.1",   |                       |
| "react-router-config": "^5.1.1",   |                       |
| "react-router-dom": "^5.2.0",      |                       |
| "redux": "^4.0.5",                 |                       |
| "redux-thunk": "^2.3.0",           |                       |
| "serialize-javascript": "^4.0.0",  |                       |
| "webpack": "^4.43.0",              |                       |
| "webpack-cli": "^3.3.12",          |                       |
| "webpack-dev-server": "^3.11.0",   |                       |
| "webpack-merge": "^4.2.2",         |                       |
| "webpack-node-externals": "^1.7.2" |                       |



## 2. 创建 web 服务器

server 文件夹下创建 http.js 和 index.js 文件，在 http.js 中通过 express 创建web服务器。然后在 index.js 中定义路由监听。

http.js

```js
const express = require('express')

const app = express()

app.listen(3999, () => { console.log('server is running at port 3999'); })

export default app
```

index.js

```js
import app from './http'
import Home from '../share/pages/Home'
import React from 'react'
import { renderToString } from 'react-dom/server'

app.get('/', (req, res) => {
  const content = renderToString(<Home />)
  res.send(`
    <html>
      <head>
        <meta charset="utf-8" />
        <title>React SSR</title>
      </head>
      <body>
        <div id="root">${content}</div>
      </body>
    </html>
  `)
})
```

Home.js

```js
import React from 'react'

export default function Home() {
  return (
    <div>Home</div>
  )
}
```



## 3. 服务端 webpack 打包配置

根目录下创建 webpack.server.js，并添加下面的配置

```js
const path = require('path')

module.exports = {
  entry: './src/server/index.js',
  mode: 'development',
  target: 'node', // 将打包目标环境设置为node，否则无法打包成功
  output: {
    filename: 'bundle.server.js',
    path: path.resolve(__dirname, 'build')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [
            '@babel/preset-env',
            '@babel/preset-react'
          ]
        }
      }
    ]
  }
}
```



## 4. 在 package.json 中配置命令

```json
"scripts": {
  "dev:server-run": "nodemon --watch build --exec \"node build/bundle.server.js\"",
  "dev:server-build": "webpack --config webpack.server.js --watch",
}
```

dev:server-build 以watch模式进行服务端代码打包，如果文件发生变化，则会自动进行打包。

dev:server-run build/bundle.server.js 文件修改时，重新执行文件。

先运行 dev:server-build 打包服务端代码，然后运行dev:server-run启动服务端，之后就可以在 localhost:3999 中看到页面效果。



## 5. 为组件元素添加事件

### 1. 首先尝试直接给 Home 组件的元素添加事件

```js
import React from 'react'

export default function Home() {
  return (
    <div onClick={() => { console.log('hi') }}>Home</div>
  )
}
```

重新启动项目，访问  localhost:3999 ，发现点击事件并不起作用。这是因为服务端返回的 html 中并没有 js 代码。

**解决方法** 在服务端返回的 html 中添加 js 脚本的引入。并执行脚本时重新渲染组件，并为组件添加事件。



### 2. 添加客户端渲染入口

在 client 文件夹下添加 index.js 文件作为客户端渲染的入口文件

client/index.js

```js
import React from 'react'
import ReactDOM from 'react-dom'
import Home from '../share/pages/Home'

// hydrate 和 render 方法类似，都是用来渲染组件，但是 hydrate 渲染组件时会复用已经存在的DOM节点，减少重新生成节点以及删除原本DOM节点的开销
ReactDOM.hydrate(<Home />, document.getElementById('root'))
```



### 3. 添加客户端打包配置

webpack.client.js

```js
const path = require('path')

module.exports = {
  entry: './src/client/index.js',
  mode: 'development',
  output: {
    filename: 'bundle.client.js',
    path: path.resolve(__dirname, 'public')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [
            '@babel/preset-env',
            '@babel/preset-react'
          ]
        }
      }
    ]
  }
}
```



### 4. 在 server/index.js 中添加 js 文件的引入

在返回的html模板中引入打包之后的 js 文件。要想能够正常访问文件，还需要通过 express 添加静态资源访问的功能。

server/index.js

```js
import app from './http'
import Home from '../share/pages/Home'
import React from 'react'
import { renderToString } from 'react-dom/server'

app.get('/', (req, res) => {
  const content = renderToString(<Home />)
  res.send(`
    <html>
      <head>
        <meta charset="utf-8" />
        <title>React SSR</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <!- 添加js引用 ->
        <script src="bundle.client.js"></script>
      </body>
    </html>
  `)
})
```

server/http.js

```js
const express = require('express')

const app = express()

// 开启静态资源访问功能
app.use(express.static('public'))

app.listen(3999, () => { console.log('server is running at port 3999'); })

export default app
```



### 5. 添加客户端打包命令

添加客户端打包命令并通过 npm-run-all 合并命令

```json
"scripts": {
  "dev": "npm-run-all --parallel dev:*",
  "dev:server-run": "nodemon --watch build --exec \"node build/bundle.server.js\"",
  "dev:server-build": "webpack --config webpack.server.js --watch",
  "dev:client-build": "webpack --config webpack.client.js --watch"
}
```



运行 dev 命令之后，访问 localhost:3999 点击元素，即可触发元素的点击效果。这是由于 js 文件重新渲染了组件，并为组件添加了点击事件。



## 6. 合并 webpack 配置

创建 webpack.base.js 文件存储客户端和服务端公共的打包配置。并通过 webpack-merge 插件来合并 webpack 配置再导出。

webpack.base.js

```js
module.exports = {
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [
            '@babel/preset-env',
            '@babel/preset-react'
          ]
        }
      }
    ]
  }
}
```

webpack.server.js

```js
const merge = require('webpack-merge')
const baseConfig = require('./webpack.base')
const path = require('path')

const config = {
  entry: './src/server/index.js',
  target: 'node', // 将打包目标环境设置为node，否则无法打包成功
  output: {
    filename: 'bundle.server.js',
    path: path.resolve(__dirname, 'build')
  },
}

module.exports = merge(baseConfig, config)
```

webpack.client.js

```js
const merge = require('webpack-merge')
const baseConfig = require('./webpack.base')
const path = require('path')

const config = {
  entry: './src/client/index.js',
  output: {
    filename: 'bundle.client.js',
    path: path.resolve(__dirname, 'public')
  }
}

module.exports = merge(baseConfig, config)
```



## 7. 优化服务端打包

服务端打包之后的文件体积为 1.26 MB， 但服务端的代码我们只有 20 行不到的代码。通过下图可以看到将许多没有用到的模板打包了。

![image-20210730112515006](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730112515006.png)



可以通过 webpack-node-externals 插件来避免将 node 中没有用到的模块打包到文件内。 修改 webpack.server.js 文件。

webpack.server.js

```js
const merge = require('webpack-merge')
const baseConfig = require('./webpack.base')
const path = require('path')
const nodeExternal = require('webpack-node-externals')

const config = {
  entry: './src/server/index.js',
  target: 'node', // 将打包目标环境设置为node，否则无法打包成功
  output: {
    filename: 'bundle.server.js',
    path: path.resolve(__dirname, 'build')
  },
  externals: [nodeExternal()]
}

module.exports = merge(baseConfig, config)
```

再次运行打包，可以看到 bundle.server.js 的体积大大减少

![image-20210730113956234](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730113956234.png)



## 8. 优化-代码拆分

将要返回的内容单独放到一个文件中管理。在 server 文件夹下创建 renderer.js 文件，将要返回的内容通过一个方法返回。

server/renderer.js

```js
import Home from '../share/pages/Home'
import React from 'react'
import { renderToString } from 'react-dom/server'

export default () => {
  const content = renderToString(<Home />)
  return `
    <html>
      <head>
        <meta charset="utf-8" />
        <title>React SSR</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script src="bundle.client.js"></script>
      </body>
    </html>
  `
}
```

server/index.js

```js
import app from './http'
import renderer from './renderer'

app.get('/', (req, res) => {
  res.send(renderer())
})
```



优缺点分析：

​	**优点**: 内容文件单独管理，增加了代码的可维护性，可读性，以及可扩展性

​	**缺点**: 增加了打包之后的文件体积

​	![image-20210730120003053](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730120003053.png)



## 9. 路由实现

ssr 的路由包含两部分路由客户端路由和服务端路由

**客户端路由**: 供用户通过交互，跳转页面时使用

**服务端路由**: 在浏览器禁用 js 时，通过地址栏输入地址时使用。

要想实现 srr 路由，需要先定义路由规则，路由规则是客户端和服务端共用的，属于同构代码。

page/routes.js

```js
import Home from './pages/Home'
import ListPage from './pages/ListPage'

const routes = [
  {
    path: '/',
    exact: true,
    component: Home
  },
  {
    path: '/list',
    component: ListPage
  }
]

export default routes
```



### 1. 服务端路由的实现

服务端路由的实现需要通过 StaticRouter 组件来实现，组件表示静态路由，没有跳转功能，访问哪个路由就匹配哪个路由。

在renderer返回服务端渲染的html时，需要根据获取当前访问的路由，然后根据路由匹配要展示的组件。

renderer.js

```js
import React from 'react'
import { renderToString } from 'react-dom/server'
import { StaticRouter } from 'react-router-dom'
import { renderRoutes } from 'react-router-config'
import routes from '../share/routes'


export default req => {
  const content = renderToString(<StaticRouter location={req.path}>
    { renderRoutes(routes) }
  </StaticRouter>)
  return `
    <html>
      <head>
        <meta charset="utf-8" />
        <title>React SSR</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script src="bundle.client.js"></script>
      </body>
    </html>
  `
}
```

同时 app.get 要监听所有路由，然后将再根据路由返回访问的内容。

index.js

```js
import app from './http'
import renderer from './renderer'

app.get('*', (req, res) => {
  res.send(renderer(req))
})
```



### 2. 客户端路由的实现

客户端路由的实现和普通的react应用一致。需要在入口文件中使用 BroswerRouter 组件注册路由。

client/index.js

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { renderRoutes } from 'react-router-config'
import { BrowserRouter as Router } from 'react-router-dom'
import routes from '../share/routes'

// hydrate 和 render 方法类似，都是用来渲染组件，但是 hydrate 渲染组件时会复用已经存在的DOM节点，减少重新生成节点以及删除原本DOM节点的开销
ReactDOM.hydrate(<Router>
  { renderRoutes(routes) }
</Router>, document.getElementById('root'))
```



## 10. ssr redux 

ssr redux 也有两套，客户端 redux 和 服务端 redux。

客户端 redux 就是通过 js 管理 store 中的数据。

服务端 redux 用于管理组件状态数据，和HTML拼接之后返回。

服务端和客户端公用一套redux代码。但是创建 store 的代码由于参数不同，不能共用。



### 1. 创建公共的 redux 代码

share 文件夹下创建 store 文件夹，然后在 store 文件夹下创建 reducers 和 actions 文件夹用来存放 reducer 和 action。

在 actions 中创建 user_action.js 文件，定义远程获取数据的action和更新store中数据的action

store/actions/user_action.js

```js
import Axios from 'axios'

export const FETCH_USER = 'fetch_user'

export const fetchUser = () => async dispatch => {
  const response = await Axios.get('http://jsonplaceholder.typicode.com/users')
  dispatch({
    type: FETCH_USER,
    payload: response.data
  })
}
```

然后在 reducers 文件夹下创建 user_reducer.js 定义 action 对应的更新 Store 中数据的逻辑

store/reducers/user_reducer.js

```js
import { FETCH_USER } from '../actions/user_action';

const useReducer = (state = [], action) => {
  switch (action.type) {
    case FETCH_USER:
      return action.payload
    default:
      return state
  }
}

export default useReducer
```

由于 reducer 会有多个，所以需要合并 reducer，并集中导出。reducers 文件夹下创建 index.js ，然后在 index.js 中通过 combineReducers 合并 reducer。

```js
import { combineReducers } from 'redux'
import userReducer from './user_reducer'

export default combineReducers({
  user: userReducer
})
```



### 2. 客户端 redux 实现

客户端redux的实现和普通 redux项目一致。

#### 1. 创建 store

在 client 文件夹下创建 createStore.js，在此文件中创建客户端 store，并导出，然后在index.js 中使用。

```js
import { applyMiddleware, createStore } from 'redux'
import thunk from 'redux-thunk'
import reducers from '../share/store/reducers'

const store = createStore(reducers, {}, applyMiddleware(thunk))

export default store
```

#### 2. 向下共享 store

通过 Provider 组件向下共享 store ，Provider组件必须是根组件。在 client/index.js 中添加这部分内容

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import { renderRoutes } from 'react-router-config'
import { BrowserRouter as Router } from 'react-router-dom'
import routes from '../share/routes'
import store from './createStore'

// hydrate 和 render 方法类似，都是用来渲染组件，但是 hydrate 渲染组件时会复用已经存在的DOM节点，减少重新生成节点以及删除原本DOM节点的开销
ReactDOM.hydrate(<Provider store={store}>
  <Router>
    { renderRoutes(routes) }
  </Router>
</Provider>, document.getElementById('root'))
```

#### 3. 在组件中使用 Store 中的状态

在组件中通过 connect 向组件中注入状态数据，通过 dispatch 方法触发获取数据的 action。

share/pages/ListPage.js

```jsx
import React, { useEffect } from 'react'
import { connect } from 'react-redux'
import { Link } from 'react-router-dom'
import { fetchUser } from '../store/actions/user_action'

function ListPage({ user, dispatch }) {
  useEffect(() => {
    dispatch(fetchUser())
  }, [])
 return (
  <div>
    <h1>List Page</h1>
    <Link to="/">back to home</Link>
    <ul>
      {
         user.map(itme => (<li key={itme.id}>{ itme.name }</li>))
      }
    </ul>
  </div>
 )
}

const mapStateToProps = (state) => ({
  user: state.user
})

export default connect(mapStateToProps)(ListPage)
```



#### 5. 出现过的问题

##### 1. 浏览器报错

![image-20210730155434527](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730155434527.png)

是因为浏览器不支持异步模式，即 async/await 的使用。需要通过 polyfill 来解决这个问题。使用 polyfill 只需要在安装好 polyfill 之后对 presets 做一个简单的配置。修改 webpack.base.js 中 babel-loader 的 presets 配置如下：

```js
{
  test: /\.js$/,
    exclude: /node_modules/,
      loader: 'babel-loader',
        options: {
          presets: [
            ['@babel/preset-env', {
              useBuiltIns: 'usage' // 这个配置的意思是在使用了浏览器不支持的特性时，自动添加该特性
            }],
            '@babel/preset-react'
          ]
        }
}
```

 

##### 2. 在 /list 路由下直接刷新，页面出现错误

![image-20210730172450306](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730172450306.png)

这是由于直接刷新时，是服务端渲染，服务端渲染还没有配置 store 。



### 3. 服务端 redux 实现

服务端 redux 的实现和客户端不同， 需要服务端去调用接口获取数据，更新 Store，之后再去拼接 HTML 并返回。因为服务端返回 HTML 时，组件的 js 是没法执行的。组件的js只有在客户端执行。

#### 1. 创建 store

服务端的 store 创建是在接收到请求时创建的，因为每个请求的 store 是不一样的，如果使用是一个 store，会造成所以请求都共享一个store，造成数据混乱。

在 server 文件夹下创建 createStore.js 文件，返回一个 createStore 方法。

```js
import { applyMiddleware, createStore } from 'redux'
import thunk from 'redux-thunk'
import reducers from '../share/store/reducers'

const store = createStore(reducers, {}, applyMiddleware(thunk))

export default store
```



#### 2. 向下层组件共享 store

修改 renderer.js 文件， 通过 Provider 组件向下共享 store

```jsx
import React from 'react'
import { renderToString } from 'react-dom/server'
import { StaticRouter } from 'react-router-dom'
import { renderRoutes } from 'react-router-config'
import routes from '../share/routes'
import { Provider } from 'react-redux'

export default (req, store) => {
  const content = renderToString(<Provider store={store}>
  <StaticRouter location={req.path}>
    { renderRoutes(routes) }
  </StaticRouter>
</Provider>)
  return `
    <html>
      <head>
        <meta charset="utf-8" />
        <title>React SSR</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script src="bundle.client.js"></script>
      </body>
    </html>
  `
}
```



#### 3. 组件提供获取数据的方法

服务端在接收到请求时，需要调用组件提供的方法获取数据，来更新 store

修改 ListPage.js ，提供一个 loadData 方法来获取数据

```jsx
import React, { useEffect } from 'react'
import { connect } from 'react-redux'
import { Link } from 'react-router-dom'
import { fetchUser } from '../store/actions/user_action'

function ListPage({ user, dispatch }) {
  useEffect(() => {
    dispatch(fetchUser())
  }, [])
 return (
  <div>
    <h1>List Page</h1>
    <Link to="/">back to home</Link>
    <ul>
      {
         user.map(itme => (<li key={itme.id}>{ itme.name }</li>))
      }
    </ul>
  </div>
 )
}

const mapStateToProps = (state) => ({
  user: state.user
})

function loadData(store) {
  return store.dispatch(fetchUser())
}

export default {
  component: connect(mapStateToProps)(ListPage),
  loadData
}
```



#### 4. 将组件获取数据的方法添加到路由规则中

服务端可以通过访问路径来匹配路由规则，进而获取到组件中获取数据的方法

```js
import Home from './pages/Home'
import ListPage from './pages/ListPage'

const routes = [
  {
    path: '/',
    exact: true,
    component: Home
  },
  {
    path: '/list',
    ...ListPage
  }
]

export default routes
```



#### 5. 服务端在接收到请求时，获取数据更新 store

服务端在接收到请求时，先根据请求路径获取到匹配的路由规则，即要渲染的组件。然后依次调用组件的获取数据的方法，等待方法成功返回之后，然后再将拼接的 html 内容返回。

```js
import app from './http'
import renderer from './renderer'
import createStore from './createStore'
import routes from '../share/routes'
import { matchRoutes } from 'react-router-config'

app.get('*', (req, res) => {
  // 每个请求的 store 是不一样的
  const store = createStore()
  // 获取匹配的路由，可能有多个，调用请求数据的接口
  const promises = matchRoutes(routes, req.path).map(({ route }) => {
    if (route.loadData) return route.loadData(store)
  })
  // 等待数据请求完毕之后
  Promise.all(promises).then(() => {
    res.send(renderer(req, store))
  })
})
```



### 4. 客户端和服务端数据不同步告警问题

直接刷新 /list 页面时，服务端渲染结合数据拼接 HTML，返回的页面结构中 ul 包含多个 li 元素。但是在客户端初次渲染时，ul 是空白的，hydrate 方法监测到这种情况时，会提示告警。

![image-20210730183630075](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210730183630075.png)

要解决这个问题，需要在客户端初次渲染 store 创建时，给 store 一个初始值，这个初始值就是服务端渲染的 store 的值。

服务端怎么把这个值传递给客户端呢？

首先，创建 HTML 结构时，将 store 的值存储在全局变量中

renderer.js

```jsx
import React from 'react'
import { renderToString } from 'react-dom/server'
import { StaticRouter } from 'react-router-dom'
import { renderRoutes } from 'react-router-config'
import routes from '../share/routes'
import { Provider } from 'react-redux'

export default (req, store) => {
  const content = renderToString(<Provider store={store}>
  <StaticRouter location={req.path}>
    { renderRoutes(routes) }
  </StaticRouter>
  </Provider>)
  const initialState = JSON.stringify(store.getState())
  return `
    <html>
      <head>
        <meta charset="utf-8" />
        <title>React SSR</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script>window.INITIAL_STATE = ${initialState}</script>
        <script src="bundle.client.js"></script>
      </body>
    </html>
  `
}
```

然后在客户端创建 store 时，从全局变量中获取初始值。

```js
import { applyMiddleware, createStore } from 'redux'
import thunk from 'redux-thunk'
import reducers from '../share/store/reducers'
// 从 window.INITIAL_STATE 中获取服务端保存的初始值
const store = createStore(reducers, window.INITIAL_STATE, applyMiddleware(thunk))

export default store
```



### 5. 防止 xxs 攻击

假如返回的数据中，含有脚本攻击的代码。我们需要将数据进行转义之后才能使用。

```js
const response = {
  data: [
    {
      id: 1,
      name: '</script><script>alert(1)</script>'
    }
  ]
}
```

解决方法只需要通过 serialize-javascript 插件转化数据即可。使用 serialize-javascript 默认导出的方法替换 JSON.stringify 即可

```js
// const initialState = JSON.stringify(store.getState())
const initialState = serializeJavascript(store.getState())
```

