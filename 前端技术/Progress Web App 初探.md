# Progressive Web App初探

标签（空格分隔）： 前端技术

[TOC]

---


### 什么是 Progressive Web App?
#### 1、先看看目前流行的App分类：Native App、Hybrid App、Web App
![此处输入图片的描述][1]
**Native App：**原生的应用，直接使用Object C(ios)或者Java(android)开发的App。
**Web App：**即一般的网页应用，在浏览器中打开。
**Hybrid App：**半原生半Web的混合类App，普遍采用的是Native的框架，Web的内容。如快塑网App，由Cordova提供不同系统的native框架，并向页面暴露访问设备的接口，使用Polymer等前端技术编写UI。

|&nbsp;|Native App|Web App|PWA|
|:--|:---|:---|:---|
|操作体验|✔️好|✘  较差|✔️
|性能|✔️稳定|  ✘受限于浏览器|✔️
|访问本地资源（通讯录、相册等）|✔️   |✘|    ✔️
|动效，转场|✔️ 动画效果出色|✘ |✔️
|消息推送|✔️ |✘|   ✔️
|开发成本|✘高  |✔️相对较低|✔️
|更新发布|✘缓慢，根据不同平台，提交-审核-上线流程复杂|✔升级于无形|✔️
|跨平台|✘根据不同平台开发多个版本|✔️能够跨多个平台和终端|✔️


**针对Web App和NativeApp的优缺点，Hybrid App就是一个折中的解决办法。**

#### 2、Google的解决方案：Progressive Web App

>
PWA本质上仍然是一个**Web App**，它依赖了更多ES6已标准化或起草阶段的API。
它的特点：
>
1. **渐进增强：** 能够让每一位用户使用，无论用户使用什么浏览器，因为它是始终以渐进增强为原则。
1. **响应式用户界面：** 适应任何环境：桌面电脑，智能手机，笔记本电脑，或者其他设备。
1. **不依赖网络连接：** 通过 service workers 可以在离线或者网速极差的环境下工作。
1. **类原生应用：** 有像原生应用般的交互和导航给用户原生应用般的体验，因为它是建立在 app shell model 上的。
1. **持续更新：** 受益于 service worker 的更新进程，应用能够始终保持更新。
1. **安全：** 通过 HTTPS 来提供服务来防止网络窥探，保证内容不被篡改。
1. **可发现：** 得益于 W3C manifests 元数据和 service worker 的登记，让搜索引擎能够找到 web 应用。
1. **再次访问：** 通过消息推送等特性让用户再次访问变得容易。
1. **可安装：** 允许用户保留对他们有用的应用在主屏幕上，不需要通过应用商店。
1. **可连接性：** 通过 URL 可以轻松分享应用，不用复杂的安装即可运行。

[官方解释][2]


----------

### PWA的黑科技
>
![此处输入图片的描述][3]
>
**PWA的核心是ServiceWorker。**
**[ServiceWorker的官方解释][4]：**
    >>  服务工作线程是浏览器在后台独立于网页运行的脚本，它打开了通向不需要网页或用户交互的功能的大门。现在，它们已包括如推送通知和后台同步等功能。将来，服务工作线程将会支持如定期同步或地理围栏等其他功能。
>
1. 它是一种 JavaScript 工作线程，无法直接访问 DOM。 服务工作线程通过响应 postMessage 接口发送的消息来与其控制的页面通信，页面可在必要时对 DOM 执行操作。
1. 服务工作线程是一种可编程网络代理，让您能够控制页面所发送网络请求的处理方式。
1. 它在不用时会被中止，并在下次有需要时重启，因此，您不能依赖于服务工作线程的 onfetch 和 onmessage 处理程序中的全局状态。如果存在您需要持续保存并在重启后加以重用的信息，服务工作线程可以访问 IndexedDB API。
1. 服务工作线程广泛地利用了 promise。


#### 1、如何实现离线工作
> 要实现离线工作，无非就是把资源缓存在本地，当网络环境不好的时候，直接从本地获取资源。PWA的离线能力依赖于：
>
1. 在Service workers 生命周期的不同阶段，可以根据资源的类型来缓存数据。
1. Service workers 可以截获 Progressive Web App 发起的请求并从缓存中返回响应。


在serviceWorker安装成功后，install事件会被触发，此时可以把静态的UI缓存起来。
![此处输入图片的描述][5]
```javascript
/**
 * service-worker.js
 * 存储应用外壳需要的资源
 */
const cacheName = 'weatherPWA-step-6-1';
//资源列表
const filesToCache = [
  '/',
  '/index.html',
  '/scripts/app.js',
  '/styles/inline.css',
  '/images/clear.png',
  '/images/cloudy-scattered-showers.png',
  '/images/cloudy.png',
  //...
];

self.addEventListener('install', e => {
  console.log('[ServiceWorker] Install');
  e.waitUntil(
    caches.open(cacheName).then(cache => {
      console.log('[ServiceWorker] Caching app shell');
      return cache.addAll(filesToCache);
    })
  );
});
```

serviceWorker拦截请求，根据请求匹配缓存中的响应，如果找到了则直接返回，否则将发起网络请求，然后把响应的结果返回，并且添加到缓存中以备下次使用。
![此处输入图片的描述][6]
```
/**
 * service-worker.js
 * 在页面发起http请求时，serviceWorker可以通过fetch事件拦截请求，并且给出自己的响应。
 *
 * 当一个请求被拦截时，首先会检查缓存里是否有上次发生的相同请求的响应，如果有则直接返回，否则重新发起请求，并把响应缓存起来供下次使用。
 * 当然，这样做并不合理，因为除了第一次外，每次请求都是取到缓存的数据，但是缓存可能早已过期。
 */
const dataCacheName = 'weatherData-v1';
self.addEventListener('fetch', e => {
  e.respondWith(
      caches.match(e.request).then(response => {
      return response || fetch(e.request).then(response => {
        return caches.open(dataCacheName).then(cache => {
            cache.put(e.request.url, response.clone());
            return response;
        });
    })
    );
  }
});
```
以上是两种最朴素的缓存策略，[查看更多][7] [翻译地址][8]

另外要解决一个问题：缓存过期时如何处理？PWA需要编程式的告诉serviceWorker需要删除哪些缓存。当serviceWorker被激活后，可以判断哪些缓存已经过期了（如何判断取决于你），然后把那些过期的移除。
>
要注意，当有文件被修改后，都需要修改``cacheName``，修改后，整个缓存都需要重新下载。这显然不是很高效。

![此处输入图片的描述][9]

```javascript
const cacheName = 'weatherPWA-step-6-2';
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.filter(function(cacheName) {
          // Return true if you want to remove this cache,
          // but remember that caches are shared across
          // the whole origin
        }).map(function(cacheName) {
          return caches.delete(cacheName);
        })
      );
    })
  );
});
```
#### 2、如何实现消息推送
##### 图解订阅与发送消息的过程
>
![此处输入图片的描述][10]
>
1. 下载应用，应用安装服务工作线程。用户授权允许消息推送服务。
1. App发起消息订阅。
1. 在订阅流期间，浏览器联系消息服务器来创建新的订阅并将其返回应用。
    ``注：您不需要知道消息服务器的网址。每个浏览器厂商都为旗下浏览器管理自有的消息服务器。``
1. 在订阅流后，您的应用会将订阅对象传回您的应用服务器。
1. 在稍后的某个时间，您的应用服务器会向消息服务器发送一条消息，后者会将其转发至接收方。


##### 代码实现
```javascript
/**
 * 注册service worker
 */
if('serviceWorker' in navigator) {
  navigator.serviceWorker
    .register('/service-worker.js')  //service-worker.js包含允许在后台运行的脚本
    .then(swRegistration => {
    //swRegistration后续需要用来订阅/退订消息
    // Registration was successful
    }).catch(err => {
        // registration failed :(
        console.log('ServiceWorker registration failed: ', err);
    });
}
```
利用service worker订阅和退订消息

```javascript
/**
 * 订阅消息
 */
swRegistration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: applicationServerKey
  }).then(subscription => {
    //User is subscribed
    updateSubscriptionOnServer(subscription);
  }).catch(err => {
    //Failed to subscribe the user
  });
```

```javascript
/**
 * 退订消息
 */
swRegistration.pushManager.getSubscription()
  .then(subscription => {
    if (subscription) {
      return subscription.unsubscribe();
    }
  }).catch(error => {
    console.log('Error unsubscribing', error);
  }).then(() => {
    //remove subscription on server
  );
```

------

### 理想很丰满，现实很骨感
> PWA目前还难以应用起来，看看PWA的核心技术在各大浏览器中的支持情况：
>   <div style="color:white;text-align:center"><div style="background:#39b54a;width:120px;display:inline-block;">Supported</div><div style="background:#c44230;width:120px;display:inline-block;">Not supported</div><div style="background:#A8BD04;width:120px;display:inline-block;">Partial support</div></div>
>
**[ServiceWorker][11]：**
![此处输入图片的描述][12]
>
**[Fetch][13]：**
![此处输入图片的描述][14]
>
**[Push API][15]：**
![此处输入图片的描述][16]

> [**More**][17]

------

附：[JavaScript API标准化情况][18]


  [1]: http://image.uisdc.com/wp-content/uploads/2014/12/texingfenxi.png
  [2]: https://developers.google.com/web/fundamentals/getting-started/codelabs/your-first-pwapp/
  [3]: http://om0gfsbon.bkt.clouddn.com/20170319148989360795452.png
  [4]: https://developers.google.com/web/fundamentals/getting-started/primers/service-workers
  [5]: http://om0gfsbon.bkt.clouddn.com/20170319148991086990501.png
  [6]: http://om0gfsbon.bkt.clouddn.com/20170319148991404983668.png
  [7]: https://jakearchibald.com/2014/offline-cookbook/
  [8]: https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/
  [9]: http://om0gfsbon.bkt.clouddn.com/20170319148991525144396.png
  [10]: http://om0gfsbon.bkt.clouddn.com/20170319148990193770114.png
  [11]: http://caniuse.com/#search=service%20worker
  [12]: http://om0gfsbon.bkt.clouddn.com/2017022714881615329564.png
  [13]: http://caniuse.com/#search=fetch
  [14]: http://om0gfsbon.bkt.clouddn.com/2017022714881615329564.png
  [15]: http://caniuse.com/#search=push%20api
  [16]: http://om0gfsbon.bkt.clouddn.com/20170227148816194567793.png
  [17]: https://jakearchibald.github.io/isserviceworkerready/
  [18]: https://www.w3.org/TR/#tr_Javascript_APIs