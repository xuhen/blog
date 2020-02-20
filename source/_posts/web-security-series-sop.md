---
title: 网络安全第四篇之SOP(同源策略)
date: 2020-02-20 21:35:47
tags:
    - web security
---


> 同源策略是浏览器的安全特性，限制一个域的网页和脚本和另一个域的资源的交互。因为浏览器能加载和展示来自其他域的资源；我们也可能在浏览器上打开多个网站标签；一个网页也能嵌入来自不同域的`iframes`。如果这一切的交互方式没有限制，那就会存在很大安全隐患。
>
> 同源策略通过禁止对来自不同源的资源进行读权限，而让网页更安全的。

## 1、什么是同源？

一个源是被协议（比如HTTP或者HTTPS）、端口和主机定义的。当两个`URL`的这三个值相同时，我们认为是同源的。比如，`http://www.example.com/foo`和`http://www.example.com/bar`是同源的，但是和`https://www.example.com/bar`是不同源的，因为协议不同。


## 2、非同源下什么是被允许，什么是被禁止的？

一般来说，嵌入非同源的资源是可以的，但是读取资源是禁止的。


|     |   |
|  ----  | ----  |
| iframes  | 跨域嵌入，通常是允许的（需要看X-Frame-Options指令的配置情况），但是跨域读取（用JavaScript去读取iframe嵌入的文件DOM结构）是被禁止的。 |
|||
|||
| CSS  | 跨域的CSS文件可以通过`<link>`元素嵌入，或者`@import`在css文件里引入。需要服务器返回正确的`Content-Type` |
|||
|||
| forms  | 跨域的`URL`可以做为action的值放在form表单里。网络应用可以把表单数据发送给跨域的地址。 |
|||
|||
| images | 嵌入跨域的图片到网页是被允许的。然而，读取跨域图片（比如用JavaScript读取跨域图片到canvas元素）是被禁止的。 |
|||
|||
| multimedia   | 跨域的video和audio可以嵌入`<video>`和`<audio>`元素 |
|||
|||
| script | 跨域脚本是可以被嵌入的；但是调用特定API是被禁止的（比如跨域发起的fetch请求） |


## 3、Clickjacking是什么，怎样防止？

将网站嵌入`iframe`，并且将透明的按钮覆盖在上面，指向其他网址的做法就叫 clickjacking。用户被诱导点击之后其实跳到其他地址了。

为了防止其他网站将你的网页嵌入`iframe`中，可以添加一条CSP指令`fram-ancestors`到HTTP头里。

另外也可以添加`X-Frame-Option`（已经被CSP指令替换了）到HTTP头里，选项请看MDN[MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

下面是`clickJacking`示例：

![clickJacking](https://web.dev/same-origin-policy/clickjacking.png)

## 链接

[Same-origin policy](https://web.dev/same-origin-policy/)