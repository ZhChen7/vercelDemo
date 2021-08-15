# react-dnd拖拽库

### 本文速览

- react-dnd 基本使用
- 用 React Hooks 的方式使用 react-dnd



## 简介

React DnD 是一组 React 高阶组件，使用的时候只需要使用对应的 API 将目标组件进行包裹，即可实现拖动或接受拖动元素的功能。将拖动的事件转换成对象中对应状态的形式，不需要开发者自己判断拖动状态，只需要在传入的 spec 对象中各个状态属性中做对应处理即可。



`React Dnd` 提供了几个重要的 `API` 供我们使用：

- DragSource
- DropTarget
- DragDropContext && DragDropContextProvider



### API的基本使用





### DragSource

`DragSource` 是一个高阶组件，使用 `DragSource` 高阶组件包裹的组件可以进行拖拽操作。

**基本用法：**

```javascript
import { DragSource } from 'react-dnd'

class MyComponent {
  /* ... */
}

export default DragSource(type, spec, collect)(MyComponent)
```

**参数：**

- type：指定拖拽元素的类型，值的类型可以是 `string`、 `symbol` 或者 `func` ，只有具有相同type类型的元素才能被 `drop target` 所响应。
- spec: 一个js对象，上面定义了一些方法，用来描述 `drag source` 如何对拖动事件进行响应。
  - beginDrag(props, monitor, component): 必填项。当拖拽开始的时候，这个方法就会被调用，这个方法必须要返回一个js 对象来描述被拖拽的元素，比如返回一个 `{ id: props.id }`,通过`monitor.getItem()` 方法可以获取到返回结果。
  - endDrag(props, monitor, component): 非必填项。当拖拽停止的时候，这个方法会被调用，通过 `monitor.didDrop()` 可以判断 `drag source` 是否已经被 `drop target` 处理完毕。如果在 `drop target` 的 `drop` 方法中返回了一个对象，在这里可以通过 `monitor.getDropResult()` 获取到返回结果。
  - canDrag(props, monitor): 可选参数。可以指定当前的拖拽操作是否被允许。
  - isDragging(props, monitor): 可选参数。拖拽时触发的事件，注意，在这个方法里面不能再调用 `monitor.isDragging()`。

```autohotkey
方法中的参数解释：
- props：当前组件的 `props` 参数。
- monitor：一个 `DragSourceMonitor` 实例。通过它可以获取当前的拖拽信息，比如可以获取当前被拖拽的项目及其类型，当前和初始坐标和偏移，以及它是否已被删除。
- component：是组件的实例。使用它可以访问DOM元素来进行位置或大小测量，或调用组件里面定义的方法，或者进行 `setState` 操作。有时候在 isDragging、 canDrag 方法里可能获取不到 `component` 这个参数，因为它们被调用时实例可能不可用。
```

- collect: 必填项，把拖拽过程中需要的信息注入组件的 `props`，接收两个参数 `connect` 和 `monitor`。
  - connect: `DragSourceConnector` 的实例，包括 `dragPreview()` 和 `dragSource()` 两个方法，常用的是 `dragSource()` 这个方法。
    - dragSource: 返回一个函数，传递给组件用来将 `source DOM` 和 `React DnD Backend` 连接起来。
    - dragPreview: 返回一个函数，传递给组件用来将拖动时预览的 `DOM` 节点 和 `React DnD Backend` 连接起来。
  - monitor: `DragSourceMonitor` 的实例，包含的具体方法可以参考[这里](https://link.segmentfault.com/?url=https%3A%2F%2Freact-dnd.github.io%2Freact-dnd%2Fdocs%2Fapi%2Fdrag-source-monitor)。



### DropTarget

`DropTarget` 是一个高阶组件，被 `DropTarget` 包裹的组件能够放置拖拽组件，能够对 `hover` 或者 `dropped` 事件作出响应。

**基本用法：**

```javascript
import { DropTarget } from 'react-dnd'

class MyComponent {
  /* ... */
}

export default DropTarget(types, spec, collect)(MyComponent)
```

**参数：**

- types: 指定拖拽元素的类型，值的类型可以是 `string`、 `symbol` 或者 `array` ，`drop target` 只接受具有相同 type 类型的 `drag source`。
- spec: 一个js对象，上面定义了一些方法，描述了拖放目标对拖放事件的反应。
  - drop(props, monitor, component): 可选参数。当item被放置到目标元素上时会被调用。如果这个方法返回的是一个js对象，在 `drag source` 的 `endDrag` 方法里面，调用 `monitor.getDropResult()` 可以获得返回结果。
  - hover(props, monitor, component): 可选参数。当item经过 `drop target` 的时候被调用。可以通过 `monitor.isOver({ shallow: true })` 方法来检查悬停是仅发生在当前目标上还是嵌套上。
  - canDrop(props, monitor): 可选参数。这个方法可以用来检测 `drop target` 是否接受 item。

```autohotkey
方法中的参数解释：
- props：当前组件的 `props` 参数。
- monitor：一个 `DropTargetMonitor` 实例。通过它可以获取当前的拖拽信息，比如可以获取当前被拖拽的项目及其类型，当前和初始坐标和偏移，以及它是否已被删除。
- component：是组件的实例。使用它可以访问DOM元素来进行位置或大小测量，或调用组件里面定义的方法，或者进行 `setState` 操作。有时候在 isDragging、 canDrag 方法里可能获取不到 `component` 这个参数，因为它们被调用时实例可能不可用。
```

- collect: 必填项，把拖拽过程中需要的信息注入组件的 `props`，接收两个参数 `connect` 和 `monitor`。
  - connect: `DropTargetConnector` 的实例，包括 `dropTarget` 一个方法。
    - dropTarget: 返回一个函数，传递给组件用来将 `source DOM` 和 `React DnD Backend` 连接起来。
  - monitor: `DropTargetMonitor` 的实例，包含的具体方法可以参考[这里](https://link.segmentfault.com/?url=https%3A%2F%2Freact-dnd.github.io%2Freact-dnd%2Fdocs%2Fapi%2Fdrop-target-monitor)。



### DragDropContext && DragDropContextProvider

使用 `DragSource` 和 `DropTarget` 包裹的组件必须放置在 `DragDropContext` 或者 `DragDropContextProvider` 组件内部。

**基本用法：**

```javascript
import Backend from 'react-dnd-html5-backend'
import { DndProvider } from 'react-dnd'

export default function MyReactApp() {
  return (
    <DndProvider backend={Backend}>
      /* your drag-and-drop application */
    </DndProvider>
  )
}
```

**参数**：

- backend： 必填项。HTML5 DnD API 兼容性不怎么样，并且不适用于移动端，所以干脆把 DnD 相关具体DOM事件抽离出去，单独作为一层，即 Backend，我们可以根据 React DnD提供的约定协议定义自己的 Backend。











## 用 React Hooks 的方式使用 react-dnd



### react-dnd的hooks API主要有2个:

1. useDrag 拖动
2. useDrop 放置



### 环境搭建

- 安装 `react-dnd`：`yarn add react-dnd`
- 安装 `react-dnd-html5-backend`：`yarn add react-dnd-html5-backend`



~~~js
// index.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { DndProvider } from 'react-dnd';
import {HTML5Backend} from 'react-dnd-html5-backend'
import Box from './Box';

ReactDOM.render(
    <DndProvider backend={ HTML5Backend }>
        <Box />
    </DndProvider>,
    document.getElementById('root'));
~~~











