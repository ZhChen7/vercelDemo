# Service Workers

### 本文速览

- service worker 的基本架构
- 怎么注册一个 service worker
- 一个新  service worker 的 install 及 activation 过程
- 怎么更新 service worker 还有它的缓存控制和自定义响应



------





## Service Workers

> Service Worker 提供这些**功能所依赖的技术基础** ：
>
> 丰富的离线体验、定期的后台同步以及推送通知等通常需要将面向本机应用的功能将引入到网页应用中



## 背景

有一个困扰 web 用户多年的难题——丢失网络连接。即使是世界上最好的 web app，如果下载不了它，也是非常糟糕的体验。如今虽然已经有很多种技术去尝试着解决这一问题。而随着[离线](https://developer.mozilla.org/en-US/Apps/Build/Offline)页面的出现，一些问题已经得到了解决。但是，最重要的问题是，**仍然没有一个好的统筹机制对资源缓存和自定义的网络请求进行控制** 。

之前的尝试 — AppCache — 看起来是个不错的方法，因为它可以很容易地指定需要离线缓存的资源。但是，它假定你使用时会遵循诸多规则，如果你不严格遵循这些规则，它会把你的APP搞得一团糟。关于APPCache的更多详情，请看Jake Archibald的文章： [Application Cache is a Douchebag](https://alistapart.com/article/application-cache-is-a-douchebag).

> **注意**:  从Firefox44起，当使用 [AppCache](https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache) 来提供离线页面支持时，会提示一个警告消息，来建议开发者使用 [Service workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers) 来实现离线页面。([bug 1204581](https://bugzilla.mozilla.org/show_bug.cgi?id=1204581).)

Service worker 最终要去解决这些问题。虽然 Service Worker 的语法比 AppCache 更加复杂，但是你可以使用 JavaScript 更加精细地控制 AppCache 的静默行为。有了它，你可以解决目前离线应用的问题，同时也可以做更多的事。 Service Worker 可以使你的应用先访问本地缓存资源，**所以在离线状态时，在没有通过网络接收到更多的数据前，仍可以提供基本的功能（一般称之为 [Offline First](http://offlinefirst.org/)）**。这是原生APP 本来就支持的功能，这也是相比于 web app，原生 app 更受青睐的主要原因。



### 什么是 Service Worker

Service Worker 是浏览器在后台独立于网页运行的脚本，它打开了通向不需要网页或用户交互的功能的大门。 现在，它们已包括如[推送通知](https://developers.google.com/web/updates/2015/03/push-notifications-on-the-open-web)和[后台同步](https://developers.google.com/web/updates/2015/12/background-sync)等功能。 **将来，Service Worker 将会支持如定期同步或地理围栏等其他功能**。 

这个 API 之所以令人兴奋，是因为它可以支持**离线体验，让开发者能够全面控制这一体验**。

Service Worker 相关注意事项:

- 它是一种 [JavaScript Worker](https://www.html5rocks.com/en/tutorials/workers/basics/)，无法直接访问 DOM。 Service Worker 通过响应 [postMessage](https://html.spec.whatwg.org/multipage/workers.html#dom-worker-postmessage) 接口发送的消息来与其控制的页面通信，页面可在必要时对 DOM 执行操作。
- Service Worker 是一种可编程网络代理，让您能够控制页面所发送网络请求的处理方式。
- Service Worker 在不用时会被中止，并在下次有需要时重启，因此，您不能依赖 Service Worker `onfetch` 和 `onmessage` 处理程序中的全局状态。 如果存在您需要持续保存并在重启后加以重用的信息，Service Worker 可以访问 [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)。
- Service Worker 广泛地利用了 promise





## 基本架构

通常遵循以下基本步骤来使用 service workers：

1. service worker URL 通过 [`serviceWorkerContainer.register()`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerContainer/register) 来获取和注册。
2. 如果注册成功，service worker 就在 [`ServiceWorkerGlobalScope`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerGlobalScope) 环境中运行； 这是一个特殊类型的 worker 上下文运行环境，与主运行线程（执行脚本）相独立，同时也没有访问 DOM 的能力。
3. service worker 现在可以处理事件了。
4. 受 service worker 控制的页面打开后会尝试去安装 service worker。最先发送给 service worker 的事件是安装事件(在这个事件里可以开始进行填充 IndexDB和缓存站点资源)。这个流程同原生 APP 或者 Firefox OS APP 是一样的 — 让所有资源可离线访问。
5. 当 `oninstall` 事件的处理程序执行完毕后，可以认为 service worker 安装完成了。
6. 下一步是激活。当 service worker 安装完成后，会接收到一个激活事件(activate event)。 `onactivate` 主要用途是清理先前版本的 service worker 脚本中使用的资源。
7. Service Worker 现在可以控制页面了，但仅是在 `register()` 成功后的打开的页面。也就是说，页面起始于有没有 service worker ，且在页面的接下来生命周期内维持这个状态。所以，页面不得不重新加载以让 service worker 获得完全的控制。

![img](https://mdn.mozillademos.org/files/12636/sw-lifecycle.png)

下图展示了 service worker 所有支持的事件：

![install, activate, message, fetch, sync, push](https://mdn.mozillademos.org/files/12632/sw-events.png)



### [Promises](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers#promises)

[Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 是一种非常适用于异步操作的机制，一个操作依赖于另一个操作的成功执行。这是 service worker 的核心工作机制。







## 基本使用

### 注册你的 worker

我们 app 的 JavaScript 文件里 — `app.js` — 的第一块代码就像下面的一样。这是我们使用 service worker 的入口：

```javascript
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw-test/sw.js', { scope: '/sw-test/' }).then(function(reg) {
    // registration worked
    console.log('Registration succeeded. Scope is ' + reg.scope);
  }).catch(function(error) {
    // registration failed
    console.log('Registration failed with ' + error);
  });
}
```

1. 外面的代码块做了一个特性检查，在注册之前确保 service worker 是支持的。
2. 接着，我们使用 [`ServiceWorkerContainer.register()`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerContainer/register) 函数来注册站点的 service worker，service worker 只是一个驻留在我们的 app 内的一个 JavaScript 文件 (注意，这个文件的url 是相对于 origin， 而不是相对于引用它的那个 JS 文件)。
3.  `scope` 参数是选填的，可以被用来指定你想让 service worker 控制的内容的子目录。在这个例子例，我们指定了 `'/sw-test/'`，表示 app 的 origin 下的所有内容。如果你留空的话，默认值也是这个值， 我们在指定只是作为例子。
4. `.then()` 函数链式调用我们的 promise，当 promise resolve 的时候，里面的代码就会执行。
5. 最后面我们链了一个 `.catch()` 函数，当 promise rejected 才会执行。



## Service workers demo

> 页面非常简单，而且是静态的，但也注册、安装和激活了 service worker，当浏览器支持的时候，它将缓存所有依赖的文件，它可以在离线的时候访问！ [demo地址](https://mdn.github.io/sw-test/) 

~~~js
// register service worker
// app.js

if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw-test/sw.js', { scope: '/sw-test/' }).then(function(reg) {

    if(reg.installing) {
      console.log('Service worker installing');
    } else if(reg.waiting) {
      console.log('Service worker installed');
    } else if(reg.active) {
      console.log('Service worker active');
    }

  }).catch(function(error) {
    // registration failed
    console.log('Registration failed with ' + error);
  });
}

// function for loading each image via XHR

function imgLoad(imgJSON) {
  // return a promise for an image loading
  return new Promise(function(resolve, reject) {
    var request = new XMLHttpRequest();
    request.open('GET', imgJSON.url);
    request.responseType = 'blob';

    request.onload = function() {
      if (request.status == 200) {
        var arrayResponse = [];
        arrayResponse[0] = request.response;
        arrayResponse[1] = imgJSON;
        resolve(arrayResponse);
      } else {
        reject(Error('Image didn\'t load successfully; error code:' + request.statusText));
      }
    };

    request.onerror = function() {
      reject(Error('There was a network error.'));
    };

    // Send the request
    request.send();
  });
}

var imgSection = document.querySelector('section');

window.onload = function() {

  // load each set of image, alt text, name and caption
  for(var i = 0; i<=Gallery.images.length-1; i++) {
    imgLoad(Gallery.images[i]).then(function(arrayResponse) {

      var myImage = document.createElement('img');
      var myFigure = document.createElement('figure');
      var myCaption = document.createElement('caption');
      var imageURL = window.URL.createObjectURL(arrayResponse[0]);

      myImage.src = imageURL;
      myImage.setAttribute('alt', arrayResponse[1].alt);
      myCaption.innerHTML = '<strong>' + arrayResponse[1].name + '</strong>: Taken by ' + arrayResponse[1].credit;

      imgSection.appendChild(myFigure);
      myFigure.appendChild(myImage);
      myFigure.appendChild(myCaption);

    }, function(Error) {
      console.log(Error);
    });
  }
};
~~~

~~~js
// sw.js
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open('v1').then(function(cache) {
      return cache.addAll([
        '/sw-test/',
        '/sw-test/index.html',
        '/sw-test/style.css',
        '/sw-test/app.js',
        '/sw-test/image-list.js',
        '/sw-test/star-wars-logo.jpg',
        '/sw-test/gallery/bountyHunters.jpg',
        '/sw-test/gallery/myLittleVader.jpg',
        '/sw-test/gallery/snowTroopers.jpg'
      ]);
    })
  );
});

self.addEventListener('fetch', function(event) {
  event.respondWith(caches.match(event.request).then(function(response) {
    // caches.match() always resolves
    // but in case of success response will have value
    if (response !== undefined) {
      return response;
    } else {
      return fetch(event.request).then(function (response) {
        // response may be used only once
        // we need to save clone to put one copy in cache
        // and serve second one
        // 为什么要这样做？这是因为请求和响应流只能被读取一次。为了给浏览器返回响应以及把它缓存起来，我们不得不克隆一份。所以原始的会返回给浏览器，克隆的会发送到缓存中。它们都是读取了一次。
        let responseClone = response.clone();
        
        caches.open('v1').then(function (cache) {
          cache.put(event.request, responseClone);
        });
        
        return response;
      }).catch(function () {
        return caches.match('/sw-test/gallery/myLittleVader.jpg');
      });
    }
  }));
});
~~~

1. 这里我们 新增了一个 `install` 事件监听器，接着在事件上接了一个[`ExtendableEvent.waitUntil()`](https://developer.mozilla.org/zh-CN/docs/Web/API/ExtendableEvent/waitUntil) 方法——这会确保Service Worker 不会在 `waitUntil()` 里面的代码执行完毕之前安装完成。
2. 在 `waitUntil()` 内，我们使用了 [`caches.open()`](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/open) 方法来创建了一个叫做 `v1` 的新的缓存，将会是我们的站点资源缓存的第一个版本。它返回了一个创建缓存的 promise，当它 resolved 的时候，我们接着会调用在创建的缓存示例上的一个方法 `addAll()`，这个方法的参数是一个由一组相对于 origin 的 URL 组成的数组，这些 URL 就是你想缓存的资源的列表。
3. 如果 promise 被 rejected，安装就会失败，这个 worker 不会做任何事情。这也是可以的，因为你可以修复你的代码，在下次注册发生的时候，又可以进行尝试。
4. 当安装成功完成之后， service worker 就会激活。在第一次你的 service worker 注册／激活时，这并不会有什么不同。但是当 service worker 更新 (稍后查看 [Updating your service worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers#updating_your_service_worker) 部分) 的时候 ，就不太一样了。

> **注意**: [localStorage (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) 跟 service worker 的 cache 工作原理很类似，但是它是同步的，所以不允许在 service workers 内使用。



### 自定义请求的响应

现在你已经将你的站点资源缓存了，你需要告诉 service worker 让它用这些缓存内容来做点什么。有了 `fetch` 事件，这是很容易做到的。

每次任何被 service worker 控制的资源被请求到时，都会触发 `fetch` 事件，这些资源包括了指定的 scope 内的文档，和这些文档内引用的其他任何资源（比如 `index.html` 发起了一个跨域的请求来嵌入一个图片，这个也会通过 service worker 。）

你可以给 service worker 添加一个 `fetch` 的事件监听器，接着调用 event 上的 `respondWith()` 方法来劫持我们的 HTTP 响应，然后你用可以用自己的方法来更新他们。

```javascript
this.addEventListener('fetch', function(event) {
  event.respondWith(
    // magic goes here
  );
});
```

我们可以用一个简单的例子开始，在任何情况下我们只是简单的响应这些缓存中的 url  和网络请求匹配的资源。

```javascript
this.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
  );
});
```

`caches.match(event.request)` 允许我们对网络请求的资源和 cache 里可获取的资源进行匹配，查看是否缓存中有相应的资源。这个匹配通过 url 和 vary header进行，就像正常的 http 请求一样。



![image.png](https://i.loli.net/2021/09/08/4jXnQeaZH3ci2dM.png)





## 恢复失败的请求

在有 service worker cache 里匹配的资源时， `caches.match(event.request)` 是非常棒的。但是如果没有匹配资源呢？如果我们不提供任何错误处理，promise 就会 reject，同时也会出现一个网络错误。

幸运的是，service worker 的基于 promise 的结构，使得提供更多的成功的选项变得微不足道。 我们可以这样做：

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(response) {
      return response || fetch(event.request);
    })
  );
});
```

如果 promise reject了， catch() 函数会执行默认的网络请求，意味着在网络可用的时候可以直接像服务器请求资源。

如果我们足够聪明的话，我们就不会只是从服务器请求资源，而且还会把请求到的资源保存到缓存中，以便将来离线时所用！这意味着如果其他额外的图片被加入到 Star Wars 图库里，我们的 app 会自动抓取它们。下面就是这个诀窍：

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(resp) {
      return resp || fetch(event.request).then(function(response) {
        return caches.open('v1').then(function(cache) {
          cache.put(event.request, response.clone());
          return response;
        });
      });
    })
  );
});
```

这里我们用 `fetch(event.request)` 返回了默认的网络请求，它返回了一个 promise 。当网络请求的 promise 成功的时候，我们 通过执行一个函数用 `caches.open('v1')` 来抓取我们的缓存，它也返回了一个 promise。当这个 promise 成功的时候， `cache.put()` 被用来把这些资源加入缓存中。资源是从 `event.request` 抓取的，它的响应会被 `response.clone()` 克隆一份然后被加入缓存。这个克隆被放到缓存中，它的原始响应则会返回给浏览器来给调用它的页面。

为什么要这样做？这是因为请求和响应流只能被读取一次。为了给浏览器返回响应以及把它缓存起来，我们不得不克隆一份。所以原始的会返回给浏览器，克隆的会发送到缓存中。它们都是读取了一次。

我们现在唯一的问题是当请求没有匹配到缓存中的任何资源的时候，以及网络不可用的时候，我们的请求依然会失败。让我们提供一个默认的回退方案以便不管发生了什么，用户至少能得到些东西：

```javascript
this.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function() {
      return fetch(event.request).then(function(response) {
        return caches.open('v1').then(function(cache) {
          cache.put(event.request, response.clone());
          return response;
        });
      });
    }).catch(function() {
      return caches.match('/sw-test/gallery/myLittleVader.jpg');
    })
  );
});
```

因为只有新图片会失败，我们已经选择了回退的图片，一切都依赖我们之前看到的 `install` 事件侦听器中的安装过程。





