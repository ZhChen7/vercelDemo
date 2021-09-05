# web Worker

### 本文速览

- web Worker



------



## Web Worker 简介：为 JavaScript 引入线程技术

JavaScript 语言采用的是单线程模型，也就是说，所有任务只能在一个线程上完成，一次只能做一件事。前面的任务没做完，后面的任务只能等着。随着电脑计算能力的增强，尤其是多核 CPU 的出现，单线程带来很大的不便，无法充分发挥计算机的计算能力。

Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。



## Web Worker 的类型

> 两种 Web Worker：[专用 Worker](http://www.whatwg.org/specs/web-workers/current-work/#dedicated-workers-and-the-worker-interface) 和 [共用 Worker](http://www.whatwg.org/ pecs/web-workers/current-work/#sharedworker)

在专用workers的情况下，[`DedicatedWorkerGlobalScope`](https://developer.mozilla.org/zh-CN/docs/Web/API/DedicatedWorkerGlobalScope) 对象代表了worker的上下文（专用workers是指标准worker仅在单一脚本中被使用；共享worker的上下文是[`SharedWorkerGlobalScope` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorkerGlobalScope)对象）。一个专用worker仅仅能被首次生成它的脚本使用，而共享worker可以同时被多个脚本使用。



## Web Worker 基本用法（[专用 Worker](http://www.whatwg.org/specs/web-workers/current-work/#dedicated-workers-and-the-worker-interface) ）

> 一个worker是使用一个**构造函数**创建的一个对象(e.g. [`Worker()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/Worker)) 运行一个命名的 JavaScript文件 - 这个文件包含将在工作线程中运行的代码; **workers** 运行在另一个全局上下文中,不同于当前的[`window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window). 因此，在 [`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker) 内通过 [`window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)获取全局作用域 (而不是[`self`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/self)) 将返回错误。



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

```
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





## [共享worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#共享worker)

一个共享worker可以被多个脚本使用——即使这些脚本正在被不同的window、iframe或者worker访问。这一部分，我们会讨论[共享worker基础示例](https://github.com/mdn/simple-shared-worker)（[运行共享worker](https://mdn.github.io/simple-shared-worker/)）中的javascript代码：该示例与专用worker基础示例非常相像，只是有2个可用函数被存放在不同脚本文件中：两数相乘函数，以及求平方函数。这两个脚本用同一个worker来完成实际需要的运算。



### [生成一个共享worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#生成一个共享worker)

生成一个新的共享worker与生成一个专用worker非常相似，只是构造器的名字不同 ——生成共享worker的代码如下：

```javascript
var myWorker = new SharedWorker('worker.js');
```

一个非常大的区别在于，与一个共享worker通信必须通过端口对象——一个确切的打开的端口供脚本与worker通信（在专用worker中这一部分是隐式进行的）。

在传递消息之前，端口连接必须被显式的打开，打开方式是使用onmessage事件处理函数或者start()方法。示例中的 [multiply.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js) 和 [worker.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/worker.js) 文件没有调用了start()方法，这些调用并不那么重要是因为onmessage事件处理函数正在被使用。start()方法的调用只在一种情况下需要，那就是消息事件被addEventListener()方法使用。

在使用start()方法打开端口连接时，如果父级线程和worker线程需要双向通信，那么它们都需要调用start()方法。

```
myWorker.port.start();  // 父级线程中的调用
```

Copy to Clipboard

```
port.start(); // worker线程中的调用, 假设port变量代表一个端口
```



### [共享worker中消息的接收和发送](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#共享worker中消息的接收和发送)

现在，消息可以像之前那样发送到worker了，但是`postMessage()` 方法必须被端口对象调用（你会再一次看到 [multiply.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js) 和 [square.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/square.js)中相似的结构）：

```javascript
squareNumber.onchange = function() {
  myWorker.port.postMessage([squareNumber.value,squareNumber.value]);
  console.log('Message posted to worker');
}
```

Copy to Clipboard

回到worker中，这里也有一些些复杂（[worker.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/worker.js)）:

```javascript
onconnect = function(e) {
  var port = e.ports[0];

  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
    port.postMessage(workerResult);
  }
}
```

Copy to Clipboard

首先，当一个端口连接被创建时（例如：在父级线程中，设置onmessage事件处理函数，或者显式调用start()方法时），使用onconnect事件处理函数来执行代码。

使用事件的ports属性来获取端口并存储在变量中。

然后，为端口添加一个消息处理函数用来做运算并回传结果给主线程。在worker线程中设置此消息处理函数也会隐式的打开与主线程的端口连接，因此这里跟前文一样，对port.start()的调用也是不必要的。

最后，回到主脚本，我们处理消息（你会又一次看到 [multiply.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js) 和 [square.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/square.js)中相似的结构）：

```javascript
myWorker.port.onmessage = function(e) {
  result2.textContent = e.data;
  console.log('Message received from worker');
}
```

当一条消息通过端口回到worker，我们检查结果的类型，然后将运算结果放入结果段落中合适的地方。



## [关于线程安全](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#关于线程安全)

[`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker)接口会生成真正的操作系统级别的线程，如果你不太小心，那么并发会对你的代码产生有趣的影响。然而，对于 web worker 来说，与其他线程的通信点会被很小心的控制，这意味着你很难引起并发问题。你没有办法去访问非线程安全的组件或者是 DOM，此外你还需要通过序列化对象来与线程交互特定的数据。所以你要是不费点劲儿，还真搞不出错误来。

## [内容安全策略](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#内容安全策略)

有别于创建它的document对象，worker有它自己的执行上下文。因此普遍来说，worker并不受限于创建它的document（或者父级worker）的内容安全策略。我们来举个例子，假设一个document有如下头部声明：

```
Content-Security-Policy: script-src 'self'
```

这个声明有一部分作用在于，禁止它内部包含的脚本代码使用eval()方法。然而，如果脚本代码创建了一个worker，在worker上下文中执行的代码却是可以使用eval()的。

为了给worker指定内容安全策略，必须为发送worker代码的请求本身加上一个 [内容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)。

有一个例外情况，即worker脚本的源如果是一个全局性的唯一的标识符（例如，它的URL指定了数据模式或者blob），worker则会继承创建它的document或者worker的CSP（Content security policy内容安全策略）。

### [划分任务给多个 worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#划分任务给多个_worker)

当多核系统流行开来，将复杂的运算任务分配给多个 worker 来运行已经变得十分有用，这些 worker 会在多处理器内核上运行这些任务。

## [其它类型的worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#其它类型的worker)

除了专用和共享的web worker，还有一些其它类型的worker：

- [ServiceWorkers](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker_API) （服务worker）一般作为web应用程序、浏览器和网络（如果可用）之前的代理服务器。它们旨在（除开其他方面）创建有效的离线体验，拦截网络请求，以及根据网络是否可用采取合适的行动并更新驻留在服务器上的资源。他们还将允许访问推送通知和后台同步API。
- Chrome Workers 是一种仅适用于firefox的worker。如果您正在开发附加组件，希望在扩展程序中使用worker且有在你的worker中访问 [js-ctypes](https://developer.mozilla.org/en/js-ctypes) 的权限，你可以使用Chrome Workers。详情请参阅`ChromeWorker`。
- [Audio Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API#Audio_Workers) （音频worker）使得在web worker上下文中直接完成脚本化音频处理成为可能。



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