## Redux 学习指南



### 设计思想

Redux 的设计思想很简单，就两句话。

> （1）Web 应用是一个状态机，视图与状态是一一对应的。
>
> （2）所有的状态，保存在一个对象里面。

请务必记住这两句话，下面就是详细解释。



### Redux 三大核心

1. 单一数据源
2. State是只读的
3. 使用纯函数来执行修改 





**store、components、actionCreaters、reducers的关系即为：**

- 首先有一个组件，组件要去获取store中的一些数据
- actionCreaters通过dispatch(action)方法  让store知道 组件要获取数据
- store在reducer查组件需要什么数据，reducer返回组件应该拿到的数据
- store获得数据后把数据 返给 组件





## Redux 核心

### state 状态

- DomainState：服务器返回的State
- UI State ： 关于当前组件的State
- App State：全局的State

`Store`对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。

当前时刻的 State，可以通过`store.getState()`拿到。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(fn);
> 
> const state = store.getState();
> ```

Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。



### Action 事件对象

- 本质就是一个JS 对象
- 必须包含Type属性
- 只是描述了有事情要发生，并没有描述如何去更新State

State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。

Action 是一个对象。其中的`type`属性是必须的，表示 Action 的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/acdlite/flux-standard-action)可以参考。

> ```javascript
> const action = {
>   type: 'ADD_TODO',
>   payload: 'Learn Redux'
> };
> ```

上面代码中，Action 的名称是`ADD_TODO`，它携带的信息是字符串`Learn Redux`。

可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。



### Action Creator

View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。

> ```javascript
> const ADD_TODO = '添加 TODO';
> 
> function addTodo(text) {
>   return {
>     type: ADD_TODO,
>     text
>   }
> }
> 
> const action = addTodo('Learn Redux');
> ```

上面代码中，`addTodo`函数就是一个 Action Creator。



### Store

- 用来把 action 喝 reducer 关联到一起
- 通过createStore 来构建 store
- 通过 subscribe 来注册监听
- 通过 dispatch 来发送 action

Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。

Redux 提供`createStore`这个函数，用来生成 Store。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(fn);
> ```

上面代码中，`createStore`函数接受另一个函数作为参数，返回新生成的 Store 对象。



###  store.dispatch()

`store.dispatch()`是 View 发出 Action 的唯一方法。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(fn);
> 
> store.dispatch({
>   type: 'ADD_TODO',
>   payload: 'Learn Redux'
> });
> ```

上面代码中，`store.dispatch`接受一个 Action 对象作为参数，将它发送出去。

结合 Action Creator，这段代码可以改写如下。

> ```javascript
> store.dispatch(addTodo('Learn Redux'));
> ```





### Reducer

- 本质就是一个函数
- 响应发送过来的 action
- 函数接收两个参数，第一个是初始化 state，第二个是发送过来的action
- 必须要有 return 返回值

**Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。** 

Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

> ```javascript
> const reducer = function (state, action) {
>   // ...
>   return new_state;
> };
> ```

整个应用的初始状态，可以作为 State 的默认值。下面是一个实际的例子。

> ```javascript
> const defaultState = 0;
> const reducer = (state = defaultState, action) => {
>   switch (action.type) {
>     case 'ADD':
>       return state + action.payload;
>     default: 
>       return state;
>   }
> };
> 
> const state = reducer(1, {
>   type: 'ADD',
>   payload: 2
> });
> ```

上面代码中，`reducer`函数收到名为`ADD`的 Action 以后，就返回一个新的 State，作为加法的计算结果。其他运算的逻辑（比如减法），也可以根据 Action 的不同来实现。

实际应用中，Reducer 函数不用像上面这样手动调用，`store.dispatch`方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入`createStore`方法。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(reducer);
> ```

上面代码中，`createStore`接受 Reducer 作为参数，生成一个新的 Store。以后每当`store.dispatch`发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

为什么这个函数叫做 Reducer 呢？因为它可以作为数组的`reduce`方法的参数。请看下面的例子，一系列 Action 对象按照顺序作为一个数组。

> ```javascript
> const actions = [
>   { type: 'ADD', payload: 0 },
>   { type: 'ADD', payload: 1 },
>   { type: 'ADD', payload: 2 }
> ];
> 
> const total = actions.reduce(reducer, 0); // 3
> ```

上面代码中，数组`actions`表示依次有三个 Action，分别是加`0`、加`1`和加`2`。数组的`reduce`方法接受 Reducer 函数作为参数，就可以直接得到最终的状态`3`。



### 纯函数

由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法。

> ```javascript
> // State 是一个对象
> function reducer(state, action) {
>   return Object.assign({}, state, { thingToChange });
>   // 或者
>   return { ...state, ...newState };
> }
> 
> // State 是一个数组
> function reducer(state, action) {
>   return [...state, newItem];
> }
> ```

最好把 State 对象设成只读。你没法改变它，要得到新的 State，唯一办法就是生成一个新对象。这样的好处是，任何时候，与某个 View 对应的 State 总是一个不变的对象。



### store.subscribe()

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(reducer);
> 
> store.subscribe(listener);
> ```

显然，只要把 View 的更新函数（对于 React 项目，就是组件的`render`方法或`setState`方法）放入`listen`，就会实现 View 的自动渲染。

`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。

> ```javascript
> let unsubscribe = store.subscribe(() =>
>   console.log(store.getState())
> );
> 
> unsubscribe();
> ```



## 工作流程

首先，用户发出 Action。

> ```javascript
> store.dispatch(action);
> ```

然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。 Reducer 会返回新的 State 。

> ```javascript
> let nextState = todoApp(previousState, action);
> ```

State 一旦有变化，Store 就会调用监听函数。

> ```javascript
> // 设置监听函数
> store.subscribe(listener);
> ```

`listener`可以通过`store.getState()`得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。

> ```javascript
> function listerner() {
>   let newState = store.getState();
>   component.setState(newState);   
> }
> ```







## Store 的实现

上一节介绍了 Redux 涉及的基本概念，可以发现 Store 提供了三个方法。

> - store.getState()
> - store.dispatch()
> - store.subscribe()

> ```javascript
> import { createStore } from 'redux';
> let { subscribe, dispatch, getState } = createStore(reducer);
> ```

`createStore`方法还可以接受第二个参数，表示 State 的最初状态。这通常是服务器给出的。

> ```javascript
> let store = createStore(todoApp, window.STATE_FROM_SERVER)
> ```

上面代码中，`window.STATE_FROM_SERVER`就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。

下面是`createStore`方法的一个简单实现，可以了解一下 Store 是怎么生成的。

> ```javascript
> const createStore = (reducer) => {
>   let state;
>   let listeners = [];
> 
>   const getState = () => state;
> 
>   const dispatch = (action) => {
>     state = reducer(state, action);
>     listeners.forEach(listener => listener());
>   };
> 
>   const subscribe = (listener) => {
>     listeners.push(listener);
>     return () => {
>       listeners = listeners.filter(l => l !== listener);
>     }
>   };
> 
>   dispatch({});
> 
>   return { getState, dispatch, subscribe };
> };
> ```







## Redux 简单 demo

**src/action/index.js:** 

~~~js
const ADD_TODO = "ADD_TODO";

export function addTodo(text) {
  return {
    type: ADD_TODO,
    text,
  };
}
~~~



**src/reducer/index.js**

~~~js
const demoInitialState = {
    value: '默认值'
}
const reducer = (state = demoInitialState, action) => {
    console.log(state, action);
    switch (action.type) {
        case 'ADD_TODO':
            return {
                ...state, ...action
            }
            default:
                return state
    }
}

module.exports = {
    reducer
}
~~~



**src/store/index.js：** 

~~~js
import { createStore } from "redux";

// 倒入我们自己创建的 reducer

import { reducer } from "../reducer";

// 构建 store
const store = createStore(reducer);

export default store;
~~~



**Home.jsx:**

~~~js
import React, { useEffect } from "react";

//导入 store
import store from "../store";

// 导入 action
import { addTodo } from "../action";

const Home = () => {
  const handleClick = () => {
    store.dispatch(addTodo("11111"));
  };

  useEffect(() => {
    store.subscribe(() => {
      console.log(store.getState());
    });
  });

  return (
    <div>
      <button onClick={handleClick}>Home 按钮</button>
    </div>
  );
};

export default Home;
~~~



























