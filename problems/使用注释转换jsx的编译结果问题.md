提示错误：pragma and pragmaFrag cannot be set when runtime is automatic

原因： React17 默认采用了新版自动导入运行时，无法使用编译指示器。

解决方法：

# 1. 使用旧的运行时

```js
/** @jsxRuntime classic */
/** @jsx jsx */
import { jsx } from '@emotion/react'
```

`/** @jsxRuntime classic */` 启用旧版本传统模式手动导入运行时

`/** @jsx jsx */` 指明运行时导入



# 2. 导入额外的运行时

```jsx
/** @jsxImportSource @emotion/react */
```

`/** @jsxImportSource @emotion/react */` 导入 @emotion/react 下的所有运行时，就不需要再手动导入 @emotion/react 的 jsx、css 等。



# 3. 配置 babel preset 插件

安装 @emotion/babel-preset-css-prop 插件， 并在 preset 中配置插件

```json
"babel": {
  "presets": [
    "react-app",
    "@emotion/babel-preset-css-prop"
  ]
}
```

