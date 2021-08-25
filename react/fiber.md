# fiber

### 本文速览

- Fiber 出现的背景
- Fiber的诞生（如何解决问题？）【 requestIdleCallback的核心用法 】
- Fiber是什么 【执行单元】【数据结构】
- Fiber vs React 15 的 Stack Reconciler
- Fiber执行阶段



------





## Fiber 出现的背景

**预备知识**：JavaScript 引擎和页面渲染引擎两个线程是互斥的，**当其中一个线程执行时，另一个线程只能挂起等待**。 

在这样的机制下，如果 JavaScript 线程长时间地占用了主线程，那么渲染层面的更新就不得不长时间地等待，界面长时间不更新，会导致**页面响应度变差，用户可能会感觉到卡顿**。

而这正是 **React 15 的 Stack Reconciler** 所面临的问题，即是 JavaScript 对主线程的超时占用问题。Stack Reconciler （diff算法） 是一个**同步的递归过程**，使用的是 JavaScript 引擎自身的函数调用栈，它会**一直执行到栈空为止**，所以当 React 在渲染组件时，从开始到渲染完成整个过程是一气呵成的。如果渲染的组件比较庞大，js 执行会占据主线程较长时间，会导致页面响应度变差。

而且所有的任务都是按照先后顺序，没有区分优先级，这样就会导致优先级比较高的任务无法被优先执行。

针对这一现象，[React](https://www.w3cschool.cn/react/) 团队从框架层面对 web 页面的运行机制做了优化，此后，`Fiber`诞生了。



说到16ms，我们来看这样的一个概念

### 屏幕刷新率

- 目前大多数设备的屏幕刷新率为60次/秒
- 浏览器的渲染动画或页面的每一帧的速率也需要跟设备屏幕的刷新率保持一致。
- 页面是一帧一帧绘制出来的，当每秒绘制的帧数(FPS)达到60时，页面是流畅的，小于这个值时，用户会感觉到卡顿。
- 每个帧的预算时间是16.66毫秒（1秒/60）
- 1s 60帧，所以我们书写代码时尽量不让一帧的工作量超过16ms



## Fiber的诞生（如何解决问题？）

> 解决主线程长时间被 JS 晕眩占用这一问题的基本思路，是将`运算切割为多个步骤`（Fiber 把每一个分割得很细的任务视作一个"执行单元"），分批完成。也就是说在完成一部分任务之后， 将`控制权交回`给浏览器，让浏览器有时间再进行页面的渲染。等浏览器忙完之后，再继续之前React未完成的任务。

旧版 React `通过递归`的方式进行渲染，使用的是 JS 引擎自身的函数调用栈，**它会一直执行到栈空为止**。

而`Fiber`实现了自己的组件调用栈，它以链表的形式遍历组件树，可以灵活地暂停、继续和丢弃执行的任务。实现的方式是使用了 浏览器的`requestIdleCallback`这一 API。官方的解释是这样的：

> window.requestIdleCallback()会在浏览器`空闲时期`依次调用函数，这就可以让开发者在`主事件循环`中执行`后台`或`优先级低`的任务，而且不会像对动画和用户交互这些延迟触发产生关键的事件影响。函数一般会按先进先调用的顺序执行，除非函数在浏览器调用它之前就到了它的超时时间。

### requestIdleCallback的核心用法

- 希望快速响应用户，让用户觉得够快，不能阻塞用户的交互行为
- requestIdleCallback 使开发者能够在`主事件循环`上执行后台和`低优先级`的工作，而不会影响延迟关键事件，例如动画和输入的响应
- 正常帧任务完成后`没超过16ms`，说明时间有赋予，此时就会执行`requestIdleCallback`里注册的任务

#### requestIdleCallback执行流程

![image.png](https://i.loli.net/2021/08/24/VM4esRX3ZhDcoQy.png)





> React 16 的`Reconciler`基于 Fiber 节点实现，被称为 Fiber Reconciler。





## Fiber是什么

> Fiber 是一个执行单元，**每次执行完一个执行单元, React 就会检查现在还剩多少时间**，如果没有时间就将控制权让出去

![image.png](https://i.loli.net/2021/08/24/TaiWJQ7N3xFwsCe.png)

![image.png](https://i.loli.net/2021/08/24/kEUyWiZM3OhGjP7.png)

### Fiber Node 及 Fiber Tree

> Fiber是一种数据结构，[React](https://www.w3cschool.cn/react/) 目前的做法是使用链表, 每个 VirtualDOM 节点内部表示为一个`Fiber`，它可以用一个 JS 对象来表示

从流程图上看到会有 `Fiber Node` 节点，这个是在 react 生成的 Virtual Dom 基础上增加的一层数据结构，主要是为了将递归遍历转变成循环遍历，配合 `requestIdleCallback` API, 实现任务拆分、中断与恢复。

为了实现循环遍历，Fiber Node 上携带了更多的信息， 其数据结构如下所示：

```js
export type Fiber = {
  tag: TypeOfWork,
  key: null | string,
  type: any,

  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,

  effectTag: TypeOfSideEffect,
  nextEffect: Fiber | null,
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,

  alternate: Fiber | null,
  stateNode: any,
  ...
}
```

每一个 Fiber Node 节点与 Virtual Dom 一一对应，所有 Fiber Node 连接起来形成 Fiber tree, 是个单链表树结构，如下图所示：

![image.png](https://i.loli.net/2021/08/24/usiMcfrQbUq6KeC.png)



## Fiber vs 以前的协调阶段

> *在 Fiber 出现之前， React 会不断递归遍历虚拟 DOM 节点，占用着浏览器资源，积极地浪费性能，造成卡顿现象，且协调阶段是不能*`被打断的`*。*
>
> *Fiber 出现之后，通过某些 Fiber 调度策略合理分配 CPU 资源，让自己的*`协调阶段变成可被终端`*，*`适时`*地让 CPU（浏览器）执行权，提高了性能优化。*





## Fiber执行阶段

每次渲染有两个阶段：Reconciliation(协调/render阶段)和Commit(提交阶段)

- 协调阶段: 这个阶段`可以被中断`, 通过Dom-Diff算法找出所有节点变更，例如`节点新增`、`删除`、`属性变更`等等, 这些变更React 称之为`副作用`(Effect)
- 提交阶段: 将上一个阶段计算出来的需要处理的副作用(Effects)一次性执行了。这个阶段必须`同步`执行，`不能被打断`

简单理解的话

- 阶段1：生成Fiber树，得出需要`更新`的`节点信息`。（`可打断`）
- 阶段2：将需要更新的节点一次性地`批量更新`。（`不可打断`）

> Fiber的协调阶段，可以被优先级较高的任务（如键盘输入）打断。
>
> 阶段1可被打断的特性，让`优先级更高的任务先执行`，从框架层面大大降低了页面掉帧的概率。





## Fiber执行流程

### render阶段

Fiber Reconciliation(协调) 在阶段一进行 Diff 计算的时候，会生成一棵 `Fiber 树`。这棵树是在 `Virtual DOM 树`的基础上增加额外的信息`生成`来的，它本质来说是一个链表。





### commit提交阶段

**Fiber 树在首次渲染的时候会一次过生成**。在`后续`需要 `Diff` 的时候，会根据已有树和最新 Virtual DOM 的信息，生成一棵新的树。这颗新树每生成一个新的节点，都会将控制权交回给主线程，去检查有没有优先级更高的任务需要执行。如果没有，则继续构建树的过程。

> 1.如果过程中有优先级更高的任务需要进行，则 Fiber Reconciler 会丢弃正在生成的树，在空闲的时候再重新执行一遍。
>
> 2.在构造 Fiber 树的过程中，Fiber Reconciler 会将需要更新的节点信息保存在`Effect List`当中，在阶段二执行的时候，会批量更新相应的节点。









## 参考

- [requestIdleCallback](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)
- [React Fiber Architecture](https://link.zhihu.com/?target=https%3A//github.com/acdlite/react-fiber-architecture)
- [Update on Async Rendering](https://link.zhihu.com/?target=https%3A//reactjs.org/blog/2018/03/27/update-on-async-rendering.html)
- [Lin Clark - A Cartoon Intro to Fiber - React Conf 2017](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DZCuYPiUIONs%26t%3D489s)
- [You Probably Don't Need Derived State](https://link.zhihu.com/?target=https%3A//reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)
- [Beyond React 16 by Dan Abramov - JSConf Iceland](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3Dv6iR3Zk4oDY%26t%3D1815s)
- https://www.w3cschool.cn/article/8940ab0f69de0e.html

