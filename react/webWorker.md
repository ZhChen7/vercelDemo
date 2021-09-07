# web Worker

### 本文速览

- web Worker 【 专用worker 】
- SharedWorker 【 共享worker 】



------



## Web Worker 简介：为 JavaScript 引入线程技术

> **背景：** JavaScript 语言采用的是 **单线程模型**，也就是说，所有任务只能在一个线程上完成，**一次只能做一件事**。前面的任务没做完，后面的任务只能等着。随着电脑计算能力的增强，尤其是多核 CPU 的出现，单线程带来很大的不便，无法充分发挥计算机的计算能力。

**Web Worker 的作用** 

- 就是为 JavaScript 创造多线程环境，允许 **主线程** 创建 Worker 线程，将一些任务分配给后者运行。**在主线程运行的同时，Worker 线程在后台运行，两者互不干扰**。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，**一些计算密集型或高延迟的任务** ，**被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。** 

Worker 线程一旦新建成功，就会始终运行，**不会被主线程上的活动（比如用户点击按钮、提交表单）打断**。这样有利于随时响应主线程的通信。但是，**这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。**

> 介绍：Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面。此外，他们可以使用XMLHttpRequest执行 I/O (尽管responseXML和channel属性总是为空)。一旦创建， 一个worker 可以将消息发送到创建它的JavaScript代码, 通过将消息发布到该代码指定的事件处理程序（反之亦然）。



## Web Worker 的类型

> 两种 Web Worker：[专用 Worker](http://www.whatwg.org/specs/web-workers/current-work/#dedicated-workers-and-the-worker-interface) 和 [共用 Worker - SharedWorker ](http://www.whatwg.org/ pecs/web-workers/current-work/#sharedworker) 

在专用workers的情况下，[`DedicatedWorkerGlobalScope`](https://developer.mozilla.org/zh-CN/docs/Web/API/DedicatedWorkerGlobalScope) 对象（也就是 [`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker) 全局作用域）代表了worker的上下文（ 专用workers是指标准 worker仅在单一脚本中被使用；共享worker的上下文是[`SharedWorkerGlobalScope` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorkerGlobalScope)对象）。

> ⚠️ ： **一个专用worker仅仅能被首次生成它的脚本使用，而共享worker可以同时被多个脚本使用。**  



> **`DedicatedWorkerGlobalScope`** 对象（也就是 [`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker) 全局作用域）可以通过 [`self`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/self) 关键字来访问。一些在 worker 全局作用域下不可用的全局函数、命名空间对象以及构造器，也可以通过此对象使用。在 [JavaScript 参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference)的 [Web Workers 可以使用的函数和类 (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers)页面中，有列举这些特性。



## Web Worker 基本用法（[专用 Worker](http://www.whatwg.org/specs/web-workers/current-work/#dedicated-workers-and-the-worker-interface) ）

> **一个worker** 是使用一个**构造函数**创建的一个对象 运行一个命名的 JavaScript文件 - 这个文件包含将在工作线程中运行的代码;  
>
> **workers** 运行在另一个全局上下文中,不同于当前的[`window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window). 因此，在 [`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker) 内通过 [`window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)获取全局作用域 (而不是[`self`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/self)) 将返回错误。

### 主线程

主线程采用`new`命令，调用`Worker()`构造函数，新建一个 Worker 线程。

> ```javascript
> var worker = new Worker('work.js');
> ```

`Worker()`构造函数的参数是一个脚本文件，该文件就是 Worker 线程所要执行的任务。由于 Worker 不能读取本地文件，所以这个脚本必须来自网络。如果下载没有成功（比如404错误），Worker 就会默默地失败。

然后，主线程调用`worker.postMessage()`方法，向 Worker 发消息。

> ```javascript
> worker.postMessage('Hello World');
> worker.postMessage({method: 'echo', args: ['Work']});
> ```

`worker.postMessage()`方法的参数，就是主线程传给 Worker 的数据。它可以是各种数据类型，包括二进制数据。

接着，主线程通过`worker.onmessage`指定监听函数，接收子线程发回来的消息。

> ```javascript
> worker.onmessage = function (event) {
>   console.log('Received message ' + event.data);
>   doSomething();
> }
> 
> function doSomething() {
>   // 执行任务
>   worker.postMessage('Work done!');
> }
> ```

上面代码中，事件对象的`data`属性可以获取 Worker 发来的数据。

Worker 完成任务以后，主线程就可以把它关掉。

> ```javascript
> worker.terminate();
> ```



**worker特性检测** 

为了更好的错误处理控制以及向下兼容，将你的worker运行代码包裹在以下代码中是一个很好的想法(main.js)：

```js
if (window.Worker) {
  ...
}
```



### Worker 线程

Worker 线程内部需要有一个监听函数，监听`message`事件。

> ```javascript
> self.addEventListener('message', function (e) {
>   self.postMessage('You said: ' + e.data);
> }, false);
> ```

上面代码中，`self`代表子线程自身，即子线程的全局对象。因此，等同于下面两种写法。

> ```javascript
> // 写法一
> this.addEventListener('message', function (e) {
>   this.postMessage('You said: ' + e.data);
> }, false);
> 
> // 写法二
> addEventListener('message', function (e) {
>   postMessage('You said: ' + e.data);
> }, false);
> ```

除了使用`self.addEventListener()`指定监听函数，也可以使用`self.onmessage`指定。监听函数的参数是一个事件对象，它的`data`属性包含主线程发来的数据。`self.postMessage()`方法用来向主线程发送消息。

根据主线程发来的数据，Worker 线程可以调用不同的方法，下面是一个例子。

> ```javascript
> self.addEventListener('message', function (e) {
>   var data = e.data;
>   switch (data.cmd) {
>     case 'start':
>       self.postMessage('WORKER STARTED: ' + data.msg);
>       break;
>     case 'stop':
>       self.postMessage('WORKER STOPPED: ' + data.msg);
>       self.close(); // Terminates the worker.
>       break;
>     default:
>       self.postMessage('Unknown command: ' + data.msg);
>   };
> }, false);
> ```

上面代码中，`self.close()`用于在 Worker 内部关闭自身。



### Worker 加载脚本

Worker 内部如果要加载其他脚本，有一个专门的方法`importScripts()`。

> ```javascript
> importScripts('script1.js');
> ```

该方法可以同时加载多个脚本。

> ```javascript
> importScripts('script1.js', 'script2.js');
> ```



### 错误处理

主线程可以监听 Worker 是否发生错误。如果发生错误，Worker 会触发主线程的`error`事件。

> ```javascript
> worker.onerror(function (event) {
>   console.log([
>     'ERROR: Line ', e.lineno, ' in ', e.filename, ': ', e.message
>   ].join(''));
> });
> 
> // 或者
> worker.addEventListener('error', function (event) {
>   // ...
> });
> ```

Worker 内部也可以监听`error`事件。



### 关闭 Worker

使用完毕，为了节省系统资源，必须关闭 Worker。

> ```javascript
> // 主线程
> worker.terminate();
> 
> // Worker 线程
> self.close();
> ```



## 同页面的 Web Worker

通常情况下，Worker 载入的是一个单独的 JavaScript 脚本文件，但是也可以载入与主线程在同一个网页的代码。

> ```html
> <!DOCTYPE html>
>   <body>
>     <script id="worker" type="app/worker">
>       addEventListener('message', function () {
>         postMessage('some message');
>       }, false);
>     </script>
>   </body>
> </html>
> ```

上面是一段嵌入网页的脚本，注意必须指定`<script>`标签的`type`属性是一个浏览器不认识的值，上例是`app/worker`。

然后，读取这一段嵌入页面的脚本，用 Worker 来处理。

> ```javascript
> var blob = new Blob([document.querySelector('#worker').textContent]);
> var url = window.URL.createObjectURL(blob);
> var worker = new Worker(url);
> 
> worker.onmessage = function (e) {
>   // e.data === 'some message'
> };
> ```

上面代码中，先将嵌入网页的脚本代码，转成一个二进制对象，然后为这个二进制对象生成 URL，再让 Worker 加载这个 URL。这样就做到了，主线程和 Worker 的代码都在同一个网页上面。



## API

###  主线程

浏览器原生提供`Worker()`构造函数，用来供主线程生成 Worker 线程。

> ```javascript
> var myWorker = new Worker(jsUrl, options);
> ```

`Worker()`构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

> ```javascript
> // 主线程
> var myWorker = new Worker('worker.js', { name : 'myWorker' });
> 
> // Worker 线程
> self.name // myWorker
> ```

`Worker()`构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下。

> - Worker.onerror：指定 error 事件的监听函数。
> - Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在`Event.data`属性中。
> - Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
> - Worker.postMessage()：向 Worker 线程发送消息。
> - Worker.terminate()：立即终止 Worker 线程。
>
> 

### Worker 线程

Web Worker 有自己的全局对象，不是主线程的`window`，而是一个专门为 Worker 定制的全局对象。因此定义在`window`上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

> - self.name： Worker 的名字。该属性只读，由构造函数指定。
> - self.onmessage：指定`message`事件的监听函数。
> - self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
> - self.close()：关闭 Worker 线程。
> - self.postMessage()：向产生这个 Worker 线程发送消息。
> - self.importScripts()：加载 JS 脚本。



## Web Worker使用要点

- 同源限制：分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。
- DOM 限制：Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用document、window、parent这些对象。但是，Worker 线程可以navigator对象和location对象。
- 通信联系：Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。
- 脚本限制：Worker 线程不能执行alert()方法和confirm()方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。
- 文件限制：Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。后面我们允许会做处理。





## 其它类型的worker

除了专用和共享的web worker，还有一些其它类型的worker：

- [ServiceWorkers](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker_API) （服务worker）一般作为web应用程序、浏览器和网络（如果可用）之前的代理服务器。它们旨在（除开其他方面）创建有效的离线体验，拦截网络请求，以及根据网络是否可用采取合适的行动并更新驻留在服务器上的资源。他们还将允许访问推送通知和后台同步API。
- Chrome Workers 是一种仅适用于firefox的worker。如果您正在开发附加组件，希望在扩展程序中使用worker且有在你的worker中访问 [js-ctypes](https://developer.mozilla.org/en/js-ctypes) 的权限，你可以使用Chrome Workers。详情请参阅`ChromeWorker`。
- [Audio Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API#Audio_Workers) （音频worker）使得在web worker上下文中直接完成脚本化音频处理成为可能。



## worker中可用的函数和接口

你可以在web worker中使用大多数的标准javascript特性，包括

- [`Navigator`](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator)
- [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
- [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array), [`Date`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date), [`Math`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math), and [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)
- [`WindowTimers.setTimeout` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) and [`WindowTimers.setInterval` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)

在一个worker中最主要的你不能做的事情就是直接影响父页面。包括操作父页面的节点以及使用页面中的对象。你只能间接地实现，通过[`DedicatedWorkerGlobalScope.postMessage` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope/postMessage)回传消息给主脚本，然后从主脚本那里执行操作或变化。

**注意：**获取worker中完整的方法列表，请参阅[Functions and interfaces available to workers](https://developer.mozilla.org/en-US/docs/Web/Reference/Functions_and_classes_available_to_workers)。