# eslint 安装使用

## 安装

```bash
yarn add eslint -D
```

## 初始化配置

```bash
yarn eslint --init
```

运行上面的命令行，然后按照命令行提示一步步选择默认的 eslint 配置。运行完毕之后会生成一个 eslint 的配置文件 `.eslintrc`。

## vs code 编辑器设置

安装 eslint 扩展，并设置默认语法检查器为 eslint。

```js
setting / editor / default formatter 设置为 eslint。
```

保存时使用 eslint --fix 修复代码中的格式错误：

```js
修改setting，增加下面的配置
"editor.codeActionsOnSave": {
  "source.fixAll": true
}
```



# git提交前进行eslint检查

要在 git 提交前进行 eslint 检查，需要通过 git hooks 来实现。这里需要借助 husky 插件来实现。

## husky安装

运行下列命令安装 husky，会生成一个 .husky 文件夹

```bash
yarn add husky -D
```

husky 安装 git hooks

```bash
yarn husky install
```

## yarn windows 环境下的配置

在 .husky 文件夹下创建一个 pre-commit 文件，存储 git pre-commit 钩子要执行的任务。可以用下列命令行生成。

```bash
 husky add .husky/pre-commit "your cmd"
```

your cmd 就是 commit 之前要执行的命令。要进行 eslint 检查就是`yarn eslint src`。

pre-commit 文件内容。

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"
. "$(dirname "$0")/common.sh"

yarn eslint src
```

windows yarn 环境下还需要再 .husky 文件夹下添加一个 common.sh 文件，文件内容如下：

```bash
command_exists () {
  command -v "$1" >/dev/null 2>&1
}

# Workaround for Windows 10, Git Bash and Yarn
if command_exists winpty && test -t 1; then
  exec < /dev/tty
fi
```

然后提交代码时会先进行 eslint 检查，如果出错将会中断提交。

## lint-staged 的使用



# eslint对象箭头函数解析

在定义类时，有时需要在方法中使用 this ，如下：

```react
class Login extends React.Component {
  componentDidMount () {}

  login = (vals) => {
    const {
      history
    } = this.props
  };

  render () {
    return (
      <div className={styles.main}>
      </div>
    )
  }
}
```

eslint 无法解析对象中的箭头函数，会提示下列错误：

```shell
Parsing error: Unexpected token =
```

如果按照 eslint 的规则修改，需要在 constructor 中将 this 绑定到方法的作用域：

```react
class Login extends React.Component {
  constructor (props) {
    super(props)
    this.login = this.login.bind(this)
  }

  login = (vals) => {
    const {
      history
    } = this.props
  };

  render () {
    return (
      <div className={styles.main}>
      </div>
    )
  }
}
```

每次都需要手动绑定方法的作用域很不方便，需要更换解释器来识别这种写法，使用 babel-eslint 来解析。

安装 babel-eslint，然后修改 .eslintrc.json 配置文件，在配置文件中添加配置:

```json
"parser": "babel-eslint",
```

