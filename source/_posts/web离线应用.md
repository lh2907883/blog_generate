---
title: web离线应用
tags: javascript
date: 2017-05-14 15:57:53
---

## 什么是web离线应用
在正常情况下,客户端使用HTTP协议通过网络得到服务器的资源,然后展示,但是如果在网络断开的情况下,客户端就没办法了,这时就需要一种技术去解决这一问题,目前有两种方式:[AppCache](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache)和[Service Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers#Browser_compatibility),他们都是通过在断网时读取本地缓存资源来实现web应用的离线访问的.

## 比较AppCache和Service Workers
### 使用方式
AppCache通过指定缓存清单文件来设置哪些资源需要被缓存

Service Workers则有更新强大的API来通过脚本精准控制缓存(包括更新,追加,删除缓存)

### 兼容性
下图是各大浏览器对AppCache的支持情况,可以看出基本都支持
![](https://lh2907883.github.io/store/blog/web%E7%A6%BB%E7%BA%BF%E5%BA%94%E7%94%A8/1.png)

而Service Workers还是有一部分浏览器不支持的(特别是移动平台)
![](https://lh2907883.github.io/store/blog/web%E7%A6%BB%E7%BA%BF%E5%BA%94%E7%94%A8/2.png)

### W3C标准
如果你看过[AppCache](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache)的文档,你会发现
> 该特性已经从Web标准中删除，虽然一些浏览器目前仍然支持它，但也许会在未来的某个时间停止支持，请尽量不要使用该特性。

> 在此刻使用这里描述的应用程序缓存功能高度不鼓励; 它正在处于从Web平台中被删除的过程。请改用[Service Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers#Browser_compatibility)代替。

我个人认为Service Workers各方面都要优于AppCache,除了兼容性,不过既然Service Workers已经是W3C标准,想必未来的支持情况应该会好一些,所以下面我们将详细讨论Service Workers的用法

## 使用Service Workers的前提条件
> 在已经支持 serivce workers 的浏览器的版本中, 很多特性没有默认开启, 需要开启一下浏览器的相关配置：
> * Firefox Nightly: 访问 about:config 并设置 dom.serviceWorkers.enabled 的值为 true; 重启浏览器；
> * Chrome Canary: 访问 chrome://flags 并开启 experimental-web-platform-features; 重启浏览器 (注意：有些特性在Chrome中没有默认开放支持)；
> * Opera: 访问 opera://flags 并开启 ServiceWorker 的支持; 重启浏览器。 

> 另外，你需要通过 **HTTPS** 来访问你的页面 — 出于安全原因，Service Workers 要求要在必须在 HTTPS 下才能运行。Github 是个用来测试的好地方，因为它就支持HTTPS。为了便于本地开发，localhost 也被浏览器认为是安全源。

## 如何使用
### 注册service worker
```javascript
/**
 * @scriptURL ServiceWorker脚本资源路径(可以是绝对或者相对当前URL的路径)
 * @options {scope: './'} scope是一个路径,默认值就是'./',它总是相对于ServiceWorker脚本资源路径的,scope指定了一个ServiceWorker的生效范围,只有在这个路径范围内的资源才能支持离线访问(所以跨域的资源是没法离线访问的)
 */
ServiceWorkerContainer.register(scriptURL, options)
    .then(
        function(ServiceWorkerRegistration) {
            // do something
        }
);
```
代码如下:
```javascript
//兼容性判断
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js', {scope: './'}).then(function(reg) {
        if (reg.installing) {
            console.log('Service worker installing');
        } else if (reg.waiting) {
            console.log('Service worker installed');
        } else if (reg.active) {
            console.log('Service worker active');
        }
    }).catch(function(error) {
        // registration failed
        console.log('Registration failed with ' + error);
    });
}
```
事实上当浏览器运行上面的代码时, 会先检测是否之前就获取并注册过**sw.js**
* 如果是**首次运行**,那就先下载并保存在本地, chrome中可以在*Application -> Service Workers*面板中查看
![](https://lh2907883.github.io/store/blog/web%E7%A6%BB%E7%BA%BF%E5%BA%94%E7%94%A8/3.png)
这时会触发Service Worker的``install``事件,之后会进入``navigator.serviceWorker.register``返回的``Promise.then``,状态``state``为``installing``. 
* 如果发现本地已经有**sw.js** *(即使本地的**sw.js**不是最新代码,这次访问任然会使用本地**sw.js**的代码)*,那么会直接进入``navigator.serviceWorker.register``返回的``Promise.then``,状态``state``为``activated``,这时Service Worker为已激活状态,后面只要是有``scope``范围内的请求都会被Service Worker的``fetch``事件拦截*(注意:如果在scope下的html页面访问了跨域的资源,比如图片什么的,那这个跨域资源请求也会被拦截)*,在``fetch``的回调函数里面你可以自行控制是**读缓存**,还是**请求网络**,你甚至可以**伪造响应数据** *(我想这也是为什么Service Worker只支持HTTPS的原因)*
* 如果发现本地已经有**sw.js**并且本地的**sw.js**不是最新代码,那么**sw.js**会自动更新,并触发Service Worker的``install``事件,在下次访问页面时将使用新版本的**sw.js**.

### 关于sw.js
#### sw.js的职责
从上面的描述,我们可以看出**sw.js**的职责其实是控制web应用中资源的访问策略,他和具体的业务逻辑,UI操作都没有任何关系,官方说明:
> 如果注册成功，service worker 就在 [ServiceWorkerGlobalScope](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerGlobalScope) 环境中运行； 这是一个特殊类型的 woker 上下文运行环境，与主运行线程（执行脚本）相独立，同时也没有访问 DOM 的能力。

#### 具体实现
* 注册install事件
```javascript
this.addEventListener('install', function(event) {
    //console.log('sw.js install');
    event.waitUntil(
        caches.open('v1').then(function(cache) {
            return cache.addAll([
                '/demo/service-workers/',
                '/demo/service-workers/index.html',
                '/demo/service-workers/js/main.js',
                '/demo/service-workers/json/data.json',
                '/demo/service-workers/images/1.jpg',
                '/demo/service-workers/images/2.jpg',
            ]);
        })
    );
});
```
> install 事件一般是被用来填充你的浏览器的离线缓存能力。为了达成这个目的，我们使用了 Service Worker 的 新的标志性的存储 [API — cache](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) — 一个 service worker 上的全局对象，它使我们可以存储网络响应发来的资源，并且根据它们的请求来生成key。这个 API 和浏览器的标准的缓存工作原理很相似，但是是特定你的域的。它会一直持久存在，直到你告诉它不再存储，你拥有全部的控制权。
> 注意:  Cache API  并不被每个浏览器支持。（查看 Browser support  部分了解更多信息。） 如果你现在就想使用它，可以考虑采用一个 polyfill，比如 [Google topeka demo](https://github.com/Polymer/topeka/blob/master/sw.js)，或者把你的资源存储在 [IndexedDB](https://developer.mozilla.org/zh-CN/docs/Glossary/IndexedDB) 中。

缓存方案有很多种, 示例代码中采用``Cache``, 值得注意的是``cache.addAll``可不仅仅只是记录需要缓存的资源URL列表, 它同时还会``fetch``这些资源把响应数据缓存起来,以便后面访问的时候使用 *(如果是一个复杂的web应用,个人觉得有些不太合理,谁又会在页面刚刚进来时就加载所有资源呢?)*

* 注册fetch事件
```javascript
this.addEventListener('fetch', function(event) {
    //console.log(event.request);
    //console.log(caches);
    event.respondWith(
        caches.match(event.request)
        .catch(function() {
            return fetch(event.request);
        }).then(function(response) {
            //console.log(response);
            caches.open('v1').then(function(cache) {
                //console.log(cache);
                cache.put(event.request, response);
            });
            return response.clone();
        })
    );
});
```
1. ``event.respondWith``接受一个返回response响应的Promise
2. ``caches.match``会尝试在缓存中匹配当前fetch的请求,匹配到了就直接返回response
3. ``cache.put``则是更新缓存

* 注册activate事件
```javascript
this.addEventListener('activate', function(event) {
    var cacheWhitelist = ['v2'];

    event.waitUntil(
        caches.keys().then(function(keyList) {
            return Promise.all(keyList.map(function(key) {
                if (cacheWhitelist.indexOf(key) === -1) {
                    return caches.delete(key);
                }
            }));
        })
    );
});
```
activate事件回调一般被用来删除旧的缓存

#### 改进后的代码
```javascript
this.addEventListener('install', function(event) {
    console.log('sw.js install');
    event.waitUntil(
        //这里先不往缓存中写
        caches.open('v1').then(function(cache) {
            return cache.addAll([
                // '/demo/service-workers/',
                // '/demo/service-workers/index.html',
                // '/demo/service-workers/js/main.js',
                // '/demo/service-workers/json/data.json',
                // '/demo/service-workers/images/1.jpg',
                // '/demo/service-workers/images/2.jpg',
                // '/demo/service-workers/child.html',
            ]);
        })
    );
});
this.addEventListener('fetch', function(event) {
    event.respondWith(
        caches.match(event.request)
        .then(function(response) {
            if (response) {
                //如果匹配到了
                return response;
            } else {
                //如果有资源没有匹配(比如child.html里面的跨域图片),就先发请求访问,然后缓存一份
                console.log('url not match;' + event.request.url);
                return fetch(event.request).then(function(response) {
                    console.log(response);
                    caches.open('v1').then(function(cache) {
                        cache.put(event.request, response);
                    });
                    return response.clone();
                });
            }
        })
        .catch(function() {
            return fetch(event.request);
        })
    );
});
this.addEventListener('activate', function(event) {
    console.log('activate')
});
```
[演示地址](https://lh2907883.github.io/store/demo/service-workers/)

## 参考资料
* [MDN 使用 Service Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers)
