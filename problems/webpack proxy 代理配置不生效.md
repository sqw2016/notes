# 1. 检查后端接口服务是否启动

# 2. 检查配置是否正确

## 1. 请求接口的路径不需要加域名地址

```js
  // BASE_URL: '"http://10.1.2.199:8888/api"' // 无效地址
  BASE_URL: '"/api"'
```

## 2. 配置转发之后的路径是否正确



```js
proxy: {
  '/api': {
    // 以 /api 开头的 localhost:8080 请求将被 替换为 https://api.guthub.com 即localhost:8080/api/user -> https://api.guthub.com/api/user
    target: 'http://10.1.2.199:8092',
    // target: 'http://10.1.2.199:8092/api', // 需要替换掉 api
      // 重写替换后的请求路径，将 './api' 部分替换为 ‘’
      // pathRewrite: {
      //   '/api': ''
      // },
      // 不能使用 localhost:8080 作为请求 GitHub 的主机名
      changeOrigin: true,
  }
}
```

