# React Redux

### 本文速览

- 为什么需要 react-redux
- APi的基本使用
- Redux Hook的使用



## react-redux 

> [Redux](https://github.com/reactjs/redux) 官方提供的 React 绑定库。 具有高效且灵活的特性。
>
> 这个库是可以选用的。实际项目中，你应该权衡一下，是直接使用 Redux，还是使用 React-Redux。后者虽然提供了便利，但是需要掌握额外的 API，并且要遵守它的组件拆分规范。



### 安装

React Redux 依赖 **React 0.14 或更新版本。**

```shell
npm install --save react-redux
```



![image.png](https://i.loli.net/2021/08/10/AJVxFqHwkdLDgil.png)



### 为什么需要 react-redux

> 设想我们不知道react-redux库来连接react和redux,来试一下单纯的 **通过redux作为react组件的状态管理 **器；要在各组件之间共享变量store有两种方式可取，一种是通过 **props共享**，一种是通过 **context实现**。
>
> **缺点** ：当组件多的时候，每个组件都需写自己的获取context，订阅事件强制更新自身，获取state，这样的样板代码，实际是没必要的，完全可以把这部分抽象出来，而react-redux就帮我们做了这些，让我们省去了自定义context和订阅事件，获取state等操作

**子组件获取context：**

```js
class VisibleTodoList extends Component {
  componentDidMount() {
    const { store } = this.context;
    this.unsubscribe = store.subscribe(() =>
      this.forceUpdate()
    );
  }

  render() {
    const props = this.props;
    const { store } = this.context;
    const state = store.getState();
    // ...
  }
}

VisibleTodoList.contextTypes = {
  store: React.PropTypes.object
}
```



## APi的基本使用

### `<Provider store>`

> `<Provider store>` 使组件层级中的 `connect()` 方法都能够获得 Redux store。正常情况下，你的根组件应该嵌套在 `<Provider>` 中才能使用 `connect()` 方法。
>
> 如果你**真的**不想把根组件嵌套在 `<Provider>` 中，你可以把 `store` 作为 props 传递到每一个被 `connect()` 包装的组件，但是我们只推荐您在单元测试中对 `store` 进行伪造 (stub) 或者在非完全基于 React 的代码中才这样做。正常情况下，你应该使用 `<Provider>`。

React-Redux 提供`Provider`组件，可以让容器组件拿到`state`。

```js
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp);

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```



### `connect`

> 参数：`connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`
>
> 连接 React 组件与 Redux store。
>
> 连接操作不会改变原来的组件类。
>
> 反而**返回**一个新的已与 Redux store 连接的组件类。

`connect`方法的完整 API 如下

> ```javascript
> import { connect } from 'react-redux'
> 
> const VisibleTodoList = connect(
>   mapStateToProps,
>   mapDispatchToProps
> )(TodoList)
> ```

上面代码中，`connect`方法接受两个参数：`mapStateToProps`和`mapDispatchToProps`。它们定义了 UI 组件的业务逻辑。前者负责输入逻辑，即将`state`映射到 UI 组件的参数（`props`），后者负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。



### `mapStateToProps()`

> `mapStateToProps`是一个函数。它的作用就是像它的名字那样，建立一个从（外部的）`state`对象到（UI 组件的）`props`对象的映射关系。

作为函数，`mapStateToProps`执行后应该返回一个对象，里面的每一个键值对就是一个映射。请看下面的例子

~~~js
const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
~~~

上面代码中，`mapStateToProps`是一个函数，它接受`state`作为参数，返回一个对象。这个对象有一个`todos`属性，代表 UI 组件的同名参数，后面的`getVisibleTodos`也是一个函数，可以从`state`算出 `todos` 的值。

`connect`方法可以省略`mapStateToProps`参数，那样的话，UI 组件就不会订阅Store，就是说 Store 的更新不会引起 UI 组件的更新。





### `mapDispatchToProps()`

`mapDispatchToProps`是`connect`函数的第二个参数，用来建立 UI 组件的参数到`store.dispatch`方法的映射。也就是说，它定义了哪些用户的操作应该当作 Action，传给 Store。它可以是一个函数，也可以是一个对象。

如果`mapDispatchToProps`是一个函数，会得到`dispatch`和`ownProps`（容器组件的`props`对象）两个参数。

> ```javascript
> const mapDispatchToProps = (
>   dispatch,
>   ownProps
> ) => {
>   return {
>     onClick: () => {
>       dispatch({
>         type: 'SET_VISIBILITY_FILTER',
>         filter: ownProps.filter
>       });
>     }
>   };
> }
> ```

从上面代码可以看到，`mapDispatchToProps`作为函数，应该返回一个对象，该对象的每个键值对都是一个映射，定义了 UI 组件的参数怎样发出 Action。

如果`mapDispatchToProps`是一个对象，它的每个键名也是对应 UI 组件的同名参数，键值应该是一个函数，会被当作 Action creator ，返回的 Action 会由 Redux 自动发出。举例来说，上面的`mapDispatchToProps`写成对象就是下面这样。

> ```javascript
> const mapDispatchToProps = {
>   onClick: (filter) => {
>     type: 'SET_VISIBILITY_FILTER',
>     filter: filter
>   };
> }
> ```



## 实例

















### Redux Hook的使用