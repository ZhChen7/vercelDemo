# webpack plugin

> Webpack plugin 积累

------

模版：

首先安装插件：

~~~js

~~~

并且调整`webpack.config.js` 文件：

~~~js

~~~

------

**配置选项：** 

------

### HtmlWebpackplugin

> [`HtmlWebpackPlugin`](https://github.com/jantimon/html-webpack-plugin) 简化了 HTML 文件的创建，以便为你的 webpack 包提供服务。这对于那些文件名中包含哈希值，并且哈希值会随着每次编译而改变的 webpack 包特别有用。你可以让该插件为你生成一个 HTML 文件，使用 [lodash 模板](https://lodash.com/docs#template)提供模板，或者使用你自己的 [loader](https://webpack.docschina.org/loaders)

**首先安装插件：**

~~~shell
Webpack 5: npm install --save-dev html-webpack-plugin
Webpack 4: npm install --save-dev html-webpack-plugin@4
~~~

**并且调整`webpack.config.js` 文件：** 

~~~js
plugins : [
  new HtmlWebpackPlugin({
    title:'我是标题',
    template:'./public/index.html',
    filename:'assets/[app].[hash:4].html',
    inject: 'body',
    minify: { // 压缩
      minifyJS: true, // 压缩 HTML 中的 JS
      minifyCSS: true, // 压缩 HTML 中的 CSS
      removeComments: true, // 移除注释
      collapseWhitespace: true, // 去除空格
      // removeEmptyAttributes: true // 去除空属性
      // preserveLineBreaks: true, // 标签分行
    },
    cache: true //设置 js css 文件的缓存，当文件没有发生变化时， 是否设置使用缓存
  })
]
~~~

**index.html**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
  </body>
</html>
```

**配置选项：** 

|       `title`        |           `{String}`            |                 `Webpack App`                 | 用于生成的 HTML 文档的标题                                   |
| :------------------: | :-----------------------------: | :-------------------------------------------: | :----------------------------------------------------------- |
|         名称         |              类型               |                     默认                      | 描述                                                         |
|      `filename`      |       `{String|Function}`       |                `'index.html'`                 | 要将 HTML 写入到的文件。默认为`index.html`. 您也可以在此处指定子目录（例如：）`assets/admin.html`。该`[name]`占位符将与输入的名称所取代。也可以是一个函数，例如`(entryName) => entryName + '.html'`。 |
|      `template`      |           `{String}`            |                      ``                       | `webpack`模板的相对或绝对路径。默认情况下，`src/index.ejs`如果存在，它将使用。请参阅[文档](https://github.com/jantimon/html-webpack-plugin/blob/master/docs/template-option.md)了解详细信息 |
|  `templateContent`   |    `{string|Function|false}`    |                    错误的                     | 可用于代替`template`提供内联模板 - 请阅读[编写您自己的模板](https://github.com/jantimon/html-webpack-plugin#writing-your-own-templates)部分 |
| `templateParameters` |   `{Boolean|Object|Function}`   |                    `false`                    | 允许覆盖模板中使用的参数 - 请参阅[示例](https://github.com/jantimon/html-webpack-plugin/tree/master/examples/template-parameters) |
|       `inject`       |       `{Boolean|String}`        |                    `true`                     | `true || 'head' || 'body' || false`将所有资产注入给定的`template`or `templateContent`。传递时，`'body'`所有 javascript 资源都将放置在 body 元素的底部。`'head'`将脚本放在 head 元素中。传递`true`将根据`scriptLoading`选项将其添加到头部/身体。通过`false`将禁用自动注射。- 请参阅[注入：假示例](https://github.com/jantimon/html-webpack-plugin/tree/master/examples/custom-insertion-position) |
|     `publicPath`     |        `{String|'auto'}`        |                   `'auto'`                    | 用于脚本和链接标签的 publicPath                              |
|   `scriptLoading`    | `{'blocking'|'defer'|'module'}` |                   `'defer'`                   | 现代浏览器支持非阻塞 javascript 加载 ( `'defer'`) 以提高页面启动性能。设置为`'module'`添加属性[`type="module"`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#applying_the_module_to_your_html)。这也意味着“延迟”，因为模块会自动延迟。 |
|      `favicon`       |           `{String}`            |                      ``                       | 将给定的图标路径添加到输出 HTML                              |
|        `meta`        |           `{Object}`            |                     `{}`                      | 允许注入`meta`-tags。例如`meta: {viewport: 'width=device-width, initial-scale=1, shrink-to-fit=no'}` |
|        `base`        |     `{Object|String|false}`     |                    `false`                    | 注入[`base`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base)标签。例如`base: "https://example.com/path/page.html` |
|       `minify`       |       `{Boolean|Object}`        | `true`如果`mode`是`'production'`，否则`false` | 控制是否以及以何种方式应该缩小输出。有关更多详细信息，请参阅下面的[缩小](https://github.com/jantimon/html-webpack-plugin#minification)。 |
|        `hash`        |           `{Boolean}`           |                    `false`                    | 如果`true`然后将唯一的`webpack`编译哈希附加到所有包含的脚本和 CSS 文件。这对于缓存破坏很有用 |
|       `cache`        |           `{Boolean}`           |                    `true`                     | 仅当文件被更改时才发出文件                                   |
|     `showErrors`     |           `{Boolean}`           |                    `true`                     | 错误详细信息将写入 HTML 页面                                 |
|       `chunks`       |              `{?}`              |                      `?`                      | 允许您仅添加一些块（例如，仅单元测试块）                     |
|   `chunksSortMode`   |       `{String|Function}`       |                    `auto`                     | 允许控制块在被包含到 HTML 之前应该如何排序。允许的值为`'none' | 'auto' | 'manual' | {Function}` |
|   `excludeChunks`    |       `{Array.<string>}`        |                      ``                       | 允许您跳过一些块（例如，不要添加单元测试块）                 |
|       `xhtml`        |           `{Boolean}`           |                    `false`                    | 如果`true`将`link`标签呈现为自闭合（XHTML 兼容）             |



### clean-webpack-plugin

> 清除dist目录（打包目录）

首先安装插件：

~~~shell
npm install --save-dev clean-webpack-plugin
~~~

并且调整`webpack.config.js` 文件：

~~~js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

plugins: [
  new CleanWebpackPlugin()
]
~~~



### devtool source map

> 此选项控制是否以及如何生成源映射。
>
> 选择一种[源映射](http://blog.teamtreehouse.com/introduction-source-maps)样式来增强调试过程。这些值会显着影响构建和重建速度。

**背景** ：当您的源代码经过转换后，在浏览器中进行调试就会成为一个问题。**源映射**通过提供原始源代码和转换后的源代码之间的**映射来**解决这个问题。除了将源代码编译为 JavaScript 之外，这也适用于样式。

**建议** ：对于开发，请使用`cheap-module-eval-source-map`. 对于生产，请使用`cheap-module-source-map`.

**注意** ：如果您使用 webpack 4 或更新版本和`mode`选项，该工具将在`development`模式下为您自动生成源映射。但是，生产使用需要注意。

**调整`webpack.config.js` 文件：** 

~~~js
module.exports = {
    devtool: 'source-map'|| false
}
~~~



### webpack-dev-server

> [webpack-dev-server](https://github.com/webpack/webpack-dev-server) 可用于快速开发应用程序

**首先安装插件：**

~~~js
npm install webpack-dev-server --save-dev
~~~

**并且调整`webpack.config.js` 文件：** 

~~~js
module.exports = {
  devServer: {
    static: {
      directory: path.join(__dirname, './dist')
    },
    compress: true,
    historyApiFallback: true,
    https: false,
    open: true,
    hot: true,
    port: 9002,
    proxy: {
      '/api': 'http://localhost:9000'
    }
    devMiddleware: {
    writeToDisk: true,
  },
},
}
~~~

**启动命令shell**：`npx webpack-dev-server`

**NPM Scripts** 

~~~js
// 命令行启动的两种方式
"scripts": {
  "dev": "NODE_ENV=development webpack-dev-server --config webpack.config.js", 
  "serve": "webpack serve"
},
~~~

**配置选项：** 









