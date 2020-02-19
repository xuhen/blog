---
title: 网络安全第三篇之CSP
date: 2020-02-17 20:45:45
tags:
    - web security
---

> 根据前两个文章的介绍，我们知道可WEB有很多不安全的地方。作为开发者的我们，会想法设法去阻止XSS、XSRF的发生，但是那会非常繁琐，而且极易疏漏个别细节，进而前功尽气。那CSP的出现感觉就是这个救星，可以带给我对WEB安全的更大控制权。


## 什么是CSP

CSP（Content Security Policy）是一个浏览器的标准，它允许我们控制在我们的页面：什么域名或子域名下的何种资源可以加载。


## CSP的白名单

之所以我们的网站会遭受XSS攻击，是浏览器不能区分网页正常的脚本和被攻击者注入的脚本。比如我们的页面需要加载 `https://apis.google.com/js/plusone.js `脚本，我们信任里面的代码。但是我们不能期望浏览器知道来自`apis.google.com`域下的代码很棒，而来自`apis.evil.example.com`域下的代码有问题。

为了避免浏览器不加区分的下载所有的资源。CSP定义了个Content-Security-Policy 的HTTP头允许我们创建白名单的域资源，指导浏览器只执行或展示来自这些白名单下的资源。

因为我们相信`apis.google.com`域下的代码，让我们定义一个规则，允许脚本来自这两个域下的代码得到执行：

```
Content-Security-Policy: script-src 'self' https://apis.google.com
```

估计我们也能猜到，`script-src`是一个指令，控制着一系列的脚本相关的特权。我们定义 'self'作为一个合法的域，`https://apis.google.com`作为另一个合法的域。这样，浏览器就会遵照我们的指示，从apis.google.com域下经过https下载和执行脚本，当然也会从当前的页面域名下载和执行脚本。

如果现在我们去下载其他非法域名下的脚本的话，浏览器就会报下面的错误了：

![csp error](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091303.png)

## CSP的其他规则

显然脚本资源是最大的安全隐患，CSP也提供非常多的规则指令供我们对资源的各个方面进行细粒度的控制。

* base-uri：限制能出现在`<base>`标签里的URs。
* child-src：列出能加载在页面的workers和嵌入的frame内容。比如`child-src https://youtube.com`允许我们加载来自YouTube的资源，而不是其他域名下的。
* connect-src：限制我们能连接的域（通过：XHR，WebSokets，和 EventSource）。
* font-src：允许加载的字体域名，Google的字体库可以这样定义`font-src https://themes.googleusercontent.com`。
* form-action：限制能放在`<form>`标签上提交的接口。
* fram-ancestors：指定域名下的资源可以嵌入当前页面，这个指令有效的标签有 `<frame>`，`<iframe>`，`<embed>`和`<applet>`，这个指令不能用在`<meta>`标签里。
* frame-src：失效了，请用child-src。
* img-src：页面图片能加载的域名。
* media-src：页面能加载音频和视频的域名。
* object-src：控制Flash和其他插件。
* plugin-types：限制页面能包含的插件类型。
* report-uri：当违反了CSP浏览器发送报告的链接地址，这个指令不能用在`<meta>`标签里。
* style-src：限制css的加载域名。
* upgrade-insecure-requests：指导浏览器资源改变协议：从HTTP到HTTPS。

默认，指令的规则没有限制，如果不指定font-src的规则，默认就是`font-src *`，也就是你能从任何地方加载字体。我们也能用`default-src`指令去重写那些以`-src`结尾的指令的默认行为。

但是下面的指令不能用 `default-src` 指令作为兜底：

* base-uri
* form-action
* frame-ancestors
* plugin-types
* report-uri
* sandbox

每个指令之间我们用分号隔开，同时我们要确保一种指令只写一次，如果写成下面的形式：

```
script-src http://host1.com; script-src https://host2.com
```

那么只有第一次出现的指令才有效，后出现的都会被忽略掉。下面的指令就指定了两个域下代码都合法：

```
script-src https://host1.com https://host2.com
```

比如说，我们有一个应用，从内容分发网络（CDN）加载所有的资源（`https://cdn.example.net`），同时你也知道不会需要嵌入内容和插件，那我们就可以像下面的方式定义指令：

```
Content-Security-Policy: default-src https://cdn.example.net; child-src 'none'; object-src 'none'
```

## CSP的实现细节

你可能会在网络的其他教程里看到还有 `X-WebKit-CSP`和 `X-Content-Security-Policy`头的实现，你应该忽略掉这些有前缀的表示方法。现代浏览器（IE除外）都支持不带前缀的 `Content-Security-Policy`的头。

每个指令里的域名列表都很灵活。你能只指定协议（data:,https:），也能指定域名（比如example.com，就匹配在这个域名下的所有协议和所有端口），当然我们也能指定全路径（`https://example.com:443`，这就只能匹配HTTPS协议下的example.com域名的443端口）。通配符也是可以的，但是只能是协议，端口和域名的最左边的部分：`*://*.example.com:*`就会匹配`example.com`的所有子域名（但是不包括它自己），所有协议，所有端口号。

下面的关键词在域名列表处也是可以的：

* 'none'：不匹配任何域名。
* 'self'：匹配当前页面的域名，但是子域名不匹配。
* 'unsafe-inline'：允许内联的JavaScript和CSS（稍后介绍）。
* 'unsafe-eval'：允许字符串到代码的转换机制比如 `eval` （稍后介绍）。

所有这四个关键词都需要加单引号，如果我们写成这样 `script-src self` 表示的允许加载来自名称为self的服务器上的JavaScript代码（而不是页面的当前域）。

## 沙盒

有一个指令值得一提：`sandbox`。它有异于其他的指令，sandbox是对页面能发生的行为做限制，而不是像其他指令对页面加载的资源做限制。如果sandbox指令存在的话，这个页面就会表现得像是在iframe标签上加载出来的，并且附带了sandbox属性。


## meta标签

CSPs推荐的是通过HTTP的头进行配置，但是有时我们也可以把这些规则直接内嵌在页面里。使用方式就是在meta标签里配合着`http-equiv`使用：

```
<meta http-equiv="Content-Security-Policy" content="defualt-src https://cdn.example.net; child-src 'none'; object-src 'none'">
```
但是`frame-ancestors`，`report-uri`，`sandbox`不能在meta标签里使用。


## 禁掉有害的内联代码

我们知道了CSP是让我们信赖的域名配到白名单下，这种方式让浏览器清楚的知道哪些域名下的资源是我们能接受的，哪些是我们会拒绝的。但是域名白名单机制，不能规避内嵌脚本注入的XSS攻击。如果一个攻击者能注入一段包含恶意代码的scritp标签如：`<script>sendMyDataToEvilDotCom();</script>`，浏览器也无法区分这段代码是恶意的。

CSP解决这个问题是通过禁掉所有内嵌脚本，不仅是内嵌的script标签会被禁掉，内嵌的事件处理函数和内嵌的代码如`javascript: URLs`也通通禁掉。因此我们需要把包在script标签内的代码移到单独的文件里，用`addEventListener`代替`javascript:URLs`和`<a ...onclick="[JAVASCRIPT]">`，如下：

```
<script>
function doAmazingThings() {
    alert('I AM AMAZING!')
}
</script>
<button onclick="doAmazingThings();"></button>
```

变成下面的形式：

```
<!-- amazing.html -->
<script src="amazing.js"></script>
<button id="amazing"></button>
```

```
// amazing.js
function doAmazingThings() {
    alert('I AM AMAZING!')
}

document.addEventListener('DOMContentReady', function() {
    document.getElementById('amazing').addEventListener('click', doAmazingThings);
});
```

重写后的代码就能符合CSP的指令工作机制了。不考虑CSP的情况下，已经是最佳实践了。

内联样式跟内联脚本类似，`style`属性和`style`标签里的的代码都应该放到单独的css文件里。

如果真的非得用到内联script和style，我们能通过添加`unsafe-inline`到`script-src`和`style-src`指令。另外我们也能用nonce和hash（下面会介绍）。但是非必须情况下尽量不要使用它们。因为禁掉内联脚本是CSP能提供的最大的安全策略，同样的禁掉内联样式能让我们的应用更安全。


## 内联脚本的防范

CSP Level 2 提供了向后兼容的内联脚本，允许我们用nonce(number used once)或者hash白名单（动词）具体的内联脚本。

使用nonce，给脚本标签一个nonce属性，它的值必须和我们信任的列表匹配至少一个。比如：

```
<script nonce=EDNnf03nceIOfn39fn3e9h3sdfa>
    // some inline code.
</scritp>
```

同时添加nonce到`script-src`指令里。

```
Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'
```

每次页面请求发生时，nonce都必须重新生成，并且其值是不可猜测的。

hash和nonce的工作方式差不多。但不是添加随机码到script标签，而是为script标签的内容创建SHA的hash值，然后把这个值放到`script-src`指令里。比如

```
<script>alert('Hello, World!');</script>
```

我们的规则里就得包含经过内容hash过的值：

```
Contont-Security-Policy: script-src 'sha256-qznLcsROx4GACP2dm0UCKCzCG-HiZ1guq6ZZDob_Tng='
```

我们需要做些说明：这个`sha*-`的前缀是生成hash的算法，在上面的例子就是用的sha256去生成的hash值。CSP也支持其他一些hash算法比如`sha384-`和`sha512-`。生成hash的规则是排除掉`<script>`和`</script>`这两个标签的内容部分。大小写敏感，空格符也会识别，包括行首空格和行尾空格。

## 字符串转代码的防范

即使黑客不能直接注入脚本，但可以给你的应用设下陷阱，比如把插入的字符串解析成JavaScritp代码并执行。`eval()`，`new Function()`，`setTimeout([string], ...)`，和`setInterval([string], ...)`都可能被利用，注入的字符串可能会执行恶意的代码。CSP的方案当然是完全的禁掉这些隐患。

这个方案要求我们在写代码是要注意以下几点：

* 通过`JSON.parse`解析JSON数据，不要再依赖`eval`去解析了。
* 用内联函数的方式去执行`setTimeout`和`setInterval`，而不是字符串。

```
setTimeout("document.querySelector('a').style.display = 'none'")
```

需要改写成：

```
setTimeout(function() {
    document.querySelector('a).style.display = 'none';
}, 10)
```

* 避免运行时的内联模板：许多模板库都用`new Function()`去加快运行时的模板生成。这是非常优雅的动态编程范式，但同时也有注入恶意字符串的危险。

如果你选择的模板语言能提供预编译，那就太好了（handlerbars就提供）。预编译模板能比最快的运行时实现方式更快，并且更安全。如果`eval`和其他`text-to-JavaScript`方式的函数对你的应用来说很重要。你能将`unsafe-eval`加到`script-src`指令去允许这些函数。


## 违规拦截上传

CSP能禁掉不可信来源的代码的能力非常棒，但如果它能把这些违规行为发到我们的日志收集器里，那就更好了。很幸运的是，我们确实能指导浏览器POST JSON格式的报告到指定的url地址，这个指令就是`report-uri`：

```
Content-Security-Policy: defalut-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
```

被发送的违规数据如下：

```
{
    "csp-report": {
        "document-uri": "http://example.org/page.html",
        "referer": "http://evil.example.com/",
        "blocked-uri": "http://evil.example.com/evil.js",
        "violated-directive": "script-src 'self' https://api.google.com",
        "original-policy": "script-src 'self' https://api.google.com; report-uri http://example.org/my_amazing_csp_report_parser"
    }
}
```

上面的报告包含了很多信息，能帮助我们找到具体的违规原因，包括在哪个页面有违规行为（document-uri），页面的反向链接地址（referrer），违规的资源（blocked-uri），具体被违反的指令（violated-directive），还有这个页面的完整规则（original-policy）.


## 仅仅只违规上传

如果你只是刚刚开始用CSP，那实施严厉的规则之前，先评估应用目前违规的状态，会更好。你能让浏览器监控规则，报到违规情况，但是不强加限制。那用`Content-Security-Policy-Report-Only`头就可以了，如下：

```
Content-Security-Policy-Report-Only: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
```
这些规则只是报告模式，也就是不会禁掉这些限制的资源，只是把这些违规记录发送到指定地址。

