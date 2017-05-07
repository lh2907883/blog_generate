---
title: 关于XMLHttpRequest
date: 2017-05-07 12:37:45
tags: javascript
---
## XMLHttpRequest是什么
XMLHttpRequest 是一个[API](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest), 它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 [AJAX](https://developer.mozilla.org/zh-CN/docs/AJAX) 中被大量使用。

## 兼容性
![](https://lh2907883.github.io/store/blog/%E5%85%B3%E4%BA%8EXMLHttpRequest/1.png)
* IE8/IE9也完全不支持xhr对象, 不过可以使用[ActiveXObject](https://msdn.microsoft.com/zh-cn/library/7sw4ddf8%28v=vs.94%29.aspx)类生成xhr实例
```javascript
new window.ActiveXObject( "Microsoft.XMLHTTP" )
```
* IE10/IE11部分支持，不支持 xhr.responseType为json

## 如何使用
使用xhr给后台server发送请求,大概可以分下面几个步骤
### 1.实例化XMLHttpRequest对象
```javascript
var xhr = new XMLHttpRequest();
```

### 2.初始化一个请求
```javascript
xhr.open('POST', '/server', true);
```
``open(method, url [, async = true [, username = null [, password = null]]])``

async: 默认值为true，即为异步请求，若async=false，则为同步请求

### 3.设置xhr参数
#### 3.1 设置request header
1. 通过调用setRequestHeader方法,设置请求头,这一步必须在``xhr.open``之后调用
2. setRequestHeader可以调用多次，最终的值不会采用覆盖override的方式，而是采用追加append的方式
```javascript
xhr.setRequestHeader('X-Test', 'one');
xhr.setRequestHeader('X-Test', 'two');
// 最终request header中"X-Test"为: one, two
```

#### 3.2 设置response type
1. 可以设置``xhr.responseType``,下面是responseType支持的类型

| 值             | xhr.response 数据类型         |  说明                               |
| ------        | -------                       |   ------                           |
| ""            | String                        | 默认值(在不设置responseType时)        |
| "text"        | String                        |                                    |
| "document"    | [Document](https://developer.mozilla.org/zh-CN/docs/Web/API/Document) 对象                  | 希望返回 XML 格式数据时使用           |
| "json"        | javascript 对象                | 存在兼容性问题，IE10/IE11不支持       |
| "blob"        | [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)对象                       |                                    |
| "arrayBuffer" | [ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/API/ArrayBuffer)对象                |                                    |
客户端代码
```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', '/json');
xhr.responseType = 'json';
xhr.onreadystatechange = function(e) {
    if (this.readyState == 4 && this.status == 200) {
        //通过xhr.response可以拿到返回的json数据
        console.log(this.response);
    }
};
xhr.send();
```
服务端代码
```javascript
if (req.url == '/json') {
    res.writeHead(200, { 'content-type': 'application/json' });
    res.write('{"a": 123, "b": "dcs"}');
    res.end();
}
```

2. 通过调用``xhr.overrideMimeType``方法来重写response的content-type(不过这个方法我调用之后并没有起作用..)

#### 3.3 设置请求超时
可以设置``xhr.timeout``,来设置超时时间(单位毫秒),在xhr.send()方法调用后开始计时

#### 3.4 设置xhr状态变化时的回调
```javascript
xhr.onreadystatechange = function () {
    switch(xhr.readyState){
        case 0://UNSENT,open()方法没有调用
            break;
        case 1://OPENED,open()方法已被成功调用,send()方法还未被调用。
            break;
        case 2://HEADERS_RECEIVED,send()方法已经被调用, 响应头和响应状态已经返回
            break;
        case 3://LOADING,响应体下载中,responseText中已经获取了部分数据.
            break;
        case 4://DONE,整个请求过程已经完毕.
            break;
    }
}
```
#### 3.5 xhr的事件
| 事件                | 触发条件                          |
| -----              | -------                          |
| onreadystatechange |	每当xhr.readyState改变时触发；但xhr.readyState由非0值变为0时不触发。 |
| onloadstart	     |  调用xhr.send()方法后立即触发，若xhr.send()未被调用则不会触发此事件。   |
| onprogress         |	xhr.upload.onprogress在上传阶段(即xhr.send()之后，xhr.readystate=2之前)触发；xhr.onprogress在下载阶段（即xhr.readystate=3时）触发  |
| onload             |	当请求成功完成时触发，此时xhr.readystate=4   |
| onloadend	         |  当请求结束（包括请求成功和请求失败）时触发   |
| onabort            |	当调用xhr.abort()后触发   |
| ontimeout          |	xhr.timeout不等于0，由请求开始即onloadstart开始算起，当到达xhr.timeout所设置时间请求还未结束即onloadend，则触发此事件。   |
| onerror            |	在请求过程中，若发生Network error则会触发此事件（若发生Network error时，上传还没有结束，则会先触发xhr.upload.onerror，再触发xhr.onerror；若发生Network error时，上传已经结束，则只会触发xhr.onerror）。**注意**，只有发生了网络层级别的异常才会触发此事件，对于应用层级别的异常，如响应返回的xhr.statusCode是4xx时，并不属于Network error，所以不会触发onerror事件，而是会触发onload事件。    |

### 4.发送数据
```javascript
xhr.send(data);
```
* 对于GET/HEAD请求``send``方法通常不传参数
* 对于POST/DELETE/PUT请求,``xhr.send(data)``的参数data可以是以下几种类型：
    * [ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/API/ArrayBuffer)
    * [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)
    * [Document](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)
    * DOMString
    * [FormData](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)

来一个🌰

客户端代码
```javascript
var fileInputElement = document.getElementById('file')

var oMyForm = new FormData();

oMyForm.append("username", "dcs");
oMyForm.append("num", 123456); // 数字123456被立即转换成字符串"123456"

// fileInputElement中已经包含了用户所选择的文件
for (var i = 0; i < fileInputElement.files.length; i++) {
    oMyForm.append("file", fileInputElement.files[i]);
}

var oFileBody = '<a id="a"><b id="b">hey!</b></a>'; // Blob对象包含的文件内容
var oBlob = new Blob([oFileBody], {
    type: "text/xml"
});
oMyForm.append("blob", oBlob);

var xhr = new XMLHttpRequest();
xhr.upload.onprogress = function(event) {
    if (event.lengthComputable) {
        var completedPercent = event.loaded / event.total;
        console.log(completedPercent);
    }
}
xhr.open("POST", "/upload");
xhr.send(oMyForm);
```
服务端代码
```javascript
if (req.url == '/upload' && req.method.toLowerCase() == 'post') {
    req.on('data', function(buffer) {
        //会不时的触发data事件来接受客户端传递的数据
    })
    .on('end', function() {
        //数据传递完毕
    });
}
```
**注意**: 客户端的``xhr.upload.onprogress``表示的是上传进度触发的回调(进度取决于客户端的网络状况),参数``event.total``针对的是所有file的总大小, 而服务端的``req.on('data', function(){})``只是这次请求的数据流接受一定量的数据后触发的回调,两者之间没有直接关系

## 参考资料
* [你真的会使用XMLHttpRequest吗？](https://segmentfault.com/a/1190000004322487)
* [MDN XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
* [XMLHttpRequest2 新技巧](https://www.html5rocks.com/zh/tutorials/file/xhr2/)