#   Context

### 本文速览：

- Context

------



## Context

> 是什么？
>
> Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

**注意：在React的官方文档中，[`Context`](https://link.juejin.cn/?target=https%3A%2F%2Freactjs.org%2Fdocs%2Fcontext.html)被归类为高级部分(Advanced)，属于React的高级API，但官方并不建议在稳定版的App中使用Context。** 

不过，这并非意味着我们不需要关注`Context`。事实上，很多优秀的React组件都通过Context来完成自己的功能，比如react-redux的`<Provider />`，就是通过`Context`提供一个全局态的`store`，拖拽组件react-dnd，通过`Context`在组件中分发DOM的Drag和Drop事件，路由组件react-router通过`Context`管理路由状态等等。在React组件开发中，如果用好`Context`，可以让你的组件变得强大，而且灵活。

### 代码讲解：

在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但此种用法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

**Demo:** ( 一层一层传值 ) 

![image.png](https://i.loli.net/2021/08/04/MmBhQlbivaezjkE.png)



~~~js
import React, { useState } from "react";

const App = () => {
  const [count, setcount] = useState(0);
  return (
    <div className="app">
      <Son count={count} />
    </div>
  );
};

const Son = (props) => {
  return (
      <Smallson count={props.count}/>
  );
};

const Smallson = (props) => {
  return (
    <>
      Smallson: {props.count}
    </>
  );
};
~~~



使用 `context`, 我们可以避免通过中间元素传递 props：

![image.png](https://i.loli.net/2021/08/04/vPhsXjuCSfxKVp7.png)



~~~js
import React, { useState } from "react";

// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const MyContext = React.createContext('light');

const App = () => {
  const [count, setcount] = useState(0);

  return (
    <div className="app">
      <MyContext.Provider value='哈哈哈'>
        <Son />
      </MyContext.Provider>
    </div>
  );
};

const Son = (props) => {
  return (
    <>
      <Smallson />
    </>
  );
};

const Smallson = (props) => {
  return (
    <>
      <MyContext.Consumer>
        {
          value => (
            <h1>{value}</h1>
          )
        }
      </MyContext.Consumer>
    </>
  );
};
~~~



### API:









官方文档：https://reactjs.bootcss.com/docs/context.html

掘金：https://juejin.cn/post/6844903566381940744

