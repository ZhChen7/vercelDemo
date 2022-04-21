# 自定义plugin

> wepack 自定义 plugin
>
> 生命周期钩子函数官网：https://webpack.docschina.org/api/compiler-hooks/#done



# compiler 钩子

`Compiler` 模块是 webpack 的主要引擎，它通过 [CLI](https://webpack.docschina.org/api/cli) 或者 [Node API](https://webpack.docschina.org/api/node) 传递的所有选项创建出一个 compilation 实例。 它扩展（extends）自 `Tapable` 类，用来注册和调用插件。 大多数面向用户的插件会首先在 `Compiler` 上注册。

## 钩子

以下生命周期钩子函数，是由 `compiler` 暴露， 可以通过如下方式访问：

```js
compiler.hooks.someHook.tap('MyPlugin', (params) => {
  /* ... */
});
```

取决于不同的钩子类型，也可以在某些钩子上访问 `tapAsync` 和 `tapPromise`。

关于钩子类型的描述，请查看 [Tapable 文档](https://github.com/webpack/tapable#tapable).



### webpack.prod.conf.js

~~~js
const { ZcPlugin } = require('../plugin/customPlugin.js')

plugins: [
  new ZcPlugin({
    productName: 'xxxxx',
    platform: 'pc'
  }),
  new CleanWebpackPlugin(),
  new webpack.HotModuleReplacementPlugin()
],
~~~



### 写一个自定义plugin：ZcPlugin

~~~js
class ZcPlugin {
  constructor(config) {
    this.productName = config.productName
    this.platform = config.platform
  }

  apply(compiler){
    compiler.hooks.done.tap('ZcPlugin', (stats) => {
      console.log('stats', stats);
      console.log('哈哈哈哈哈');
      console.log('xxx', this.productName, this.platform);
    });
  }
}

module.exports = {
  ZcPlugin
}
~~~



