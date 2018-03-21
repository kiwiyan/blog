---
title: pwa-原谅js一生不羁放纵爱自由
date: 2018-03-19 22:18:42
tags: service worker
categories: pwa
commentes: true
---

## PWA

什么是PWA？
用Google Chrome 团队的开发人员 Alex Russell 的描述方式：
>**这些应用没有通过应用商店进行打包和部署，它们只是汲取了所需要的原生功能的网站而已。**

PWA不是前端UI框架，它是在普通网站的基础上，通过一些现代浏览器支持的api加强了网站的交互体验，比如网站可以离线缓存，添加桌面快捷方式等。

<!-- more -->

### 包含功能


- 响应式用户界面 – 适应任何环境：桌面电脑，智能手机，笔记本电脑，或者其他设备。
- 不依赖网络连接 – 通过 Service Workers 可以在离线或者网速极差的环境下工作。
- 类原生应用 – 有像原生应用般的交互和导航给用户原生应用般的体验，因为它是建立在 app shell model 上的。
- 持续更新 – 受益于 Service Worker 的更新进程，应用能够始终保持更新。
- 安全 – 通过 HTTPS 来提供服务来防止网络窥探，保证内容不被篡改。
- 可发现 – 得益于 W3C manifests 元数据和 Service Worker 的登记，让搜索引擎能够找到 web 应用。
- 再次访问 – 通过消息推送等特性让用户再次访问变得容易。
- 可安装 – 允许用户保留对他们有用的应用在主屏幕上，不需要通过应用商店。
- 可连接性 – 通过 URL 可以轻松分享应用，不用复杂的安装即可运行。
-  渐进增强 – 能够让每一位用户使用，无论用户使用什么浏览器，因为它是始终以渐进增强为原则。

### 主要功能点
- Service Worker
- 消息推送
- 添加到主屏幕
- service worker和页面的通信


### Service Worker
在页面安装成功后，运行于浏览器后台，可用于离线缓存、消息推送等功能。
特点：
- 网站必须使用 HTTPS，或开发时使用localhost域。
- 运行于浏览器后台，可以控制打开的作用域范围下所有的页面请求。
- 单独的作用域范围，单独的运行环境和执行线程。
- 不能操作页面 DOM。但可以通过事件机制来处理。

Service Worker从注册到激活经历这几个状态，即Service Worker的生命周期：install -> installed -> actvating -> Active -> Activated -> Redundant

#### 1.register 注册 
```js
/*页面脚本中*/
if ('serviceWorker' in navigator) {  
    navigator.serviceWorker.register('./service-worker.js')
    .then(function(registration) {
        console.log("Service Worker Registered", registration);
    })
    .catch(function(err) {
        console.log("Service Worker Failed to Register", err);
    })
} 
```
#### 2.install 安装

第一次注册service worker后会触发install事件，install时会缓存‘离线缓存文件’，这些文件都安装成功后，service worker才会安装成功，接着进入waiting或active状态。service安装失败则进入redundant状态。
安装时触发install事件：
```js
self.addEventListener('install', event => {
    // event.waitUtil 用于在安装成功之前执行一些预装逻辑
    // 但是建议只做一些轻量级和非常重要资源的缓存，减少安装失败的概率
    // 安装成功后 ServiceWorker 状态会从 installing 变为 installed
    event.waitUntil(
        // 使用 cache API 打开指定的 cache 文件
        caches.open(CACHE_NAME).then(cache => {
            console.log(cache);
            // 添加要缓存的资源列表
            return cache.addAll(urlsToCache);
        })
    );
});
```
#### 3.activate 激活

sw.js的内容有改动时，触发install事件，新的service worker完成安装后进入waiting状态，需要关闭所有已打开的相关页，新的service worker才能激活。
如果需要所有页面在sw.js更新后都能更新，如何做呢？
在install事件中执行skipWaitin，可进入activate阶段，接着执行clients.claim()更新所有客户端的sercive worker
```js
self.addEventListener('activate', evnet => event.waitUntil(
    Promise.all([
        // 更新客户端上的service worker
        clients.claim(),
        // 清理旧版本
        caches.keys().then(cacheList => Promise.all(
            cacheList.map(cacheName => {
                if (cacheName !== CACHE_NAME) {
                    caches.delete(cacheName);
                }
            })
        ))
    ])
));
```
sw.js文件可能有浏览器缓存问题，就算文件改变了，缓存可能导致service worker更新不及时？一般有如下方式解决此问题：
- 服务端设置sw.js文件的缓存不设置或设置较短
- 借助localStorage，判断版本，手动更新

#### 4.fetch 处理请求
service worker安装完成并激活后运行于浏览器后台，通过fetch拦截发出的请求，给出自定义的响应(所以才需要https的安全机制)。浏览器发出请求时，触发fetch事件。
![](/images/2018/3/pwa2.png)

添加 fetch 的事件监听器：
```js
/* In service-worker.js */
self.addEventListener('fetch', function (event) {  
  event.respondWith(
    caches.match(event.request)  // 检查传入的请求 URL 是否匹配当前缓存中存在的任何内容
    .then(function (response) {
      if (response) {   // 如果有 response 并且它不是 undefined 或 null 的话就将它返回
        return response;  // 否则只是如往常一样继续，通过网络获取预期的资源                        
      }
      return fetch(event.request);                 
    })
  );
});
```
#### 浏览器支持
PC端，浏览器除了IE不支持，以及safari最新版也开始支持了，其他现代浏览器基本都支持；移动端支持目前只有最新版浏览器支持。
![](/images/2018/3/pwa1.png)

### 消息推送

通过Notification api，浏览器首先会请求推送权限，用户允许后，可以推送消息。
```js
Notification.requestPermission().then(grant => {
    if (grant !== 'granted') {
        return;
    }
    const notification = new Notification("Hi，网络不给力哟", {
        body: '您的网络貌似离线了，不过在这里还能继续打开~',
        icon: './images/icons/msg.jpg'
    });

    notification.onclick = function() {
        notification.close();
    };
});
```

### 添加到主屏幕

设置manifest.json文件在桌面添加app的快捷方式图标。
```js
/* In manifet.json */
{
  "name": "Progressive Times web app", // 桌面图标的文本
  "short_name": "Progressive Times", // 简称
  "start_url": "/index.html", // 启动页
  "display": "standalone", // 去掉地址栏等，打开web应用看起来像一个独立的app
  "theme_color": "#FFDF00", // 地址栏的颜色
  "background_color": "#FFDF00", // 启动页的背景色
  "icons": [{
      "src": "homescreen.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "homescreen-144.png",
      "sizes": "144x144",
      "type": "image/png"
    }
  ]
}
```


### service worker和页面的通信

通过postMessage实现Service Worker和页面之间的通信。

```js
/* In service-worker.js */
// service worker监听来自页面的消息
self.addEventListener('message', function(ev) { 
    console.log(ev.data);
});
// service worker 发送消息到页面
self.clients.matchAll().then(clientList => { 
    clientList.forEach(client => {
        client.postMessage('Hi, I am send from Service worker！');
    })
});

/* 页面脚本中 */
// 页面发送消息给service worker
navigator.serviceWorker.controller.postMessage('some msg');  
// 页面监听来自service worker的消息
window.addEventListener('message', function(ev) { 
    console.log(ev.data);
})
```


###  online/offline 事件
浏览器监听断网事件
```js
window.addEventListener('offline', function() { 

})
```

PWA还有其他一些功能点，包括sync同步，分享等都能很好增加用户体验，随着浏览器普遍开始支持PWA的特性，PWA的发展前景还是很令人期待的。
PWA就这样又蚕食了原生App的一部分市场，原谅js一生不羁放纵爱自由。

