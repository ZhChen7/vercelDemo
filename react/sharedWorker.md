# SharedWorker

### 本文速览

- 共享worker -  SharedWorker 概念
- SharedWorker 基本使用



------



## 共享worker - SharedWorker

> **SharedWorker** 是一种特殊的 WebWorker，可有支持多个浏览器上下文的通信功能，例如多个窗口、iframe。
>
> 一个共享worker可以被多个脚本使用——即使这些脚本正在被不同的window、iframe或者worker访问。

**注意：如果要使** SharedWorker **连接到多个不同的页面，这些页面必须是同源的（相同的协议、host 以及端口）**



## 基本使用

### 生成一个共享worker

生成一个新的共享worker与生成一个专用worker非常相似，只是构造器的名字不同：

```javascript
var myWorker = new SharedWorker('worker.js');
```

**与专用worker的区别** ：与一个共享worker通信必须通过端口对象 —— 一个确切的打开的端口供脚本与worker通信（在专用worker中这一部分是隐式进行的）。

在传递消息之前，端口连接必须被显式的打开，打开方式是使用onmessage事件处理函数或者start()方法。示例中的 [multiply.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js) 和 [worker.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/worker.js) 文件没有调用了start()方法，这些调用并不那么重要是因为onmessage事件处理函数正在被使用。start()方法的调用只在一种情况下需要，那就是消息事件被addEventListener()方法使用。

~~~js
// multiply.js
var first = document.querySelector('#number1');
var second = document.querySelector('#number2');

var result1 = document.querySelector('.result1');

if (!!window.SharedWorker) {
  var myWorker = new SharedWorker("worker.js");

  first.onchange = function() {
    myWorker.port.postMessage([first.value, second.value]);
    console.log('Message posted to worker');
  }

  second.onchange = function() {
    myWorker.port.postMessage([first.value, second.value]);
    console.log('Message posted to worker');
  }

  myWorker.port.onmessage = function(e) {
    result1.textContent = e.data;
    console.log('Message received from worker');
    console.log(e.lastEventId);
  }
}
~~~

~~~js
// worker.js
onconnect = function(e) {
  var port = e.ports[0];
  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
    port.postMessage(workerResult);
  }
}
~~~

~~~html
<!-- index.html -->
<form>
  <div>
    <label for="number1">Multiply number 1: </label>
    <input type="text" id="number1" value="0">
  </div>
  <div>
    <label for="number2">Multiply number 2: </label>
    <input type="text" id="number2" value="0">
  </div>
</form>
<p class="result1">Result: 0</p>

<script src="multiply.js"></script>
<script src="nosubmit.js"></script>
~~~

![image.png](https://i.loli.net/2021/09/07/B4HN3MwRZpYgf7b.png)

在使用start()方法打开端口连接时，如果父级线程和worker线程需要双向通信，那么它们都需要调用start()方法。

```js
myWorker.port.start();  // 父级线程中的调用
```

```js
port.start(); // worker线程中的调用, 假设port变量代表一个端口
```



### 共享worker中消息的接收和发送

现在，消息可以像之前那样发送到worker了，但是`postMessage()` 方法必须被端口对象调用（你会再一次看到 [multiply.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js) 和 [square.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/square.js)中相似的结构）：

```js
squareNumber.onchange = function() {
  myWorker.port.postMessage([squareNumber.value,squareNumber.value]);
  console.log('Message posted to worker');
}
```

回到worker中，这里也有一些些复杂（[worker.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/worker.js)）:

```js
onconnect = function(e) {
  var port = e.ports[0];

  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
    port.postMessage(workerResult);
  }
}
```

首先，当一个端口连接被创建时（例如：在父级线程中，设置onmessage事件处理函数，或者显式调用start()方法时），使用onconnect事件处理函数来执行代码。

使用事件的ports属性来获取端口并存储在变量中。

然后，为端口添加一个消息处理函数用来做运算并回传结果给主线程。在worker线程中设置此消息处理函数也会隐式的打开与主线程的端口连接，因此这里跟前文一样，对port.start()的调用也是不必要的。

最后，回到主脚本，我们处理消息（你会又一次看到 [multiply.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js) 和 [square.js](https://github.com/mdn/simple-shared-worker/blob/gh-pages/square.js)中相似的结构）：

```js
myWorker.port.onmessage = function(e) {
  result2.textContent = e.data;
  console.log('Message received from worker');
}
```

当一条消息通过端口回到worker，我们检查结果的类型，然后将运算结果放入结果段落中合适的地方。





### 传递数据的例子

#### 例子 #1： 创建一个通用的 「异步 [`eval()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval)」

下面这个例子介绍了，如何在 worker 内使用 `eval() `来按顺序执行**异步的**任何种类的 JavaScript 代码：

```js
// Syntax: asyncEval(code[, listener])

var asyncEval = (function () {

  var aListeners = [], oParser = new Worker("data:text/javascript;charset=US-ASCII,onmessage%20%3D%20function%20%28oEvent%29%20%7B%0A%09postMessage%28%7B%0A%09%09%22id%22%3A%20oEvent.data.id%2C%0A%09%09%22evaluated%22%3A%20eval%28oEvent.data.code%29%0A%09%7D%29%3B%0A%7D");

  oParser.onmessage = function (oEvent) {
    if (aListeners[oEvent.data.id]) { aListeners[oEvent.data.id](oEvent.data.evaluated); }
    delete aListeners[oEvent.data.id];
  };


  return function (sCode, fListener) {
    aListeners.push(fListener || null);
    oParser.postMessage({
      "id": aListeners.length - 1,
      "code": sCode
    });
  };

})();
```

[data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/data_URIs) 相当于一个网络请求，它有如下返回：

```js
onmessage = function(oEvent) {
  postMessage({
    'id': oEvent.data.id,
    'evaluated': eval(oEvent.data.code)
  });
}
```

示例使用：

```js
// asynchronous alert message...
asyncEval("3 + 2", function (sMessage) {
    alert("3 + 2 = " + sMessage);
});

// asynchronous print message...
asyncEval("\"Hello World!!!\"", function (sHTML) {
    document.body.appendChild(document.createTextNode(sHTML));
});

// asynchronous void...
asyncEval("(function () {\n\tvar oReq = new XMLHttpRequest();\n\toReq.open(\"get\", \"http://www.mozilla.org/\", false);\n\toReq.send(null);\n\treturn oReq.responseText;\n})()");
```