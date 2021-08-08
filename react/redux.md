# Redux

### 本文速览：

- Redux 概述
- Redux 基本概念和 API
- 工作流程
- Store 的实现
- Redux 简单 Demo
- Redux 源码剖析

------



## 你可能不需要 Redux

首先明确一点，Redux 是一个有用的架构，但不是非用不可。事实上，大多数情况，你可以不用它，只用 React 就够了。

曾经有人说过这样一句话。

> "如果你不知道是否需要 Redux，那就是不需要它。"

Redux 的创造者 Dan Abramov 又补充了一句。

> "只有遇到 React 实在解决不了的问题，你才需要 Redux 。"

简单说，如果你的UI层非常简单，没有很多互动，Redux 就是不必要的，用了反而增加复杂性。

> - 用户的使用方式非常简单
> - 用户之间没有协作
> - 不需要与服务器大量交互，也没有使用 WebSocket
> - 视图层（View）只从单一来源获取数据

上面这些情况，都不需要使用 Redux。

> - 用户的使用方式复杂
> - 不同身份的用户有不同的使用方式（比如普通用户和管理员）
> - 多个用户之间可以协作
> - 与服务器大量交互，或者使用了WebSocket
> - View要从多个来源获取数据

上面这些情况才是 Redux 的适用场景：多交互、多数据源。

从组件角度看，如果你的应用有以下场景，可以考虑使用 Redux。

> - 某个组件的状态，需要共享
> - 某个状态需要在任何地方都可以拿到
> - 一个组件需要改变全局状态
> - 一个组件需要改变另一个组件的状态





## 一、Redux 概述

> redux是一个 JavaScript容器，提供可预测化的状态管理，用于进行全局的状态管理。
>
> [官网](https://redux.js.org/)



### 设计思想

Redux 的设计思想很简单，就两句话。

> （1）Web 应用是一个状态机，视图与状态是一一对应的。
>
> （2）所有的状态，保存在一个对象里面。



### Redux设计和使用的三个原则：

- **单一数据源头**
  - 整个应用的 [state](https://www.redux.org.cn/docs/Glossary.html#state) 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 [store](https://www.redux.org.cn/docs/Glossary.html#store) 中。
- **State是只读的** 
  - 唯一改变 state 的方法就是触发 [action](https://www.redux.org.cn/docs/Glossary.html#action)，action 是一个用于描述已发生事件的普通对象。
- **使用纯函数来执行修改**
  - 为了描述 action 如何改变 state tree ，你需要编写 [reducers](https://www.redux.org.cn/docs/Glossary.html#reducer)。

![image.png](https://i.loli.net/2021/08/08/6wRoJ7UdaOnkzFK.png)





## 二、Redux 基本概念和 API

### Store

> Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。

Redux 提供 `createStore` 这个函数，用来生成 Store。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(fn);
> ```

上面代码中， `createStore` 函数接受另一个函数作为参数，返回新生成的 Store 对象。



###  state 状态

- DomainState：服务器返回的 State
- UI State ： 关于当前组件的 State
- App State：全局的 State

`Store` 对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。

当前时刻的 State，可以通过 `store.getState()` 拿到。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(fn);
> 
> const state = store.getState(); // 拿到快照state状态
> ```

Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。



### Action 事件对象

- 本质就是一个 JS 对象
- 必须包含 type 属性（表示 Action 的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/acdlite/flux-standard-action)可以参考。）
- 只是描述了有事情要发生，并没有描述如何去更新 State

State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。

Action 是一个对象。其中的 `type` 属性是必须的，表示 Action 的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/acdlite/flux-standard-action)可以参考。

> ```javascript
> const action = {
>   type: 'ADD_TODO',
>   payload: 'Learn Redux'
> };
> ```

上面代码中，Action 的名称是 `ADD_TODO` ，它携带的信息是字符串 `Learn Redux` 。

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

上面代码中， `addTodo` 函数就是一个 Action Creator。



###  store.dispatch()

`store.dispatch()` 是 View 发出 Action 的唯一方法。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(fn);
> 
> store.dispatch({
>   type: 'ADD_TODO',
>   payload: 'Learn Redux'
> });
> ```

上面代码中， `store.dispatch` 接受一个 Action 对象作为参数，将它发送出去。

结合 Action Creator，这段代码可以改写如下。

> ```javascript
> store.dispatch(addTodo('Learn Redux'));
> ```



### Reducer

- 本质就是一个函数
- 响应发送过来的 action
- 函数接收两个参数，第一个是初始化 state，第二个是发送过来的 action
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

上面代码中， `reducer` 函数收到名为 `ADD` 的 Action 以后，就返回一个新的 State，作为加法的计算结果。其他运算的逻辑（比如减法），也可以根据 Action 的不同来实现。

实际应用中，Reducer 函数不用像上面这样手动调用， `store.dispatch` 方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入 `createStore` 方法。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(reducer);
> ```

上面代码中， `createStore` 接受 Reducer 作为参数，生成一个新的 Store。以后每当 `store.dispatch` 发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

为什么这个函数叫做 Reducer 呢？因为它可以作为数组的 `reduce` 方法的参数。请看下面的例子，一系列 Action 对象按照顺序作为一个数组。

> ```javascript
> const actions = [
>   { type: 'ADD', payload: 0 },
>   { type: 'ADD', payload: 1 },
>   { type: 'ADD', payload: 2 }
> ];
> 
> const total = actions.reduce(reducer, 0); // 3
> ```

上面代码中，数组 `actions` 表示依次有三个 Action，分别是加 `0` 、加 `1` 和加 `2` 。数组的 `reduce` 方法接受 Reducer 函数作为参数，就可以直接得到最终的状态 `3` 。



### 纯函数

由于 Reducer 是纯函数，就可以保证同样的 State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法。

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

Store 允许使用 `store.subscribe` 方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

> ```javascript
> import { createStore } from 'redux';
> const store = createStore(reducer);
> 
> store.subscribe(listener);
> ```

显然，只要把 View 的更新函数（对于 React 项目，就是组件的 `render` 方法或 `setState` 方法）放入 `listen` ，就会实现 View 的自动渲染。

`store.subscribe` 方法返回一个函数，调用这个函数就可以解除监听。

> ```javascript
> let unsubscribe = store.subscribe(() =>
>   console.log(store.getState())
> );
> 
> unsubscribe();
> ```



## 三、工作流程

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

`listener` 可以通过 `store.getState()` 得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。

> ```javascript
> function listerner() {
>   let newState = store.getState();
>   component.setState(newState);   
> }
> ```



## 四、Store 的实现

上一节介绍了 Redux 涉及的基本概念，可以发现 Store 提供了三个方法。

> - store.getState()
> - store.dispatch()
> - store.subscribe()

> ```javascript
> import { createStore } from 'redux';
> let { subscribe, dispatch, getState } = createStore(reducer);
> ```

`createStore` 方法还可以接受第二个参数，表示 State 的最初状态。这通常是服务器给出的。

> ```javascript
> let store = createStore(todoApp, window.STATE_FROM_SERVER)
> ```

上面代码中， `window.STATE_FROM_SERVER` 就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。

下面是 `createStore` 方法的一个简单实现，可以了解一下 Store 是怎么生成的。

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



## 五、Redux 简单 Demo

**官方demo：**

~~~js
import { createStore } from 'redux'

/**
 * This is a reducer - a function that takes a current state value and an
 * action object describing "what happened", and returns a new state value.
 * A reducer's function signature is: (state, action) => newState
 *
 * The Redux state should contain only plain JS objects, arrays, and primitives.
 * The root state value is usually an object.  It's important that you should
 * not mutate the state object, but return a new object if the state changes.
 *
 * You can use any conditional logic you want in a reducer. In this example,
 * we use a switch statement, but it's not required.
 */
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'counter/incremented':
      return { value: state.value + 1 }
    case 'counter/decremented':
      return { value: state.value - 1 }
    default:
      return state
  }
}

// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counterReducer)

// You can use subscribe() to update the UI in response to state changes.
// Normally you'd use a view binding library (e.g. React Redux) rather than subscribe() directly.
// There may be additional use cases where it's helpful to subscribe as well.

store.subscribe(() => console.log(store.getState()))

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'counter/incremented' })
// {value: 1}
store.dispatch({ type: 'counter/incremented' })
// {value: 2}
store.dispatch({ type: 'counter/decremented' })
// {value: 1}
~~~



**自己try：** 

**src/action/index.jsx:**

```js
const ADD_TODO = "ADD_TODO";

export function addTodo(text) {
  return {
    type: ADD_TODO,
    text,
  };
}
```

**src/reducer/index.jsx**

```js
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
```

**src/store/index.jsx：**

```js
import { createStore } from "redux";

// 倒入我们自己创建的 reducer

import { reducer } from "../reducer";

// 构建 store
const store = createStore(reducer);

export default store;
```

**Home.jsx:**

```js
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
  },[]);

  return (
    <div>
      <button onClick={handleClick}>Home 按钮</button>
    </div>
  );
};

export default Home;
```

![image.png](https://i.loli.net/2021/08/08/X2hj9G84yHmrbwY.png)









## 六、Redux 源码学习

**redux需要提供的功能是：**

- 创建store，即：`createStore()`
- 创建出来的store提供`subscribe`，`dispatch`，`getState`这些方法。
- 将多个reducer合并为一个reducer，即：`combineReducers()`
- 应用中间件，即`applyMiddleware()`

**redux的源码目录：**

![image.png](https://i.loli.net/2021/08/08/hFEynS1DeCpVwI5.png)

工具方法：`compose`,`bindActionCreators`

主要剖析（main）：`createStore`、`combineReducers`、`applyMiddleware`、`compose`的源码实现。



### createStore

> 这个函数的大致结构是这样：

```js
function createStore(reducer, preloadedState, enhancer) {
    if(enhancer是有效的){ 
        return enhancer(createStore)(reducer, preloadedState)
    } 
    
    let currentReducer = reducer // 当前store中的reducer
    let currentState = preloadedState // 当前store中存储的状态
    let currentListeners = [] // 当前store中放置的监听函数
    let nextListeners = currentListeners // 下一次dispatch时的监听函数
    // 注意：当我们新添加一个监听函数时，只会在下一次dispatch的时候生效。
    
    //...
    
    // 获取state
    function getState() {
        //...
      return currentState
    }
    
    // 添加一个监听函数，每当dispatch被调用的时候都会执行这个监听函数
    function subscribe() {
        //...
        nextListeners.push(listener)
    }
    
    // 触发了一个action，因此我们调用reducer，得到的新的state，并且执行所有添加到store中的监听函数。
    function dispatch() {
        //...
       currentState = currentReducer(currentState, action)
    }
   
    //...
    
    //dispatch一个用于初始化的action，相当于调用一次reducer
    //然后将reducer中的子reducer的初始值也获取到
    //详见下面reducer的实现。
    
    
    return {
        dispatch,
        subscribe,
        getState,
        //下面两个是主要面向库开发者的方法，暂时先忽略
        //replaceReducer,
        //observable
    }
}
```

可以看出，createStore方法创建了一个store，但是并没有直接将这个store的状态state返回，而是返回了一系列方法，外部可以通过这些方法（getState）获取state，或者间接地（通过调用dispatch）改变state。



### getState

```js
function getState() {
    return currentState
}
```

其实这很像面向对象编程中封装只读属性的方法，只提供数据的getter方法，而不直接提供setter。（虽然这里返回的是一个state的引用，你可以直接修改state，但是一般来说，redux不建议这样做。）



### subscribe

```js
function subscribe(listener) {
    // 添加到监听函数数组，
    // 注意：我们添加到了下一次dispatch时才会生效的数组
    nextListeners.push(listener)
    
    let isSubscribe = true //设置一个标志，标志该监听器已经订阅了
    // 返回取消订阅的函数，即从数组中删除该监听函数
    return function unsubscribe() {
        if(!isSubscribe) {
            return // 如果已经取消订阅过了，直接返回
        }
        
        isSubscribe = false
        // 从下一轮的监听函数数组（用于下一次dispatch）中删除这个监听器。
        const index = nextListeners.indexOf(listener)
        nextListeners.splice(index, 1)
    }
}
```

subscribe返回的是一个取消订阅的方法。取消订阅是非常必要的，当添加的监听器没用了之后，应该从store中清理掉。不然每次dispatch都会调用这个没用的监听器。



### dispatch

```js
function dispatch(action) {
    //调用reducer，得到新state
    currentState = currentReducer(currentState, action);
    
    //更新监听数组
    currentListener = nextListener;
    //调用监听数组中的所有监听函数
    for(let i = 0; i < currentListener.length; i++) {
        const listener = currentListener[i];
        listener();
    }
}
```

createStore这个方法的基本功能我们已经实现了，但是调用createStore方法需要提供reducer，让我们来思考一下reducer的作用。



### combineReducers

在理解combineReducers之前，我们先来想想reducer的功能：reducer接受一个旧的状态和一个action，当这个action被触发的时候，reducer处理后返回一个新状态。

也就是说 ，reducer负责状态的管理（或者说更新）。在实际使用中，我们应用的状态是可以分成很多个模块的，比如一个典型社交网站的状态可以分为：用户个人信息，好友列表，消息列表等模块。理论上，我们可以用一个reducer去处理所有状态的维护，但是这样做的话，我们一个reducer函数的逻辑就会太多，容易产生混乱。

因此我们可以将逻辑（reducer）也按照模块划分，每个模块再细分成各个子模块，开发完每个模块的逻辑后，再将reducer合并起来，这样我们的逻辑就能很清晰的组合起来。

对于我们的这种需求，redux提供了combineReducers方法，可以把子reducer合并成一个总的reducer。

来看看redux源码中combineReducers的主要逻辑：

```js
function combineReducers(reducers) {
    //先获取传入reducers对象的所有key
    const reducerKeys = Object.keys(reducers)
    const finalReducers = {} // 最后真正有效的reducer存在这里
    
    //下面从reducers中筛选出有效的reducer
    for(let i = 0; i < reducerKeys.length; i++){
        const key  = reducerKeys[i]
        
        if(typeof reducers[key] === 'function') {
            finalReducers[key] = reducers[key] 
        }
    }
    const finalReducerKeys = Object.keys(finalReducers);
    
    //这里assertReducerShape函数做的事情是：
    // 检查finalReducer中的reducer接受一个初始action或一个未知的action时，是否依旧能够返回有效的值。
    let shapeAssertionError
  	try {
    	assertReducerShape(finalReducers)
  	} catch (e) {
    	shapeAssertionError = e
  	}
    
    //返回合并后的reducer
    return function combination(state= {}, action){
  		//这里的逻辑是：
    	//取得每个子reducer对应的state，与action一起作为参数给每个子reducer执行。
    	let hasChanged = false //标志state是否有变化
        let nextState = {}
        for(let i = 0; i < finalReducerKeys.length; i++) {
                    //得到本次循环的子reducer
            const key = finalReducerKeys[i]
            const reducer = finalReducers[key]
            //得到该子reducer对应的旧状态
            const previousStateForKey = state[key]
            //调用子reducer得到新状态
            const nextStateForKey = reducer(previousStateForKey, action)
            //存到nextState中（总的状态）
            nextState[key] = nextStateForKey
            //到这里时有一个问题:
            //就是如果子reducer不能处理该action，那么会返回previousStateForKey
            //也就是旧状态，当所有状态都没改变时，我们直接返回之前的state就可以了。
            hasChanged = hasChanged || previousStateForKey !== nextStateForKey
        }
        return hasChanged ? nextState : state
    }
} 
```



### 为什么需要中间件

在redux的设计思想中，reducer应该是一个纯函数

维基百科关于纯函数的定义：

> 在[程序设计](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)中，若一个[函数](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E5%AD%90%E7%A8%8B%E5%BA%8F%23%E5%87%BD%E6%95%B8)符合以下要求，则它可能被认为是**纯函数**：
>
> - 此函数在相同的输入值时，需产生相同的输出。函数的输出和输入值以外的其他隐藏信息或[状态](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fw%2Findex.php%3Ftitle%3D%E7%A8%8B%E5%BC%8F%E7%8B%80%E6%85%8B%26action%3Dedit%26redlink%3D1)无关，也和由[I/O](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2FI%2FO)设备产生的外部输出无关。
> - 该函数不能有语义上可观察的[函数副作用](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E5%87%BD%E6%95%B0%E5%89%AF%E4%BD%9C%E7%94%A8)，诸如“触发事件”，使输出设备输出，或更改输出值以外物件的内容等。
>
> 纯函数的输出可以不用和所有的输入值有关，甚至可以和所有的输入值都无关。但纯函数的输出不能和输入值以外的任何资讯有关。纯函数可以传回多个输出值，但上述的原则需针对所有输出值都要成立。若引数是[传引用调用](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E6%B1%82%E5%80%BC%E7%AD%96%E7%95%A5%23%E4%BC%A0%E5%BC%95%E7%94%A8%E8%B0%83%E7%94%A8)，若有对参数物件的更改，就会影响函数以外物件的内容，因此就不是纯函数。

总结一下，纯函数的重点在于：

- 相同的输入产生相同的输出（不能在内部使用`Math.random`,`Date.now`这些方法影响输出）
- 输出不能和输入值以外的任何东西有关（不能调用API获得其他数据）
- 函数内部不能影响函数外部的任何东西（不能直接改变传入的引用变量），即不会突变

reducer为什么要求使用纯函数，文档里也有提到，总结下来有这几点：

- state是根据reducer创建出来的，所以reducer是和state紧密相关的，对于state，我们有时候需要有一些需求（比如打印每一次更新前后的state，或者回到某一次更新前的state）这就对reducer有一些要求。
- 纯函数更易于调试
  - 比如我们调试时希望action和对应的新旧state能够被打印出来，如果新state是在旧state上修改的，即使用同一个引用，那么就不能打印出新旧两种状态了。
  - 如果函数的输出具有随机性，或者依赖外部的任何东西，都会让我们调试时很难定位问题。
- 如果不使用纯函数，那么在比较新旧状态对应的两个对象时，我们就不得不深比较了，深比较是非常浪费性能的。相反的，如果对于所有可能被修改的对象（比如reducer被调用了一次，传入的state就可能被改变），我们都新建一个对象并赋值，两个对象有不同的地址。那么浅比较就可以了。

**至此，我们已经知道了，reducer是一个纯函数，那么如果我们在应用中确实需要处理一些副作用（比如异步处理，调用API等操作），那么该怎么办呢？这就是中间件解决的问题。下面我们就来讲讲redux中的中间件。**



### 中间件处理副作用的机制

中间件在redux中位于什么位置，我们可以通过这两张图来看一下。

先来看看不用中间件时的redux工作流程：



![redux工作流程_同步](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/10/165c24bf8ff7f1aa~tplv-t2oaga2asx-watermark.awebp)



1. dispatch一个action（纯对象格式）
2. 这个action被reducer处理
3. reducer根据action更新store（中的state）

而用了中间件之后的工作流程是这样的：



![redux工作流程_中间件](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/10/165c24bf96cbce24~tplv-t2oaga2asx-watermark.awebp)



1. dispatch一个“action”（不一定是标准的action）
2. 这个“action”先被中间件处理（比如在这里发送一个异步请求）
3. 中间件处理结束后，再发送一个"action"（有可能是原来的action，也可能是不同的action因中间件功能不同而不同）
4. 中间件发出的"action"可能继续被另一个中间件处理，进行类似3的步骤。即中间件可以链式串联。
5. 最后一个中间件处理完后，dispatch一个符合reducer处理标准的action（纯对象action）
6. 这个标准的action被reducer处理，
7. reducer根据action更新store（中的state）

那么中间件该如何融合到redux中呢？

在上面的流程中，2-4的步骤是关于中间件的，但凡我们想要添加一个中间件，我们就需要写一套2-4的逻辑。

如果我们需要多个中间件，我们就需要考虑如何让他们串联起来。如果每次串联都写一份串联逻辑的话，就不够灵活，万一需要增删改或调整中间件的顺序，都需要修改中间件串联的逻辑。

所以redux提供了一种解决方案，将中间件的串联操作进行了封装，经过封装后，上面的步骤2-5就可以成为一个整体，如下图：



![封装中间件后的逻辑](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/10/165c24bf9ca1fbe5~tplv-t2oaga2asx-watermark.awebp)



我们只需要改造store自带的dispatch方法。action发生后，先给中间件处理，最后再dispatch一个action交给reducer去改变状态。

### 在redux中使用中间件

还记得redux 的`createStore()`方法的第三个参数`enhancer`吗：

```js
function createStore(reducer, preloadedState, enhancer) {
    if(enhancer是有效的){  
        return enhancer(createStore)(reducer, preloadedState)
    } 
    
    //...
}
```

在这里，我们可以看到，enhancer（可以叫做强化器）是一个函数，这个函数接受一个「普通createStore函数」作为参数，返回一个「加强后的createStore函数」。

这个加强的过程中做的事情，其实就是改造dispatch，添加上中间件。

redux提供的`applyMiddleware()`方法返回的就是一个enhancer。

applyMiddleware，顾名思义，「应用中间件」。输入为若干中间件，输出为enhancer。下面来看看它的源码：

```js
function applyMiddleware(...middlewares) {
    // 返回一个函数A，函数A的参数是一个createStore函数。
    // 函数A的返回值是函数B，其实也就是一个加强后的createStore函数，大括号内的是函数B的函数体
    return createStore => (...args) => {
        //用参数传进来的createStore创建一个store
        const store  = createStore(...args)
        //注意，我们在这里需要改造的只是store的dispatch方法
        
        let dispatch = () => {  //一个临时的dispatch
            					//作用是在dispatch改造完成前调用dispatch只会打印错误信息
            throw new Error(`一些错误信息`)
        } 
        //接下来我们准备将每个中间件与我们的state关联起来（通过传入getState方法），得到改造函数。
        const middlewareAPI = {
            getState: store.getState,
            dispatch: (...args) => dispatch(...args)
        }
        //middlewares是一个中间件函数数组，中间件函数的返回值是一个改造dispatch的函数
        //调用数组中的每个中间件函数，得到所有的改造函数
        const chain = middlewares.map(middleware => middleware(middlewareAPI))
        
        //将这些改造函数compose（翻译：构成，整理成）成一个函数
        //用compose后的函数去改造store的dispatch
        dispatch = compose(...chain)(store.dispatch)
        // compose方法的作用是，例如这样调用：
        // compose(func1,func2,func3)
        // 返回一个函数: (...args) => func1( func2( func3(...args) ) )
        // 即传入的dispatch被func3改造后得到一个新的dispatch，新的dispatch继续被func2改造...
        
        // 返回store，用改造后的dispatch方法替换store中的dispatch
        return {
            ...store,
            dispatch
        }
    }
}
```

总结一下，applyMiddleware的工作方式是：

1. 调用（若干个）中间件函数，获取（若干个）改造函数
2. 把所有改造函数compose成一个改造函数
3. 改造dispatch方法

中间件的工作方式是：

- 中间件是一个函数，不妨叫做中间件函数
- 中间件函数的输入是store的`getState`和`dispatch`，输出为改造函数（改造`dispatch`的函数）
- 改造函数输入是一个`dispatch`，输出「改造后的`dispatch`」

源码中用到了一个很有用的方法：`compose()`，将多个函数组合成一个函数。理解这个函数对理解中间件很有帮助，我们来看看它的源码：

```js
function compose(...funcs) {
    // 当未传入函数时，返回一个函数：arg => arg
    if(funcs.length === 0) {
        return arg => arg
    }
    
    // 当只传入一个函数时，直接返回这个函数
    if(funcs.length === 1) {
        return funcs[0]
    }
    
    // 返回组合后的函数
    return funcs.reduce((a, b) => (...args) => a(b(...args)))
    
    //reduce是js的Array对象的内置方法
    //array.reduce(callback)的作用是：给array中每一个元素应用callback函数
    //callback函数：
    /*
     *@参数{accumulator}：callback上一次调用的返回值
     *@参数{value}：当前数组元素
     *@参数{index}：可选，当前元素的索引
     *@参数{array}：可选，当前数组
     *
     *callback( accumulator, value, [index], [array])
    */
}
```

画一张图来理解compose的作用：



![compose](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/13/165d2045cd9a75a5~tplv-t2oaga2asx-watermark.awebp)



在applyMiddleware方法中，我们传入的「参数」是原始的dispatch方法，返回的「结果」是改造后的dispatch方法。通过compose，我们可以让多个改造函数**抽象成**一个改造函数。

### 中间件的实现

> 作者注：本来只想讲redux，但是讲着讲着却发现：理解中间件，是理解redux的中间件机制的前提。

下面我们以redux-thunk为例，看看一个中间件是如何实现的。

##### redux-thunk的功能

你可能没用过redux-thunk，所以在阅读源码前，我先简要的讲一下redux-thunk的作用：

正常的dispatch函数的参数action应该是一个纯对象。像这样：

```js
store.dispatch({
    type:'REQUEST_SOME_THING',
    payload: {
        from:'bob',
    }
})
```

使用了thunk之后，我们可以dispatch一个函数:

```js
function logStateInOneSecond(name) {
    return (dispatch, getState, name) => {  // 这个函数会在合适的时候dispatch一个真正的action
        setTimeout({
            console.log(getState())
            dispatch({
                type:'LOG_OK',
                payload: {
                    name,
                }
            })
        }, 1000)
    }
}

store.dispatch(logStateInOneSecond('jay')) //dispatch的参数是一个函数
```

为什么需要这个功能？或者说「dispatch一个函数」能解决什么问题？

从上面的例子中你会发现，如果dispatch一个函数，我们可以在这个函数内做任何我们想要的操作（异步处理，调用接口等等），不受任何限制，为什么？

因为我们「还没有dispatch一个真正的action」，所以不会调用reducer，**我们并没有将副作用放在reducer中，而是在使用reducer之前就处理了副作用**。

> 如果你还不明白redux-thunk的功能，可以去[它的github仓库](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Freduxjs%2Fredux-thunk%23motivation)查看更详细的解释。

##### redux-thunk的实现

如何实现redux-thunk中间件呢？

首先中间件肯定是改造dispatch方法，改造后的dispatch应该具有这样的功能：

1. 如果传入的参数是函数，就执行这个函数。
2. 否则，就认为传入的是一个标准的action，就调用「改造前的dispatch」方法，dispatch这个action。

现在我们来看看redux-thunk的源码（8行有效代码）：

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}
```

如果三个箭头函数让你有点头晕，我来帮你展开一下：

```js
//createThunkMiddleware的作用是返回thunk中间件（middleware）
function createThunkMiddleware(extraArgument) {
    
    return function({ dispatch, getState }) { // 这是「中间件函数」
        
        return function(next) { // 这是中间件函数创建的「改造函数」
            
            return function(action) { // 这是改造函数改造后的「dispatch方法」
                if (typeof action === 'function') {
                  return action(dispatch, getState, extraArgument);
                }
                
                return next(action);
            }
        } 
    }
}
```

再加点注释？

```js
function createThunkMiddleware(extraArgument) {
    
    return function({ dispatch, getState }) { // 这是「中间件函数」
        //参数是store中的dispatch和getState方法
        
        return function(next) { // 这是中间件函数创建的「改造函数」
            //参数next是被当前中间件改造前的dispatch
            //因为在被当前中间件改造之前，可能已经被其他中间件改造过了，所以不妨叫next
            
            return function(action) { // 这是改造函数「改造后的dispatch方法」
                if (typeof action === 'function') {
                  //如果action是一个函数，就调用这个函数，并传入参数给函数使用
                  return action(dispatch, getState, extraArgument);
                }
                
                //否则调用用改造前的dispatch方法
                return next(action);
            }
        } 
    }
}
```

可以看出redux-thunk严格遵循了redux中间件的思想：在原始的dispatch方法触发reducer处理之前，处理副作用。



