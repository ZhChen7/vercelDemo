# Portals

### 速览

- Portals 基本概念



------



## 基本概念

> React Portal 是一种优秀的方法，可以将子组件渲染到由组件树层次结构定义的父 DOM 层次结构之外的 DOM 节点中。
>
> Portal 的最常见用例是子组件需要从视觉上脱离父容器的情况，

```js
ReactDOM.createPortal(child, container)
```

第一个参数（`child`）是任何[可渲染的 React 子元素](https://zh-hans.reactjs.org/docs/react-component.html#render)，例如一个元素，字符串或 fragment。第二个参数（`container`）是一个 DOM 元素【 应该注入到的 DOM 位置】。



## Portals 基本使用

通常来讲，当你从组件的 render 方法返回一个元素时，该元素将被挂载到 DOM 节点中离其最近的父节点：

```js
render() {
  // React 挂载了一个新的 div，并且把子元素渲染其中
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

然而，有时候将子元素插入到 DOM 节点中的不同位置也是有好处的：

```js
render() {
  // React 并*没有*创建一个新的 div。它只是把子元素渲染到 `domNode` 中。
  // `domNode` 是一个可以在任何位置的有效 DOM 节点。
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```

一个 portal 的典型用例是当父组件有 `overflow: hidden` 或 `z-index` 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框：



## 为什么需要 Portals？

> 当我们在特定元素（父组件）中使用模态时，模态的高度和宽度将从模态所在的组件继承。因此，模态可能会被裁剪，而无法在应用程序中正确显示。传统上模态需要 CSS 属性，如 overflow:hidden 和 z-index，以避免出现这一问题。

### 子组件需要从视觉上脱离父容器的情况:

- 模态对话框
- 工具提示
- 悬浮卡片
- 加载器



以下代码将使用 **createPortal()**，在 **root** 树层次结构之外创建一个 DOM 节点来解决这个问题。

~~~js
const Modal = ({ message, isOpen, onClose, children }) => {
  if (!isOpen) return null;
  return ReactDOM.createPortal(
    <div className="modal">
      <span>{message}</span>
      <button onClick={onClose}>Close</button>
    </div>
    , document.body);
}

export default function App() {
  const [open, setOpen] = useState(false)
  return (
    <div className="component">
      <button onClick={() => setOpen(true)}>Open Modal</button>
      <Modal
        message="Hello World!"
        isOpen={open}
        onClose={() => setOpen(false)}
      />
    </div>
  )
}
~~~

下面显示的是一个 DOM 树层次结构，这是在使用 React Portal 时产生的，其中模态将被注入到 **root** 的外部，并与 **root** 处于同一级别。

![image.png](https://i.loli.net/2021/08/22/KeDJEXLW1VrHuYQ.png)



## 使用 Portal 时要注意的事项

使用 React Portal 时，你应该注意几个问题。

- 事件冒泡（Event Bubbling）将照常工作：通过将事件传播到 React 树的祖先，事件冒泡将按预期工作，而与 DOM 中的 Portal 节点位置无关。
- React 可以控制 Portal 节点及其生命周期：通过 Portal 渲染子元素时，React 仍然可以控制其生命周期。
- Portal 仅影响 DOM 结构：Portal 仅影响 HTML DOM 结构，而不影响 React 组件树。
- 预定义 HTML 挂载点：使用 Portal 时，你需要定义一个 HTML DOM 元素作为 Portal 组件的挂载点。







## 通过 Portal 进行事件冒泡

> 尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 *React 树*， 且与 *DOM 树* 中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。

这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 *React 树*的祖先，即便这些元素并不是 *DOM 树* 中的祖先。假设存在如下 HTML 结构：

```html
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

在 `#app-root` 里的 `App` 组件能够捕获到未被捕获的从兄弟节点 `#modal-root` 冒泡上来的事件。

```js
import React, { useState } from 'react'
import ReactDOM from 'react-dom';

// 在 DOM 中有两个容器是兄弟级 （siblings）
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // 在 Modal 的所有子元素被挂载后，
    // 这个 portal 元素会被嵌入到 DOM 树中，
    // 这意味着子元素将被挂载到一个分离的 DOM 节点中。
    // 如果要求子组件在挂载时可以立刻接入 DOM 树，
    // 例如衡量一个 DOM 节点，
    // 或者在后代节点中使用 ‘autoFocus’，
    // 则需添加 state 到 Modal 中，
    // 仅当 Modal 被插入 DOM 树中才能渲染子元素。
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el
    );
  }
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { clicks: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      clicks: state.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // 这个按钮的点击事件会冒泡到父元素
  // 因为这里没有定义 'onClick' 属性
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}
```

在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 `<Modal />` 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件。

![image.png](https://i.loli.net/2021/08/22/y1oEVORge58vzH2.png)

