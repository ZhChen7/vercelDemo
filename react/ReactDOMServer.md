# ReactDOMServer

`ReactDOMServer` 对象允许你将组件渲染成静态标记。通常，它被使用在 Node 服务端上：

```js
// ES modules
import ReactDOMServer from 'react-dom/server';
// CommonJS
var ReactDOMServer = require('react-dom/server');
```



### 本文速览

下述方法可以被使用在服务端和浏览器环境。

- [`renderToString()`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostring)
- [`renderToStaticMarkup()`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostaticmarkup)

下述附加方法依赖一个**只能在服务端使用**的 package（`stream`）。它们在浏览器中不起作用。

- [`renderToNodeStream()`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertonodestream)
- [`renderToStaticNodeStream()`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostaticnodestream)



------



## 渲染方法

在服务器端渲染React 组件时，无法使用`React.render()`方法，因为服务端没有`DOM`。React 提供了两个用于服务端渲染组件的方法：[`ReactDOMServer.renderToString()`](http://itbilu.com/javascript/react/EJiqU81te.html#ReactDOMServer-renderToString)和[`ReactDOMServer.renderToStaticMarkup()`](http://itbilu.com/javascript/react/EJiqU81te.html#ReactDOMServer-renderToStaticMarkup)。这两个方法都会将虚拟DOM生成一个HTML字符串，这两个方法移除了服务器端对于浏览器环境的依赖，让React 组件在服务器端渲染成为可能。

这个方法都由React的`ReactDOMServer`对象提供。



### renderToString

> `ReactDOMServer.renderToString()`是开发中主要使用的一个方法。该方法不同于`ReactDOM.render()`，它只接受一个参数，没有`render()`方法中表示渲染位置的参数。`renderToString()`是一个同步（阻塞式）方法，渲染速度非常快，渲染完成后会返回渲染后的字符串。

```js
ReactDOMServer.renderToString(element)
```

将 React 元素渲染为初始 HTML。React 将返回一个 HTML 字符串。你可以使用此方法在服务端生成 HTML，并在首次请求时将标记下发，以加快页面加载速度，并允许搜索引擎爬取你的页面以达到 SEO 优化的目的。

如果你在已有服务端渲染标记的节点上调用 [`ReactDOM.hydrate()`](https://zh-hans.reactjs.org/docs/react-dom.html#hydrate) 方法，React 将会保留该节点且只进行事件处理绑定，从而让你有一个非常高性能的首次加载体验。

```javascript
var MyComponent = React.createClass({
  render: function() {
    return (<h2>itbilu.com</h2>);
  }
});

var html = ReactDOMServer.renderToString(<MyComponent />);
```

上面示例渲染完成后，返回如下一个字符串：

```javascript
<h2 data-reactid=".0" data-react-checksum="-472380432">itbilu.com</h2>
```

React 渲染组件后，为生成的HTML元素增加了两个`data-`前缀的特性。其中`data-reactid`被React 用于区分DOM节点，当`props`或`state`发生变化时，React 可以根据此特性快速的更新DOM。`data-react-checksum`是对创建DOM的校验值，这使得React 可以客户端复用与服务端结构相同的代码，这一特性只会被添加到根元素上。





### renderToStaticMarkup()

> `ReactDOMServer.renderToStaticMarkup()`方法同样是用于在服务端渲染React 组件，该方法除渲染的HTML不会包含`data0-`开头的特性，除此之外与`renderToStaticMarkup()`没有任何区别。

```js
ReactDOMServer.renderToStaticMarkup(element)
```

此方法与 [`renderToString`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostring) 相似，但此方法不会在 React 内部创建的额外 DOM 属性，例如 `data-reactroot`。如果你希望把 React 当作静态页面生成器来使用，此方法会非常有用，因为去除额外的属性可以节省一些字节。

如果你计划在前端使用 React 以使得标记可交互，请不要使用此方法。你可以在服务端上使用 [`renderToString`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostring) 或在前端上使用 [`ReactDOM.hydrate()`](https://zh-hans.reactjs.org/docs/react-dom.html#hydrate) 来代替此方法。

```javascript
var MyComponent = React.createClass({
  render: function() {
    return (<h2>itbilu.com</h2>);
  }
});

var html = ReactDOMServer.renderToStaticMarkup(<MyComponent />);
```

以上示例返回HTML如下：

```javascript
<h2>itbilu.com</h2>
```



#### 渲染方法的选择

`renderToString()`和`renderToStaticMarkup()`方法功能非常相似，我们应该根据自己的需要选择所要使用的方法。

`renderToStaticMarkup()`方法仅当所渲染的组件不打算在客户端使用时才应该选择使用。

大多数我们应该选项`renderToString()`，React 会使用`data-react-checksum`快速的在客户端初始化一个组件，React 会直接使用服务端提供的DOM，这样就省略了创建DOM节点及将组件挂载到DOM中的过程。减少了加载时间，而用户可以更快的与站点交互。

React 会根据`data-react-checksum`检查客户端和服务端渲染的页面结构是否一致。当检测`data-react-checksum`值不一样时，React 舍弃服务端提供的DOM，然后重新渲染组件并将其挂载到页面中，这种情况下将不再拥有服务端渲染带来的性能优势。



### renderToNodeStream

```js
ReactDOMServer.renderToNodeStream(element)
```

将一个 React 元素渲染成其初始 HTML。返回一个可输出 HTML 字符串的[可读流](https://nodejs.org/api/stream.html#stream_readable_streams)。通过可读流输出的 HTML 完全等同于 [`ReactDOMServer.renderToString`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostring) 返回的 HTML。你可以使用本方法在服务器上生成 HTML，并在初始请求时将标记下发，以加快页面加载速度，并允许搜索引擎抓取你的页面以达到 SEO 优化的目的。

如果你在已有服务端渲染标记的节点上调用 [`ReactDOM.hydrate()`](https://zh-hans.reactjs.org/docs/react-dom.html#hydrate) 方法，React 将会保留该节点且只进行事件处理绑定，从而让你有一个非常高性能的首次加载体验。

> 注意:
>
> 这个 API 仅允许在服务端使用。不允许在浏览器使用。
>
> 通过本方法返回的流会返回一个由 utf-8 编码的字节流。如果你需要另一种编码的流，请查看像 [iconv-lite](https://www.npmjs.com/package/iconv-lite) 这样的项目，它为转换文本提供了转换流。



### renderToStaticNodeStream

```js
ReactDOMServer.renderToStaticNodeStream(element)
```

此方法与 [`renderToNodeStream`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertonodestream) 相似，但此方法不会在 React 内部创建的额外 DOM 属性，例如 `data-reactroot`。如果你希望把 React 当作静态页面生成器来使用，此方法会非常有用，因为去除额外的属性可以节省一些字节。

通过可读流输出的 HTML，完全等同于 [`ReactDOMServer.renderToStaticMarkup`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostaticmarkup) 返回的 HTML。

如果你计划在前端使用 React 以使得标记可交互，请不要使用此方法。你可以在服务端上使用 [`renderToNodeStream`](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertonodestream) 或在前端上使用 [`ReactDOM.hydrate()`](https://zh-hans.reactjs.org/docs/react-dom.html#hydrate) 来代替此方法。

> 注意:
>
> 此 API 仅限于服务端使用，在浏览器中是不可用的。
>
> 通过本方法返回的流会返回一个由 utf-8 编码的字节流。如果你需要另一种编码的流，请查看像 [iconv-lite](https://www.npmjs.com/package/iconv-lite) 这样的项目，它为转换文本提供了转换流。

