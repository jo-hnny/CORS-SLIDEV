---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# 跨域相关问题探讨

-johnnyzwu

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# 主要内容
- 什么是跨域（跨源）
- 浏览器对跨域有什么限制
- 开发者又如何达到跨域的目的

---

# 什么是跨域（跨源）

Web 内容的源由用于访问它的 URL 的方案(协议)，主机(域名)和端口定义。只有当方案，主机和端口都匹配时，两个对象具有相同的起源。

## 思考

- http://example.com:80 与 http://example.com 之间是否跨域
- http://tiyan.oa.com/ 与 http://hr.oa.com/ 之间是否跨域

## notes (IE 中的特例)

- 端口：IE 未将端口号纳入到同源策略的检查中，因此 https://company.com:81/index.html 和 https://company.com/index.html 属于同源并且不受任何限制。
- 授信范围（Trust Zones）：两个相互之间高度互信的域名，如公司域名（corporate domains），则不受同源策略限制。

---

# 同源的例子

|                                                                             |                                                 |
| --------------------------------------------------------------------------- | ----------------------------------------------- |
| http://example.com/app1/index.html <br/> http://example.com/app2/index.html | 同源：相同的协议（http）和域名（example.com）   |
| http://example.com:80 <br/> http://example.com                              | 同源：因为服务器默认通过 80 端口 传输 http 内容 |

---

# 不同源的例子

|                                                                                |                  |
| ------------------------------------------------------------------------------ | ---------------- |
| http://example.com/app1 <br/> https://example.com/app2                         | 不同源：协议不同 |
| http://example.com <br/> http://www.example.com <br/> http://myapp.example.com | 不同源：域名不同 |
| http://example.com <br/> http://example.com:8080                               | 不同源：端口不同 |

---

### 浏览器对跨域有什么限制

## 同源策略

同源策略是一个重要的安全策略，它用于限制一个 origin 的文档或者它加载的脚本如何能与另一个源的资源进行交互。它能帮助阻隔恶意文档，减少可能被攻击的媒介。

## 限制

- Cookie、LocalStorage 和 IndexDB 无法读取。
- DOM 无法获得。
- AJAX 请求不能发送。

---

# cookie

两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置 document.domain 共享 Cookie。

## example

A:http://w1.example.com/a.html

B:http://w2.example.com/b.html

A:`document.cookie = "test1=hello";`

---

# DOM

如果两个窗口一级域名相同，只是二级域名不同，那么设置上一页介绍的 document.domain 属性，就可以规避同源政策，拿到 DOM。

## 对于完全不同源的网站，目前有三种方法，可以解决跨域窗口的通信问题。

- location.hash
- window.name
- 跨文档通信 API（Cross-document messaging）

---

# location.hash

## tencent.com/a.html

```html
<body>
  <iframe src="https://ali.com/b.html" id="iframe"></iframe>
  <script>
    //  监听 hash 变化
    window.addEventListener("hashchange", () => {
      console.log(location.hash);
    });
  </script>
</body>
```

## ali.com/b.html

```html
<body>
  <script>
    window.parent.location.hash = "some message";
  </script>
</body>
```

---

# window.name
name 属性可设置或返回存放窗口的名称的一个字符串，name值在不同的页面（包括域名改变）加载后依旧存在。

## example
- tencent.com/a.html
- tencent.com/b.html
- ali.com/c.html

---
layout: two-cols
---

## tencent.com/a.html

```html
<body>
  <iframe src="ali.com/c.html" onload="onload()" id="iframe"></iframe>
  <script>
    // iframe 加载完会调用 iframe， 防止src 改变出现死循环。
    let first = true;
    function onload() {
      if (first) {
        let iframe = document.getElementById("iframe");
        iframe.src = "tencent.com/b.html";
        first = false;
      } else {
        console.log(iframe.contentWindow.name); // 'test'
      }
    }
  </script>
</body>
```

::right::

## ali.com/c.html

```html
<body>
  <script>
    window.name = "test";
  </script>
</body>












```
---

# 跨文档通信 API（Cross-document messaging）
即 postMessage

## example
- tencent.com/a.html
- ali.com/b.html

---

# 父窗口向子窗口传递信息
## tencent.com/a.html

```html
<body>
    <iframe id="iframeB" src="//ali.com/b.html"></iframe>
    <script>
      const iFrame = document.querySelector("#iframeB");

      iFrame.onload = function () {
        iFrame.contentWindow.postMessage("MessageFromTencent", "*");
      };
    </script>
  </body>
```



## ali.com/a.html

```html
<body>
    <script>
      function receiveMessageFromIndex(event) {
        console.log("receiveMessageFromParent", event);
      }
      window.addEventListener("message", receiveMessageFromIndex, false);
    </script>
  </body>
```
---

# 子窗口向父窗口传递信息
## tencent.com/a.html

```html
<body>
    <iframe id="iframeB" src="//ali.com/b.html"></iframe>
    <script>
      function receiveMessageFromIndex(event) {
        console.log("receiveMessageFromParent", event);
      }
      window.addEventListener("message", receiveMessageFromIndex, false);
    </script>
  </body>
```



## ali.com/a.html

```html
<body>
    <script>
      parent.postMessage( {msg: 'MessageFromSon'}, '*');
    </script>
  </body>
```

---

# AJAX

- JSONP
- CORS

---

# JSONP

## client
```js
// 1.全局声明一个用来处理返回值的函数 fn，该函数参数为请求的返回结果。
function fn(result) {
  console.log(result);
}

// 2.将函数名与其他参数一并写入 URL 中。
let url = "http://www.b.com?callback=fn&params=...";
// 3.动态创建一个 script 标签，把 URL 赋值给 script 的 src属性。

let script = document.createElement("script");
script.setAttribute("type", "text/javascript");
script.src = url;
document.body.appendChild(script);
```

## server
```js
fn({
  list: [],
  ...
})
```

---

# CORS
CORS 是一个 W3C 标准，全称是"跨源资源共享"（Cross-origin resource sharing），通常也被译为跨域资源共享。

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

---

# 简单请求的判断条件
1. 请求方法是以下三种方法之一：
    - HEAD
    - GET
    - POST

2. HTTP的头信息不超出以下几种字段(用户可以自己设置的)：
   - Accept
   - Accept-Language
   - Content-Language
   - Last-Event-ID
   - Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

---

# 简单请求
![](https://mdn.mozillademos.org/files/17214/simple-req-updated.png)
---

# 简单请求的流程
1. 浏览器发出简单请求的时候，会在请求头部增加一个 Origin 字段，对应的值为当前请求的源信息。

2. 当服务端收到请求后，会根据请求头字段 Origin 做出判断后返回相应的内容。

3. 浏览器收到响应报文后会根据响应头部字段 Access-Control-Allow-Origin 进行判断，这个字段值为服务端允许跨域请求的源，其中通配符 * 表示允许所有跨域请求。如果头部信息没有包含 Access-Control-Allow-Origin 字段或者响应的头部字段 Access-Control-Allow-Origin 不允许当前源的请求，则会抛出错误。

---

# 非简单请求
<img class="preflight_correct" src="https://mdn.mozillademos.org/files/16753/preflight_correct.png" >

<style>
  .preflight_correct {
    width: auto; 
    height: 100%; 
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }

</style>

---

# 非简单请求流程
1. 浏览器在处理非简单的请求时，浏览器会先发出一个预检请求（Preflight）。这个预检请求为 OPTIONS 方法，并会添加了 1 个请求头部字段 Access-Control-Request-Method，值为跨域请求所使用的请求方法。
2. 在服务端收到预检请求后，除了在响应头部添加 Access-Control-Allow-Origin 字段之外，至少还会添加 Access-Control-Allow-Methods 字段来告诉浏览器服务端允许的请求方法，并返回 204 状态码。
3. 服务端还根据浏览器的 Access-Control-Request-Headers 字段回应了一个 Access-Control-Allow-Headers 字段，来告诉浏览器服务端允许的请求头部字段。
4. 浏览器得到预检请求响应的头部字段之后，会判断当前请求服务端是否在服务端许可范围之内，如果在则继续发送跨域请求，反之则直接报错。

---

# CORS常用头部字段

## origin
请求首部字段, Origin 指示了请求来自于哪个站点, 包括协议、域名、端口、不包括路径部分
在不携带凭证的情况下，可以使是一个*，表示接受任意域名的请求
## Access-Control-Allow-Origin
响应头，用来标识允许哪个域的请求
## Access-Control-Allow-Methods
响应头，用来标识允许哪些请求方法被允许
## access-control-allow-headers
响应首部， 用于预检请求中，列出了将会在正式请求的允许携带的请求头信息。
---

## Access-Control-Expose-Headers
响应头，用来告诉浏览器，服务器可以自定义哪些字段暴露给浏览器
## Access-Control-Allow-Credentials
是否允许携带Credentials,Credentials可以是 cookies, authorization headers 或 TLS client certificates。
## Access-Control-Max-Age
预检请求的缓存时长

---

# 附带身份凭证的请求

一般而言，对于跨源 XMLHttpRequest 或 Fetch 请求，浏览器不会发送身份凭证信息。如果要发送凭证信息，需要设置 XMLHttpRequest 的某个特殊标志位。
```js
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';

function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send();
  }
}
```
1. 如果服务器端的响应中未携带 Access-Control-Allow-Credentials: true ，浏览器将不会把响应内容返回给请求的发送者。
2. 对于附带身份凭证的请求，服务器不得设置 Access-Control-Allow-Origin 的值为“*”。

---

# 参考

1. [浏览器的同源策略-MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
2. [跨源资源共享（CORS）-MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)