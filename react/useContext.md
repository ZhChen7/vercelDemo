#   useContext&Context

### 本文速览：

- Context
- useContext

------



## Context

> Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但此种用法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。



~~~react
import React, { useState } from "react";

const App = () => {
  const [count, setcount] = useState(0);

  const handleCount = (num) => {
    setcount(count + 1);
    console.log(num);
  };
  
  console.log("我是father");
  return (
    <div className="app">
      <Son count={count} handleFun={handleCount} />
    </div>
  );
};

const Son = (props) => {
  console.log("我是son");
  return (
    <>
      <h2>Son:{props.count}</h2>
      <Smallson count={props.count} handleFun={props.handleFun} />
    </>
  );
};

const Smallson = (props) => {
  console.log("我是Smallson");
  return (
    <>
      Smallson: {props.count}
      <button
        onClick={() => {
          props.handleFun(1);
        }}
      >
        Smallson点击
      </button>
    </>
  );
};

export default App;
~~~







## 何时使用 Context

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据，例如当前认证的用户、主题或首选语言。举个例子，在下面的代码中，我们通过一个 “theme” 属性手动调整一个按钮组件的样式：

~~~js
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  // Toolbar 组件接受一个额外的“theme”属性，然后传递给 ThemedButton 组件。
  // 如果应用中每一个单独的按钮都需要知道 theme 的值，这会是件很麻烦的事，
  // 因为必须将这个值层层传递所有组件。
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
~~~



## Context的作用

当一些数据需要全局用到的业务场景下：例如切换用户登录信息、切换语言、切换主题，props层层传递数据就显得复杂繁琐。`Context`提供了一种共享数据的方式，不用通过props层层传递数据，在任意一级组件中都可以直接获取到需要的数据。





## Context的使用













## useContext官方解释？

> ```javascript
> const value = useContext(MyContext);
> ```
>
> 接收一个 context 对象（`React.createContext` 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 `<MyContext.Provider>` 的 `value` prop 决定。

当组件上层最近的 `<MyContext.Provider>` 更新时，该 Hook 会触发重渲染，并使用最新传递给 `MyContext` provider 的 context `value` 值。即使祖先使用 [`React.memo`](https://zh-hans.reactjs.org/docs/react-api.html#reactmemo) 或 [`shouldComponentUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)，也会在组件本身使用 `useContext` 时重新渲染。

别忘记 `useContext` 的参数必须是 *context 对象本身*：

- **正确：** `useContext(MyContext)`
- **错误：** `useContext(MyContext.Consumer)`
- **错误：** `useContext(MyContext.Provider)`

调用了 `useContext` 的组件总会在 context 值变化时重新渲染。如果重渲染组件的开销较大，你可以 [通过使用 memoization 来优化](https://github.com/facebook/react/issues/15156#issuecomment-474590693)。

作用：

1. `useContext`可以帮助我们跨越组件层级直接传递变量，实现共享
2. `Context`的作用就是对它所包含的组件树提供全局共享数据的一种技术













### **总结**

1. `useContext`相当于`<MyContext.Consumer>`来使用，使用方法如上面的例子。
2. `useContext`只能让你能够读取`context`的值以及订阅`context`的变化。需要在上层组件树中使用`<MyContext.Provider>`来为下层组件提供`context`。

综上，可以看到，`useContext`解决了组件间多层传递`props`状态值的问题，但是`useContext`本身并不能改变`state`的值。而`useReducer`却可以解决复杂状态下状态值`state`的管理。因此，可以将全局状态值放入`useReducer`中，将返回的`state`和`dispatch`通过`<MyContext.Provider>`进行传值，这样在子组件中就可以用`useContext`来接收相应的状态值，并通过`dispatch`对状态值进行操作。这样，就可以将`useContext`和`useReducer`相结合实现`Redux`的功能！





