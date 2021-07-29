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