# Vue配置服务器代理

## 1、Vue2和Vue3实现服务器代理的区别

### 1.1、Vue2实现服务器代理

在 vue.config.js文件中配置

```js
module.exports = {
	devServer: {
      proxy: {
      '/api1': {// 匹配所有以 '/api1'开头的请求路径
        target: 'http://localhost:5000',// 代理目标的基础路径
        changeOrigin: true,
        pathRewrite: {'^/api1': ''}
      }
   }
}
```

### 1.2、Vue3实现服务器代理

需要再 `vue.config.js`中配置如下内容

```js
export default defineConfig({
  server: {
    open: '/docs/index.html',
    proxy: {
        '/api1': {// 匹配所有以 '/api1'开头的请求路径
            target: 'http://localhost:5000',// 代理目标的基础路径
            changeOrigin: true,
            pathRewrite: {'^/api1': ''}
        }
  },
})
```

### 1.3、两者区别

- 配置名称不一致，在vue2中是使用`devServer` 进行配置，而vue3则是使用`server`进行配置
- 在vue3中没有使用`module.exports`进行配置，而是通过 vite 的默认文件进行配置