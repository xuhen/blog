---
title: 网络安全第一篇之xss
date: 2020-02-15 14:18:31
tags:
    - web security
---

> 网络的世界里，永远没有绝对的安全；只有时刻跟踪当前的网络安全的最佳实践，才能为用户的沟通加上相对安全的保障锁。那我们就聊聊今天的话题 XSS

## 1、什么是XSS ？

XSS 全称是Cross Site Scripting (跨站脚本)， 为了和“CSS” 区分开，故简称 XSS。是指黑客往HTML文件中或DOM中注入恶意脚本，使之在用户的浏览器上运行这些恶意脚本，以获取用户的敏感信息如 Cookie、SessionId等，进而危害数据安全。

以下方式都可能产生 XSS:

* 来自用户输入的信息
* 来自第三方的链接
* URL参数
* POST参数
* Referer (可能来自不可信的来源)
* Cookie (来自其他子域注入)

常见的XSS攻击有三种：反射型、DOM-based型和储存型。其中反射型、DOM-based 型可以归类为非持久型攻击，储存型归为持久型攻击。

## 2、反射型

反射型XSS一般是攻击者通过特定的手法（如电子邮件），诱使用户去访问一个包含恶意代码的URL，当受害者点击这些专门设计的链接时，恶意代码会直接在受害者的浏览器执行。

对于访问者而言是一次性的，表现在我们把恶意脚本通过URL的方式传递给了服务器，而服务器则是不加处理的把脚本 “反射” 回生成的页面代码内，从而使用户的浏览器访问了这个包含恶意脚本的页面。反射型XSS的触发有后端的参与，要避免反射型的XSS，必须同后端协调，后端解析前端的数据是首先做相关的字符串检查和转义处理。

此类XSS通常出现在网站的搜索栏、用户的登录表单页，常用来窃取客户端的Cookies或进行钓鱼欺骗。

整个攻击过程如下：

![xss](https://p4.ssl.qhimg.com/t01a98b7f389ece8c01.png)

下面是反射型的代码实现:

server.js
```
const express = require('express');
const router = express.Router();

router.get('/', (req, res, next) => {
    res.render('index', {
        title: 'Express',
        xss: req.query.xss,
    })
});

module.exports = router;
```

index.eje
```
<!DOCUMENT html>
<html>
<head>
    <title><%= title %> </title>
</head>
<body>
    <h1><%= title %></h1>
    <p>Weclome to <%= title %></p>
    <div> <%- xss %> </div>
</body>
</html>
```

当我们在浏览器打开如下链接，就会执行script间的而已代码了。

```
http://localhost:3000/?xss=<script>alert('你被 xss 攻击了')</script>
```


## 3、DOM-based 型

客户端的脚本程序可以动态的修改页面的内容，而不依赖于服务器端的数据。例如客户端从URL中提取数据并在本地执行，如果用户再客户端输入的数据包含了恶意的JavaScript脚本，而这些脚本未经适当过滤，那么应用程序就可能受到DOM-based XSS攻击。

整个攻击过程如下：

![xss](https://p4.ssl.qhimg.com/t0165dddb475fc1e83c.png)

## 4、储存型

攻击者事先将恶意的代码上传或者储存到漏洞服务器，只要受害者浏览器包含此恶意代码的页面就会执行这些恶意代码。这也就意味着访问了这个页面的用户都会执行这段恶意代码，因此，储存型的XSS危害更大。

储存型XSS一般出现在网站留言、评论、博客日志，恶意脚本储存在服务端的数据库中。

整个攻击过程如下：

![xss](https://p0.ssl.qhimg.com/t017b61091f1c67ca05.png)

## 5、预防储存型和反射型XSS攻击

储存型和反射型XSS都是在服务器端取出恶意代码后，插入到响应HTML里，攻击者刻意编写的“数据”被内嵌到“代码”中，被浏览器执行。

预防这两种漏洞，有两种方式：

* 改成纯前端渲染，把代码和数据分隔开
* 对HTML做充分的转义

### 5-1 纯前端渲染

纯前端渲染的过程：

1. 浏览器先加载一个静态的HTML，此HTML中不包含任何跟业务相关的数据。
2. 然后浏览器执行HTML中的JavaScript。
3. JavaScript 通过Ajax加载业务数据，调用DOM API更新到页面上。

在纯前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本 `dom.innerText`，还是属性 `dom.setAttribute`，或是样式`dom.style`等等。因此浏览器不会被轻易欺骗，执行预期意外的代码了。

但纯前端渲染还需要注意避免 DOM型XSS漏洞。

### 5-2 转义HTML

如果拼接HTML是必要的，就要采用合适的转义库，对模板各处插入点进行充分转义。常用的模板引擎对HTML通常只有一个规则，就是把 `& < > " ' /` 这几个字符转义掉，确实能起到一定的XSS防护作用，但并不完善。

所以要完善的XSS防护措施，我们要使用更完善细致的转义策略。

例如Java工程里：

```
<!-- HTML 标签内文字内容 -->
<div><%= Encode.forHtml(UNTRUSTED) %></div>

<!-- HTML 标签属性值 -->
<input value="<%= Encode.forHtml(UNTRUSTED) %>" />

<!-- CSS 属性值 -->
<div style="width:<= Encode.forCssString(UNTRUSTED) %>">

<!-- CSS URL -->
<div style="background:<= Encode.forCssUrl(UNTRUSTED) %>">

<!-- JavaScript 内联代码块 -->
<script>
  var msg = "<%= Encode.forJavaScript(UNTRUSTED) %>";
  alert(msg);
</script>

<!-- JavaScript 内联代码块内嵌 JSON -->
<script>
var __INITIAL_STATE__ = JSON.parse('<%= Encoder.forJavaScript(data.to_json) %>');
</script>

<!-- HTML 标签内联监听器 -->
<button
  onclick="alert('<%= Encode.forJavaScript(UNTRUSTED) %>');">
  click me
</button>

<!-- URL 参数 -->
<a href="/search?value=<%= Encode.forUriComponent(UNTRUSTED) %>&order=1#top">

<!-- URL 路径 -->
<a href="/page/<%= Encode.forUriComponent(UNTRUSTED) %>">

<!--
  URL.
  注意：要根据项目情况进行过滤，禁止掉 "javascript:" 链接、非法 scheme 等
-->
<a href='<%=
  urlValidator.isValid(UNTRUSTED) ?
    Encode.forHtml(UNTRUSTED) :
    "/404"
%>'>
  link
</a>
```

可见，HTML的编码是十分复杂的，在不同的上下文里要使用不同的转义规则。

## 6、预防DOM-based XXS攻击

DOM型XXS攻击，实际上就是网站的JavaScript代码本身不够严谨，把不可信的数据当代码执行了。

在使用 `dom.innerHTML`、`dom.outerHTML`、`document.write()`时要特别小心，不要把不可信的数据作为HTML片段插入页码，而应尽量使用 `dom.textContent`、`dom.setAttribute`等等。

在Vue/React技术栈，不使用 `v-html`/`dangerouslySetInnerHTML`功能，就需要在前端render阶段避免 `innerHTML`、`outerHTML`的XSS隐患。

DOM中的内联事件监听器，如 `location`、`onclick`、`onerror`、`onload`、`onmouseover`等；`<a>`标签的`href`属性；JavaScript的 `eval()`、`setTimeout()`、`setInterval()` 等，都能把字符串最为代码运行。如果不可信的数据拼接到字符串中传递给这些API，很容易产生安全隐患，切勿避免。

```
<!-- 内联事件监听器中包含恶意代码 -->
<button onclick="UNTRUSTED">click me</button>
<!-- 链接内包含恶意代码 -->
<a href="UNTRUSTED">1</a>

<script>
// setTimeout()/setInterval() 中调用恶意代码
setTimeout("UNTRUSTED")
setInterval("UNTRUSTED")

// location 调用恶意代码
location.href = 'UNTRUSTED'

// eval() 中调用恶意代码
eval("UNTRUSTED")
</script>
```

如果项目中用到这些，一定要避免在字符串中拼接不可信的数据。

## 7、其他XSS防范措施

虽然在渲染页面和执行JavaScript时，通常谨慎的转义可以防止XSS的发生，但完全依赖开发的谨慎是不够的。以下介绍一些通用的方案，可以降低遭受XSS后带来的后果。

### 7-1、Content Security Plicy

严格的 CSP在XSS的防范中可以起到下列作用：

* 禁止加载外域代码，防止复杂的攻击逻辑。
* 禁止外域提交，网站被攻击后，用户的数据不会泄露带外域。
* 禁止内联脚本执行（规则比较严）
* 禁止未授权的脚本执行
* 合理使用上报可以及时发现XSS，利于尽快修复问题

### 7-2、输入内容长度控制

对不受信任的输入，都应该限制一个合理的长度。虽然无法完全防止XSS的发生，但可以增加XSS攻击的难度。

### 7-3、其他安全措施

* HTTP-only Cookie: 禁止JavaScript读取某些敏感的Cookie字段，攻击者完成XSS后也无法窃取此Cookie。
* 验证码：防止脚本冒出用户提交危险操作。


## 结束语

虽然很难通过技术手段完全避免XSS，但我们可以遵循以下原则减少漏洞的产生：

* 利用模板引擎：开启模板引擎自带的HTML转义功能。
* 避免内联事件：尽量不要使用拼接内联事件的写法。
* 避免拼接HTML：前端使用拼接HTML的方法比较危险，如果允许尽量使用 `createElement`、`setAttribute`这样的方法实现。
* 时刻保持警惕：在插入位置为DOM属性、链接时，要严加防范。
* 增加攻击难度，降低攻击后果：通过CSP、输入长度配置、接口安全措施等，增加攻击难度，降低攻击后果。
* 主动检测和发现： 可使用XSS攻击字符串和自动扫描工具寻找潜在的XSS漏洞。

## 链接

[Web 安全漏洞之 XSS 攻击](https://cnodejs.org/topic/5bf21601e6481c5709f5d00f)

[前端安全系列](https://tech.meituan.com/2018/09/27/fe-security.html)

