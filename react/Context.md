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

#### **使用 React 上下文有四个步骤：** 

1. 使用该`createContext`方法创建上下文。
2. 获取您创建的上下文并将上下文提供程序包装在您的组件树周围。
3. 使用`value`prop将您喜欢的任何值放在上下文提供程序上。
4. 使用上下文使用者在任何组件中读取该值。

![image.png](https://i.loli.net/2021/08/04/vPhsXjuCSfxKVp7.png)



~~~js
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
import React from 'react';

export const UserContext = React.createContext();

export default function App() {
  return (
    <UserContext.Provider value="Reed">
      <User />
    </UserContext.Provider>
  )
}

function User() {
  return (
    <UserContext.Consumer>
      {value => <h1>{value}</h1>} 
      {/* prints: Reed */}
    </UserContext.Consumer>
  )
}
~~~



#### **让我们一步一步分解我们正在做的事情**：

1. 在我们的`App`组件上方，我们正在创建上下文`React.createContext()`并将结果放入变量中`UserContext`。在几乎所有情况下，您都希望像我们在这里所做的那样导出它，因为您的组件将在另一个文件中。请注意，我们可以`value`在调用时将初始值传递给我们的prop `React.createContext()`。
2. 在我们的`App`组件中，我们使用`UserContext`. 具体来说`UserContext.Provider`。创建的上下文是一个具有两个属性的对象：`Provider`和`Consumer`，这两个都是组件。为了将我们的值传递给应用程序中的每个组件，我们将 Provider 组件包裹在它周围（在本例中为`User`）。
3. 在 上`UserContext.Provider`，我们将想要传递给整个组件树的值。我们将它设置为等于`value`道具来这样做。在这种情况下，它是我们的名字（这里是里德）。
4. 在`User`，或任何我们想要消费（或使用）我们上下文中提供的内容的地方，我们使用消费者组件：`UserContext.Consumer`。为了使用我们传递下来的值，我们使用了所谓的**渲染道具模式**。它只是消费者组件作为道具提供给我们的一个函数。在该函数的返回中，我们可以返回并使用`value`.



### API:

#### 一、`React.createContext`

```js
const MyContext = React.createContext(defaultValue);
```

创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

**只有**当组件所处的树中没有匹配到 Provider 时，其 `defaultValue` 参数才会生效。此默认值有助于在不使用 Provider 包装组件的情况下对组件进行测试。注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效



#### 二、`Context.Provider`

```js
<MyContext.Provider value={/* 某个值 */}>
```

每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

通过新旧值检测来确定变化，使用了与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 相同的算法。

> 注意
>
> 当传递对象给 `value` 时，检测变化的方式会导致一些问题：详见[注意事项](https://zh-hans.reactjs.org/docs/context.html#caveats)。



#### 三、`Class.contextType`

```js
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}
MyClass.contextType = MyContext;
```

挂载在 class 上的 `contextType` 属性会被重赋值为一个由 [`React.createContext()`](https://zh-hans.reactjs.org/docs/context.html#reactcreatecontext) 创建的 Context 对象。此属性能让你使用 `this.context` 来消费最近 Context 上的那个值。你可以在任何生命周期中访问到它，包括 render 函数中。

> 注意：
>
> 你只通过该 API 订阅单一 context。如果你想订阅多个，阅读[使用多个 Context](https://zh-hans.reactjs.org/docs/context.html#consuming-multiple-contexts) 章节
>
> 如果你正在使用实验性的 [public class fields 语法](https://babeljs.io/docs/plugins/transform-class-properties/)，你可以使用 `static` 这个类属性来初始化你的 `contextType`。

```js
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* 基于这个值进行渲染工作 */
  }
}
```



#### 四、`Context.Consumer`

```js
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

一个 React 组件可以订阅 context 的变更，此组件可以让你在[函数式组件](https://zh-hans.reactjs.org/docs/components-and-props.html#function-and-class-components)中可以订阅 context。

这种方法需要一个[函数作为子元素（function as a child）](https://zh-hans.reactjs.org/docs/render-props.html#using-props-other-than-render)。这个函数接收当前的 context 值，并返回一个 React 节点。传递给函数的 `value` 值等价于组件树上方离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。

> 注意
>
> 想要了解更多关于 “函数作为子元素（function as a child）” 模式，详见 [render props](https://zh-hans.reactjs.org/docs/render-props.html)。



#### 五、`Context.displayName`

context 对象接受一个名为 `displayName` 的 property，类型为字符串。React DevTools 使用该字符串来确定 context 要显示的内容。

示例，下述组件在 DevTools 中将显示为 MyDisplayName：

```js
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';
<MyContext.Provider> // "MyDisplayName.Provider" 在 DevTools 中
<MyContext.Consumer> // "MyDisplayName.Consumer" 在 DevTools 中
```









### 参考链接：

官方文档：https://reactjs.bootcss.com/docs/context.html

掘金：		https://juejin.cn/post/6844903566381940744

https://www.freecodecamp.org/news/react-context-for-beginners/



