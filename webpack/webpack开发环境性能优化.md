# webpack开发环境性能优化

## 1 性能分析

### 1.1 构建速度分析

1. 安装依赖

```javascript
yarn add speed-measure-webpack-plugin -D
```

​	2. 配置`webpack.config.js`。使用`smp.wrap`包裹`webpack`配置

```javascript
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

const smp = new SpeedMeasurePlugin();

const webpackConfig = smp.wrap({
  plugins: [new MyPlugin(), new MyOtherPlugin()],
});
```

	3. 启动项目，开始分析，可以得到如下图的结果，清晰的看到每个阶段的耗时。

![image-20210610172310989](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210610172310989.png)

### 1.2 构建产生的bundle包分析

1. 安装依赖

```javascript
yarn add webpack-bundle-analyzer -D 
```

2. `wepack.config.js`打包配置

```javascript
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
}
```

3. 启动项目，会启动一个本地web服务显示各个bundle的大小

![image-20210610174149124](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210610174149124.png)

## 2 性能优化

### 2.1 开启`webpack`缓存

只需要在`webpack`配置中添加`cache:true`

### 2.2 使用`hard-source-webpack-plugin`插件

`hard-source-webpack-plugin`插件会缓存打包结果

安装依赖

```javascript
yarn add hard-source-webpack-plugin -D
```

修改`webpack`配置

```javascript
// webpack.config.js
var HardSourceWebpackPlugin = require('hard-source-webpack-plugin');
 
module.exports = {
  context: // ...
  entry: // ...
  output: // ...
  plugins: [
    new HardSourceWebpackPlugin()
  ]
}
```

启动项目

