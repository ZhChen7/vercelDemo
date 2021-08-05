# useContext

### 本文速览：

- useContext
- useContext 和 context 的 关系

------



## 什么是 useContext 钩子？

随着 React 钩子的出现，另一种使用上下文的方式在 React 16.8 中可用。我们现在可以使用**useContext hook**来使用上下文。

我们可以将整个上下文对象传递`React.useContext()`给组件顶部的上下文，而不是使用渲染道具。

这是上面使用 useContext 钩子的示例：

```js
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
  const value = React.useContext(UserContext);  
    
  return <h1>{value}</h1>;
}
```







## 官方文档

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

