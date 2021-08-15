# Refs 转发

### 本文速览：

- Refs 基本介绍 & 创建方式 & ref 的值类型 【Refs and the DOM】
- Refs 转发 （ React.fowardRef ）
- `useRef`  & `useImperativeHandle`



------





# Refs and the DOM

> Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素
>
> **简单来说** — 进行 DOM 操作 – 官方建议不要优先使用 Ref，而考虑使用 state

![image.png](https://i.loli.net/2021/08/15/ACtgsuN1VhwQbD8.png)

### 何时使用 Refs

下面是几个适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

**注意**  ⚠️   ------ 避免使用 refs 来做任何可以通过声明式实现来完成的事情。

举个例子，避免在 `Dialog` 组件里暴露 `open()` 和 `close()` 方法，最好传递 `isOpen` 属性。



### 创建方式

**React 提供 3 种方式进行 ref 使用：**

1. 字符串的方式 【 过时 API 】
2. 回调函数  【如果是内联函数，在更新过程中它会被执行两次】

3. React.createRef ()【  react16.3 新提供的方式 】【 官方推荐 】



#### 1、String 类型的 Refs

> 如果你之前使用过 React，你可能了解过之前的 API 中的 string 类型的 ref 属性，例如 `"textInput"`。你可以通过 `this.refs.textInput` 来访问 DOM 节点。我们不建议使用它，因为 string 类型的 refs 存在 [一些问题](https://github.com/facebook/react/pull/8333#issuecomment-271648615)。它已过时并可能会在未来的版本被移除。
>
> **注意**  ⚠️
>
> 如果你目前还在使用 `this.refs.textInput` 这种方式访问 refs ，我们建议用[回调函数](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs)或 [`createRef` API](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#creating-refs) 的方式代替。

~~~js
class MyComponent extends React.Component {
  hanle(){
    console.log(this.refs.demoref);
  }
  render(){
    return(
      <div>
      <input type="text" ref ='demoref'/> 
      <button onClick = {()=> this.hanle()}>点击</button>
		</div>
		)
	}
}
~~~



#### 2、回调函数

> 回调Refs ：就是在 dom 节点上或者组件上挂载函数，**函数的形参是 dom 节点**，达到的效果和字符串是一样的，都是获取值的引用
>
> **注意**  ⚠️
>
> 如果 `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

```js
class MyComponent extends React.Component {
      hanle() {
        console.log(this.demoref);
      }
      render() {
        return (
          <div>
            <input type="text" ref={(input) => (this.demoref = input)}/>
            <button onClick={() => this.hanle()}>点击</button>
          </div>
        );
     }
}
```

**回调函数形式存在一丢丢问题** ，在官网有说明，也就是在render再次更新时，回调函数会被调用两次，第一次拿到的是 dom元素是null，第二次才会拿到dom元素。

![image.png](https://i.loli.net/2021/08/15/NLpsUzfbwB8FYCO.png)

解决这个问题，可以定义一个回调函数，将回调交给react去执行，如下

~~~js
createRef = (c) => {
    //此参数中就是所需要的元素
}
render(){
    return (
        <input ref = {this.createRef} />
    )
}
~~~



#### 3、`React.createRef()` 【官方推荐】

Refs 是使用 `React.createRef()` 创建的，并通过 `ref` 属性附加到 React 元素。在构造组件时，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```



### ref 的值类型

**访问 Refs** ：当 ref 被传递给 `render` 中的元素时，对该节点的引用可以在 ref 的 `current` 属性中被访问。

```js
const node = this.myRef.current;
```

ref 的值根据节点的类型而有所不同：

- 当 `ref` 属性用于 HTML 元素时，构造函数中使用 `React.createRef()` 创建的 `ref` 接收   **底层 DOM 元素**  作为其 `current` 属性。
- 当 `ref` 属性用于自定义 class 组件时，`ref` 对象接收 **组件的挂载实例** 作为其 `current` 属性。



> **注意**  ⚠️   -----  **你不能在函数组件上使用 `ref` 属性**，因为他们没有实例。
>
> 如果要在函数组件中使用 `ref`，你可以使用 [`forwardRef`](https://zh-hans.reactjs.org/docs/forwarding-refs.html)（可与 [`useImperativeHandle`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle) 结合使用），或者可以将该组件转化为 class 组件。
>
> 不管怎样，你可以**在函数组件内部使用 `ref` 属性**，只要它指向一个 DOM 元素或 class 组件



# Refs 转发

> **Ref 转发** 是一项将 [ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 自动地通过组件传递到其一子组件的技巧。对于大多数应用中的组件来说，这通常不是必需的。但其对某些组件，尤其是可重用的组件库是很有用的。
>
> **Ref 转发是一个可选特性，其允许某些组件接收 `ref`，并将其向下传递（换句话说，“转发”它）给子组件。**
>
> 通俗来说：**Ref 转发使父组件可以像暴露自己的 ref 一样  --  暴露子组件的 ref**。



### React.fowardRef

> **Ref forwarding是组件一个可选的特征。一个组件一旦有了这个特征，它就能接受上层组件传递下来的ref，然后顺势将它传递给自己的子组件。** 

~~~js
import React, { useEffect } from 'react'

export default function App() {
  const ref = React.createRef() // 可以直接获取 -- 子组件的DOM的ref：

  useEffect(() => {
    console.log(ref); // {current: h2}
  })

  return (
    <div>
        <Son ref = {ref}></Son>
    </div>
  )
}

const Son= React.forwardRef((props,ref)=>{
    return (
      <div className="box">
        <h1>son</h1>
        <h2 ref={ref}>哈哈哈</h2>
      </div>
    )
})
~~~

以下是对上述示例发生情况的逐步解释：

1. 我们通过调用 `React.createRef` 创建了一个 [React ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 并将其赋值给 `ref` 变量。
2. 我们通过指定 `ref` 为 JSX 属性，将其向下传递给 `<Son ref={ref}>`。
3. React 传递 `ref` 给 `forwardRef` 内函数 `(props, ref) => ...`，作为其第二个参数。
4. 我们向下转发该 `ref` 参数到 `<h2 ref={ref}>`，将其指定为 JSX 属性。
5. 当 ref 挂载完成，`ref.current` 将指向 `<h2>` DOM 节点。

> 注意
>
> 第二个参数 `ref` 只在使用 `React.forwardRef` 定义组件时存在。常规函数和 class 组件不接收 `ref` 参数，且 props 中也不存在 `ref`。
>
> Ref 转发不仅限于 DOM 组件，你也可以转发 refs 到 class 组件实例中。



### 在高阶组件转发refs

> 这个技巧对[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)（也被称为 HOC）特别有用。



如果使用了高阶组件，还是按照上面普通的方式使用的话，会导致ref直接转发到高阶组件上

**HOC 示例开始**： 

~~~js
// 父组件
import React ,{useEffect} from 'react';
import Son from './Son';
export default function App(){
  const ref = React.createRef();

  useEffect(() => {
      console.log('ref', ref.current);
  },[])

  return (
     <Son ref={ref} />
  )
}
~~~

**子组件：**

~~~js
import React from 'react'

const Son = React.forwardRef((props, ref) => (
    <button ref={ref}>按钮</button>
));

// 功能：输出组件 props 到控制台
function logProps(Component) {  
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      return <Component {...this.props} />;
    }
  }

  return LogProps;
}

export default logProps(Son) // 使用高阶组件对其进行封装
~~~

**结果**：refs 将不会透传下去。这是因为 `ref` 不是 prop 属性。就像 `key` 一样，其被 React 进行了特殊处理。如果你对 HOC 添加 ref，该 ref 将引用最外层的容器组件，而不是被包裹的组件。

![image.png](https://i.loli.net/2021/08/15/9j5IvAbfWn4BX1l.png)



幸运的是，我们可以使用 `React.forwardRef` API 明确地将 refs 转发到内部的 `Son`  组件。`React.forwardRef` 接受一个渲染函数，其接收 `props` 和 `ref` 参数并返回一个 React 节点。例如

~~~js
import React from 'react'

const Son = React.forwardRef((props, ref) => (
    <button ref={ref}>按钮</button>
));

function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // 使用 `React.forwardRef` API 明确地将 refs 转发到内部的 `Son`  组件
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}

export default logProps(Son) // 使用高阶组件对其进行封装
~~~



![image.png](https://i.loli.net/2021/08/15/VxGI8dm5ZchDYXU.png)









## useRef

> 本质上，`useRef` 就像是可以在其 `.current` 属性中保存一个可变值的“盒子”
>
> 特点： `useRef` 会在每次渲染时返回同一个 ref 对象。
>
> 缺点：当 ref 对象内容发生变化时，`useRef` 并*不会*通知你。变更 `.current` 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用[回调 ref](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node) 来实现。

### 基本使用

> ```js
> const refContainer = useRef(initialValue);
> ```

~~~js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  
  const onButtonClick = () => {
    inputEl.current.focus();
  };
  
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
~~~



### 特点

1. 函数组件访问DOM元素；
2. 函数组件访问之前渲染变量。

函数组件每次渲染都会被执行，函数内部的局部变量一般会重新创建，利用`useRef`可以访问上次渲染的变量，类似[`类组件的实例变量`](https://link.segmentfault.com/?url=https%3A%2F%2Freactjs.org%2Fdocs%2Fhooks-faq.html%23is-there-something-like-instance-variables)效果。



### 函数组件使用`createRef`不行吗？

> `createRef`主要解决`class`组件访问DOM元素问题，并且最佳实践是在组件周期内只创建一次（ **一般在构造函数里调用** ）。如果在函数组件内使用`createRef`会造成每次`render`都会调用`createRef`：

~~~js
import React, { useState, useEffect } from 'react'

export default function App() {
  const [minus, setMinus] = useState(0);
  // 每次render都会重新创建`ref`
  const ref = React.createRef(null);

  const handleClick = () => {
    setMinus(minus + 1);
  };

  // 这里每次都是`null`
  console.log(`ref.current=${ref.current}`)

  useEffect(() => {
    console.log(`denp[minus]>`, ref.current && ref.current.innerText);
  }, [minus]);

  return (
    <div className="App">
      <h1 ref={ref}>Num: {minus}</h1>
      <button onClick={handleClick}>Add</button>
    </div>
  );
}
~~~

![image.png](https://i.loli.net/2021/08/15/Ld5QPXzcoieYqEr.png)





### createRef 与 useRef 的区别

 createRef 每次渲染都会返回一个新的引用，而 useRef 每次都会返回相同的引用。



### 如何测量 DOM 节点？

> 使用` 回调 ref` ，而不是 `useRef`

获取 DOM 节点的位置或是大小的基本方式是使用 [callback ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs)。每当 ref 被附加到一个另一个节点，React 就会调用 callback。这里有一个 [小 demo](https://codesandbox.io/s/l7m0v5x4v9):

~~~js
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
~~~

在这个案例中，我们没有选择使用 `useRef`，因为当 ref 是一个对象时它并不会把当前 ref 的值的 *变化* 通知到我们。使用 callback ref 可以确保 [即便子组件延迟显示被测量的节点](https://codesandbox.io/s/818zzk8m78) (比如为了响应一次点击)，我们依然能够在父组件接收到相关的信息，以便更新测量结果。

在此示例中，当且仅当组件挂载和卸载时，callback ref 才会被调用，因为渲染的 `<h1>` 组件在整个重新渲染期间始终存在。如果你希望在每次组件调整大小时都收到通知，则可能需要使用 [`ResizeObserver`](https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver) 或基于其构建的第三方 Hook。



## useImperativeHandle

> ~~~js
> useImperativeHandle(ref, createHandle, [deps])
> ~~~
>
> 简单来说： 子组件 自定义 暴露 实例值  
>
> `useImperativeHandle` 可以让你在使用 `ref` 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。`useImperativeHandle` 应当与 [`forwardRef`](https://zh-hans.reactjs.org/docs/react-api.html#reactforwardref) 一起使用：

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

在本例中，渲染 `<FancyInput ref={inputRef} />` 的父组件可以调用 `inputRef.current.focus()`。

~~~js
import React, { useRef, forwardRef } from "react";
import { useImperativeHandle } from "react";

const Com = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({
    name: "zc",
    add:()=>{
        console.log('111');
    }
  }));

  return (
    <div>
      <p> 我是子组建</p>
    </div>
  );
});

const HookUseCallbak = () => {
  const refdemo = useRef(null);
  return (
    <div>
      <h1>213</h1>
      <Com ref={refdemo} />
      <button onClick={() => console.log(refdemo)}>点击</button>
    </div>
  );
};

export default HookUseCallbak;
~~~

![image.png](https://i.loli.net/2021/08/15/q1xk2AscFIKXbOp.png)





**参考：** 

1. https://segmentfault.com/a/1190000024536290

