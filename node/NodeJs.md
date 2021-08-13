# NodeJs 简介

Node.js 是一个**开源与跨平台**的 JavaScript 运行时环境。 它是一个可用于几乎任何项目的流行工具！

Node.js 在浏览器外运行 **V8** JavaScript 引擎（Google Chrome 的内核）

Node.js 应用程序运行于**单个进程**中，无需为每个请求创建新的线程。 Node.js 在其标准库中提供了一组**异步的 I/O** 原生功能（用以防止 JavaScript 代码被阻塞），并且 Node.js 中的库通常是使用**非阻塞的范式**编写的（从而使阻塞行为成为例外而不是规范）。这使 Node.js 可以在一台服务器上处理数千个并发连接，而无需引入管理线程并发的负担（这可能是重大 bug 的来源）。

在 Node.js 中，可以毫无问题地使用新的 ECMAScript 标准，可以通过更改 Node.js 版本来决定要使用的 ECMAScript 版本，并且还可以通过运行带有标志的 Node.js 来启用特定的实验中的特性。



# NodeJs 重大时间节点

## 2009 

- Node.js 诞生
-  [npm](https://www.npmjs.com/) 第一版发布

## 2010

- [Express](https://expressjs.com/) 诞生

## 2013

- [Koa](https://koajs.com/) 诞生

## 2016

- [Yarn](https://yarnpkg.com/en/) 诞生

 

# NodeJs 安装

通过官网地址 http://nodejs.cn/download/ 下载安装包进行安装



# NodeJS 和 浏览器的差别

- 在浏览器中，大多数时候做的是与 DOM 或其他 Web 平台 API（例如 Cookies）进行交互。 当然，那些在 Node.js 中是不存在的。 没有浏览器提供的 `document`、`window`、以及所有其他的对象
- 在浏览器中，不存在 Node.js 通过其模块提供的所有不错的 API，例如文件系统访问功能。
- 另一个很大的不同是，在 Node.js 中，可以控制运行环境。 除非构建的是任何人都可以在任何地方部署的开源应用程序，否则你能知道会在哪个版本的 Node.js 上运行该应用程序。 与浏览器环境（你无法选择访客会使用的浏览器）相比起来，这非常方便。浏览器发展得慢一些，并且用户的升级速度也慢一些，新版本的 js 特性并不是所有浏览器都支持，所以需要通过 babel 进行转换。
-  Node.js 使用 CommonJS 模块系统，而在浏览器中，则还正在实现 ES 模块标准。



# V8 执行引擎

V8 是为 Google Chrome 提供支持的 JavaScript 引擎的名称。 当使用 Chrome 进行浏览时，它负责处理并执行 JavaScript。

V8 提供了执行 JavaScript 的运行时环境。 DOM 和其他 Web 平台 API 则由浏览器提供。

V8 在其内部编译 javascript，使用了**即时**（JIT）**编译**以加快执行速度。



# Node 程序的运行和退出

## 1. 程序的运行

node 程序的运行通过命令行来启动：

```bash
node filename.js
```



## 2. 程序的退出

有两种方法可以退出程序：

1. 在命令行通过 `ctrl+C` 中断程序运行

2. 在代码中通过 `process.exit(num)` 终止程序运行

   可以传入一个整数，向操作系统发送退出码。默认情况下，退出码为 `0`，表示成功。 不同的退出码具有不同的含义，可以在系统中用于程序与其他程序的通信。详见 http://nodejs.cn/api/process.html#process_exit_codes

   也可以设置 `process.exitCode` 属性：

   ```js
   process.exitCode = 1
   ```

   当程序结束时，Node.js 会返回该退出码。

   当进程完成所有处理后，程序会正常地退出。



# 环境变量的读取

Node.js 的 `process` 核心模块提供了 `env` 属性，该属性承载了在启动进程时设置的所有环境变量。



# 设置和获取命令行参数

## 1. 设置命令行参数

node 命令行参数可以是单个值，也可以是键值对

单值参数的设置：

```bash
node app.js 123
```

键值对参数的设置：

```bash
node app.js name=zs
```



## 2. 获取命令行参数

获取参数值的方法是使用 Node.js 中内置的 `process` 对象。它公开了 `argv` 属性，该属性是一个包含所有命令行调用参数的数组。

**process.argv** 第一个参数是 `node` 命令的完整路径。第二个参数是正被执行的文件的完整路径。

**所有其他的参数从第三个位置开始。**

![image-20210805144053740](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210805144053740.png)

![image-20210805144103548](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210805144103548.png)

![image-20210805144121219](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210805144121219.png)

对于键值对参数，可以通过 [minimist](https://www.npmjs.com/package/minimist) 库来解析

```js
const args = require('minimist')(process.argv.slice(2))
args['name'] //zs
```



# 命令行输出

## 1. console.log

console.log 非常适合在控制台中打印消息。 这就是所谓的标准输出（或称为 `stdout`）

以通过传入变量和格式说明符来格式化用语。

```js
console.log('我的%s已经%d岁', '猫', 2)
```



## 2. console.clear

`console.clear()` 会清除控制台（其行为可能取决于所使用的控制台）。



## 3. console.count

count 方法会对打印的字符串的次数进行计数，并在其旁边打印计数：



## 4. console.trace

打印函数的调用堆栈踪迹

```js
const function2 = () => console.trace()
const function1 = () => function2()
function1()
```

```bash
Trace
    at function2 (repl:1:33)
    at function1 (repl:1:25)
    at repl:1:1
    at ContextifyScript.Script.runInThisContext (vm.js:44:33)
    at REPLServer.defaultEval (repl.js:239:29)
    at bound (domain.js:301:14)
    at REPLServer.runBound [as eval] (domain.js:314:12)
    at REPLServer.onLine (repl.js:440:10)
    at emitOne (events.js:120:20)
    at REPLServer.emit (events.js:210:7)
```



## 5. time()` 和 `timeEnd()

可以使用 `time()` 和 `timeEnd()` 轻松地计算函数运行所需的时间：

```js
const doSomething = () => console.log('测试')
const measureDoingSomething = () => {
  console.time('doSomething()')
  //做点事，并测量所需的时间。
  doSomething()
  console.timeEnd('doSomething()')
}
measureDoingSomething()
```

```bash
D:\study\node\node-demo>node index.js
测试
doSomething(): 7.353ms
```

time 和 timeEnd 需要一一对应，成对出现



## 6. console.error

`console.error` 会打印到 `stderr` 流。它不会出现在控制台中，但是会出现在错误日志中。



## 7. 为输出着色

可以使用[转义序列](https://gist.github.com/iamnewton/8754917)在控制台中为文本的输出着色。 转义序列是一组标识颜色的字符。

```js
console.log('\x1b[33m%s\x1b[0m', '你好') // 输出黄色的你好
```

为控制台输出着色的最简单方法是使用[Chalk](https://github.com/chalk/chalk) 库。

```js
import chalk from 'chalk';

const log = console.log;

// Combine styled and normal strings
log(chalk.blue('Hello') + ' World' + chalk.red('!'));

// Compose multiple styles using the chainable API
log(chalk.blue.bgRed.bold('Hello world!'));

// Pass in multiple arguments
log(chalk.blue('Hello', 'World!', 'Foo', 'bar', 'biz', 'baz'));

// Nest styles
log(chalk.red('Hello', chalk.underline.bgBlue('world') + '!'));

// Nest styles of the same type even (color, underline, background)
log(chalk.green(
	'I am a green line ' +
	chalk.blue.underline.bold('with a blue substring') +
	' that becomes green again!'
));

// ES2015 template literal
log(`
CPU: ${chalk.red('90%')}
RAM: ${chalk.green('40%')}
DISK: ${chalk.yellow('70%')}
`);

// ES2015 tagged template literal
log(chalk`
CPU: {red ${cpu.totalPercent}%}
RAM: {green ${ram.used / ram.total * 100}%}
DISK: {rgb(255,131,0) ${disk.used / disk.total * 100}%}
`);

// Use RGB colors in terminal emulators that support it.
log(chalk.rgb(123, 45, 67).underline('Underlined reddish color'));
log(chalk.hex('#DEADED').bold('Bold gray!'));
```



## 8. 创建进度条

[Progress](https://www.npmjs.com/package/progress) 是一个很棒的软件包，可在控制台中创建进度条。

```js
const ProgressBar = require('progress')

const bar = new ProgressBar(':bar', { total: 10 })
const timer = setInterval(() => {
  bar.tick()
  if (bar.complete) {
    clearInterval(timer)
  }
}, 100)
```



# 从命令行接收输入

Node.js 提供了 [readline模块](http://nodejs.cn/api/readline.html)来执行以下操作：每次一行地从可读流（例如 `process.stdin` 流，在 Node.js 程序执行期间该流就是终端输入）获取输入。

```js
const readline = require('readline').createInterface({
  input: process.stdin,
  output: process.stdout
})

readline.question('你叫什么名字?', name => {
  console.log(`你好 ${name}!`)
  readline.question('你多大了?', age => {
    console.log(`你都${age}岁了!`)
  })
})
```

```bash
D:\study\node\node-demo>node index.js
你叫什么名字?1w
你好 1w!
你多大了?16
你都16岁了!
```

最简单的方式是使用 [readline-sync软件包](https://www.npmjs.com/package/readline-sync)，其在 API 方面非常相似。

[Inquirer.js 软件包](https://github.com/SBoudrias/Inquirer.js)则提供了更完整、更抽象的解决方案。

```js
const inquirer = require('inquirer')

var questions = [
  {
    type: 'input',
    name: 'name',
    message: "你叫什么名字?"
  }
]

inquirer.prompt(questions).then(answers => {
  console.log(`你好 ${answers['name']}!`)
})
```



# 导入导出

## 1. 导出

node 中导出有两种模式：

1. 通过 module.exports 导出，这样导出只能导出单个对象

   ```js
   const car = {
     brand: 'Ford',
     model: 'Fiesta'
   }
   
   module.exports = car
   
   //在另一个文件中
   
   const car = require('./car')
   ```

2. 通过 exports 导出，可以导出多个对象，即指定导出对象的属性

   ```js
   const car = {
     brand: 'Ford',
     model: 'Fiesta'
   }
   
   exports.car = car
   
   // 或
   
   exports.car = {
     brand: 'Ford',
     model: 'Fiesta'
   }
   
   const items = require('./items')
   items.car
   // 或
   const car = require('./items').car
   ```

   

# npm

## 1. 简介

`npm` 是 Node.js 标准的软件包管理器。它起初是作为下载和管理 Node.js 包依赖的方式，但其现在也已成为前端 JavaScript 中使用的工具。



## 2. 安装依赖

### 安装所有依赖

如果项目具有 `package.json` 文件，则通过运行：

```bash
npm install
```

它会在 `node_modules` 文件夹（如果尚不存在则会创建）中安装项目所需的所有东西。



### 安装单个依赖

通过运行以下命令安装特定的软件包：

```bash
npm install <package-name>
```

通常会在此命令中看到更多标志：

- `--save` 安装并添加条目到 `package.json` 文件的 dependencies。
- `--save-dev` 安装并添加条目到 `package.json` 文件的 devDependencies。

区别主要是，`devDependencies` 通常是开发的工具（例如测试的库），而 `dependencies` 则是与生产环境中的应用程序相关。



## 3. 更新软件包

更新软件包通过 `npm update` 命令，和 `install` 命令类似。

```console
npm update
```

`npm` 会检查所有软件包是否有满足版本限制的更新版本。

也可以指定单个软件包进行更新：

```console
npm update <package-name>
```



## 4. 软件包安装位置

当使用 `npm` 安装软件包时，可以执行两种安装类型：

- 本地安装
- 全局安装

默认情况下，当输入 `npm install` 命令时，例如：

```bash
npm install lodash
```

软件包会被安装到当前文件树中的 `node_modules` 子文件夹下。

在这种情况下，`npm` 还会在当前文件夹中存在的 `package.json` 文件的 `dependencies` 属性中添加 `lodash` 条目。

使用 `-g` 标志可以执行全局安装：

```bash
npm install -g lodash
```

在这种情况下，`npm` 不会将软件包安装到本地文件夹下，而是使用全局的位置。

全局的位置到底在哪里？

`npm root -g` 命令会告知其在计算机上的确切位置。



## 5. 使用或执行 npm 安装的软件包

依赖包安装之后要在代码中使用只需要通过 `require` 将其导入到程序中：

```js
const _ = require('lodash')
```

如果软件包是可执行文件，会把可执行文件放到 `node_modules/.bin/` 文件夹下。

可以输入 `./node_modules/.bin/cowsay` 来运行它，或通过 npx/yarn 命令来运行。



## 6. package.json

`package.json` 文件是项目的清单。

不常用属性说明：

`engines`: 设置此软件包/应用程序要运行的 Node.js 或其他命令的版本。

```json
"engines": {
  "node": ">= 6.0.0",
  "npm": ">= 3.0.0",
  "yarn": "^0.13.0"
}
```

`browserslist`: 用于告知要支持哪些浏览器（及其版本）。 Babel、Autoprefixer 和其他工具会用到它，以将所需的 polyfill 和 fallback 添加到目标浏览器。

```json
"browserslist": [
  "> 1%",
  "last 2 versions",
  "not ie <= 8"
]
```

**版本号**：`〜3.0.0` 或 `^0.13.0`。 它们指定了软件包能从该依赖接受的更新。

所有的版本都有 3 个数字，第一个是主版本，第二个是次版本，第三个是补丁版本，详见[规则](http://nodejs.cn/learn/semantic-versioning-using-npm/)。

还可以在范围内组合以上大部分内容，例如：`1.0.0 || >=1.1.0 <1.2.0`，即使用 1.0.0 或从 1.1.0 开始但低于 1.2.0 的版本。



## 7. package-lock.json 文件

 `package-lock.json` 文件旨在跟踪被安装的每个软件包的确切版本，以便产品可以以相同的方式被 100％ 复制（即使软件包的维护者更新了软件包）。

可以使用 semver 表示法设置要升级到的版本（补丁版本或次版本），例如：

- 如果写入的是 `〜0.13.0`，则只更新补丁版本：即 `0.13.1` 可以，但 `0.14.0` 不可以。
- 如果写入的是 `^0.13.0`，则要更新补丁版本和次版本：即 `0.13.1`、`0.14.0`、依此类推。
- 如果写入的是 `0.13.0`，则始终使用确切的版本。

无需将 node_modules 文件夹（该文件夹通常很大）提交到 Git，当尝试使用 `npm install` 命令在另一台机器上复制项目时，如果指定了 `〜` 语法并且软件包发布了补丁版本，则该软件包会被安装。 `^` 和次版本也一样。因此，原始的项目和新初始化的项目实际上是不同的。 即使补丁版本或次版本不应该引入重大的更改，但还是可能引入缺陷。

`package-lock.json` 会固化当前安装的每个软件包的版本，当运行 `npm install`时，`npm` 会使用这些确切的版本。

`package-lock.json` 文件需要被提交到 Git 仓库，以便被其他人获取（如果项目是公开的或有合作者，或者将 Git 作为部署源）。

当运行 `npm update` 时，`package-lock.json` 文件中的依赖的版本会被更新。



## 8. 查看 npm 包的版本

查看所有已安装的 npm 软件包（包括它们的依赖包）的最新版本：

```bash
npm list
```

仅获取顶层的软件包（基本上就是告诉 npm 要安装并在 `package.json` 中列出的软件包）:

```bash
npm list --depth=0
```

通过指定名称来获取特定软件包的版本：

```bash
npm list pkgName
```

查看软件包在 npm 仓库上最新的可用版本:

```bash
npm view pkgName version
```

要列出软件包所有的以前的版本:

```
npm view pkgName versions
```



## 9. 安装 npm 包的旧版本

使用 `@` 语法来安装 npm 软件包的旧版本：

```bash
npm install <package>@<version>
```



## 10 . 将所有 Node.js 依赖包更新到最新版本

首先需要全局地安装 `npm-check-updates` 软件包:

```bash
npm install -g npm-check-updates
```

然后运行：

```bash
ncu -u
```

这会升级 `package.json` 文件的 `dependencies` 和 `devDependencies` 中的所有版本，以便 npm 可以安装新的主版本。

现在可以运行更新了：

```bash
npm update
```

如果只是下载了项目还没有 `node_modules` 依赖包，并且想先安装新的版本，则运行：

```bash
npm install
```



## 11. 卸载 npm 软件包

卸载软件包要通过以下命令行：

```bash
npm uninstall <package-name>
```

如果使用 `-S` 或 `--save` 标志，则此操作还会移除 `package.json` 文件中的引用。

如果程序包是开发依赖项（列出在 `package.json` 文件的 devDependencies 中），则必须使用 `-D` 或 `--save-dev` 标志从文件中移除：

```bash
npm uninstall -S <package-name>
npm uninstall -D <package-name>
```

如果该软件包是全局安装的，则需要添加 `-g` 或 `--global` 标志：

```bash
npm uninstall -g <package-name>
```



# npx

`npx` 是一个非常强大的命令，从 **npm** 的 5.2 版本（发布于 2017 年 7 月）开始可用。

## 1. 运行本地命令

运行 `npx commandname` 会自动地在项目的 `node_modules` 文件夹中找到命令的正确引用，而无需知道确切的路径，也不需要在全局和用户路径中安装软件包。

## 2. 无需安装的命令执行

`npx` 的另一个重要的特性是，无需先安装命令即可运行命令。

1. 不需要安装任何东西。

   ```bash
   D:\study\node\node-demo>npx cowsay "hello" 
   npx: 41 安装成功，用时 15.913 秒
    _______
   < hello >
    -------
           \   ^__^
            \  (oo)\_______
               (__)\       )\/\
                   ||----w |
                   ||     ||
   ```

   还可以直接运行脚手架工具初始化项目，而不用安装脚手架工具：

   - 运行 `vue` CLI 工具以创建新的应用程序并运行它们：`npx @vue/cli create my-vue-app`。

   - 使用 `create-react-app` 创建新的 `React` 应用：`npx create-react-app my-react-app`。

2. 可以使用 @version 语法运行同一命令的不同版本。

   使用 `@` 指定版本，并将其与 [`node` npm 软件包](https://www.npmjs.com/package/node) 结合使用：

   ```bash
   npx node@10 -v #v10.18.1
   npx node@12 -v #v12.14.1
   ```

   这有助于避免使用 `nvm` 之类的工具或其他 Node.js 版本管理工具。



## 3 直接从 URL 运行任意代码片段

`npx` 并不限制使用 npm 仓库上发布的软件包。

可以运行位于 GitHub gist 中的代码，例如：

```bash
npx https://gist.github.com/zkat/4bc19503fe9e9309e2bfaa2c58074d32
```

当然，当运行不受控制的代码时，需要格外小心，因为强大的功能带来了巨大的责任。

