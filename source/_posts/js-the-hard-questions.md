---
title: 理解js的难点概念
date: 2019-11-17 18:56:28
tags:
---

## Q.history路由和hash路由的区别

```js
A:
1. history路由（怕刷新url）
    流览器新的api History API的引入，现在我们就有window.history对象提供的方法去操作流量器访问历史
    window.history.back()，window.history.forward()，还有window.history.pushState()。
    pushState会增加一条新的历史记录，而replaceState则会替换当前的历史记录。
2. hash路由
    需要监听哈希变化触发的事件 —— hashchange 事件
    window.onhashchange = function(event){
     	console.log(event.oldURL, event.newURL);
     	let hash = location.hash.slice(1);
     	document.body.style.color = hash;
    }
```

## Q. 描述下浏览器缓存机制
```
A:
缓存过期时间是根据几个headers来计算的:
1. 如果"Cache-control: max-age=N" 存在，那过期时间就是N
2. 上一个不存在，如果Expires存在，那就是它减去Date header就是过期时间
3. 如果前两都不存在，如果Last-Modified存在，那过期时间就是 (Date - Last-Modified) / 10

缓存验证：
1. ETags(强验证)： ETag header 是资源响应的一部分，
浏览器可以在以后的请求中加入If-None-Match 去验证缓存资源
2.Last-Modified(协商缓存)：如果在响应请求中有Last-Modified header ，
浏览器可以发起If-Modified-Since请求去验证缓存文件
```

## Q. 解释下ajax
```
var xhr = new XMLHttpRequest();
xhr.open("get", "url", true);
xhr.setRequestHeader("key", "value");
xhr.onload = function() {
	var data = JSON.parse(xhr.responseText);
	if (xhr.readyState === 4 && xhr.status === 200) {

	}
}
xhr.send(null);
```

## Q. 解释下jsonp怎样工作的，为什么它不是ajax
```
A:
JSON with Padding
AJAX无法跨域是受到“同源策略”的限制，但是带有src属性的标签
    如：<script>, <img> <iframe>是不受该政策限制的。
通常<script>引用静态js其实也可以引用动态资源，后台服务被访问后返回
一个函数调用形式的字符串，由于是字符串，因此在后台不会起到任何作用，
但到前台，就成了函数调用。
优点：兼容性好，能在古老的浏览器上实现
缺点：1. 只能 GET 不能 POST； 2. 二是存在安全隐患，脚本注入
<script>
var dosomething = function(data){
      console.log(data);
};
var url = "https://hengxu.gitee.io/json.txt?callback=dosomething";
var script = document.createElement('script');
script.setAttribute('src', url);
document.getElementsByTagName('body')[0].appendChild(script);
</script>

jsonp.txt
dosomething({
    "data":"JSONP Works"
});
```

## Q. 跨域的解决方案

```
目的：同源策略的目的，是为了保证用户信息的安全，防止恶意网站窃取数据。
1. 跨域资源共享（cors）
    * 客户端需要设置xhr属性withCredentials=true
    * 服务器端在response header设置两个字段
        Access-Control-Allow-Origin:domainName | *
	Access-Control-Allow-Credentials:true
2. jsonp
3. 服务器代理: 浏览器有跨域限制，但是服务器不存在跨域问题，
   所以可以由服务器请求所要域的资源再返回给客户端
4. 使用postMessage实现页面之间通信
   * 页面和新开的窗口数据交互
   * 多窗口的数据交互
   * 页面与所嵌套iframe之间的信息传递。

```

## Q. "use strict";是什么用它有啥利弊。

```
A:
作用：
1. 消除JavaScript语法的不合理、不严谨，减少怪异行为。
2. 消除代码运行的一些不安全之处，保证代码运行安全。
3. 提高编译器效率，增加运行速度。
4. 为未来新版JavaScript做好铺垫。
eg: * 全局变量必须用var显式声明，否则报错。
    * 禁止this关键字指向全局对象
    * 对一个对象的只读属性赋值，会报错。
    * 函数必须声明在顶层
	"use strict";
	if (true) {
	    function f() { } // 语法错误
	}
```

## Q. 哪些操作会引起浏览器重绘和重排？
```
重绘：屏幕的一部分要重画，比如某个css背景色变了，但是元素的几何尺寸没变。
重排：元素的几何尺寸变了，需要重新验证并计算。
    * 当增加、删除、修改DOM节点时，会导致重绘或重排。
    * 移动DOM位置，或有动画，会有重绘和重排。
    * 修改CSS样式
    * resize窗口
    * 修改网页默认字体，或字体大小。
Note: display: none，会触发重排；visibility: hidden只会触发重绘。
减少重绘或重排：
    * 不要一条一条改DOM的样式，用预定义好的css的class，去修改DOM上的className
    * clone DOM节点到内存里改完后再放到页面。
    * 为动画DOM使用fixed或absolute定位，这样修改CSS就不会引起页面重排了


```

## Q. 浏览器渲染过程
```
A:
   1. 处理HTML标记并构建DOM树。
   2. 处理CSS标记并构建CSSOM树。
   3. 将DOM与CSSOM合并成一个渲染树。
   4. 根据渲染树来布局，计算每个节点的几何信息。
   5. 将各个节点绘制到屏幕。

Note:
   * 默认情况下，CSS被视为阻塞渲染的资源，所以浏览器不会渲染任何已处理的内容，直到CSSOM构建完毕。
   * JavaScript 不仅可以读取和修改DOM属性，还可以读取和修改CSSOM属性。

```

## Q. 实现bind方法
```
A:
Function.prototype.bind = function(context){
  var args = Array.prototype.slice.call(arguments, 1),
  fn = this;
  return function(){
      var innerArgs = Array.prototype.slice.call(arguments);
      var finalArgs = args.concat(innerArgs);
      return fn.apply(context,finalArgs);
  };
};
eg:
var obj = {
	name: "Edward",
	age: 25,
	tellInfo: function() {
		setTimeout(function(p1, p2, p3) {
			console.log(p1, p2, p3);
			console.log(this.name, this.age);
		}.bind(this, "hello", "world", 23), 0);
	},
	info: function() {
		return function(p1, p2) {
			console.log(this.name, this.age, p1, p2);
		}.bind(this, "hello", "world")
	}
}
```

## Q. HTTP2.0和HTTP1.X相比的新特性
```
1. 新的二进制格式（Binary Format）HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，
文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。
基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。
2. 多路复用（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。
一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，
接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
3. header压缩。HTTP1.x的header很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。
4. 服务端推送（server push），采用了SPDY的网页，例如我的网页有一个sytle.css的请求，在客户端收到sytle.css数据的同时，
服务端会将sytle.js的文件推送给客户端，当客户端再次尝试获取sytle.js时就可以直接从缓存中获取到，不用再发请求了。
```

## Q. 以浏览器为例解释进程(process)和线程(thread).
```
A.
1. 浏览器设置 UI 渲染线程与 JavaScript 引擎线程为互斥的关系，当 JavaScript 引擎线程执行时 UI 渲染线程会被挂起，
UI 更新会被保存在一个队列中等到 JavaScript 引擎线程空闲时立即被执行。

前端某些任务是非常耗时的，比如网络请求，定时器和事件监听，如果让他们和别的任务一样，都老老实实的排队等待执行的话，
执行效率会非常的低，甚至导致页面的假死。所以浏览器是多线程的，除了之前介绍的两个互斥的呈现引擎和 JavaScript 解释器，
浏览器一般还会实现这几个线程：浏览器事件触发线程，定时触发器线程以及异步 HTTP 请求线程。

    *浏览器事件触发线程：
	当一个事件被触发时该线程会把事件添加到待处理队列的队尾，等待 JavaScript 引擎的处理。
	这些事件可以是当前执行的代码块如定时任务、也可来自浏览器内核的其他线程如鼠标点击、AJAX 异步请求等，
	但由于 JavaScript 的单线程关系所有这些事件都得排队等待 JavaScript 引擎处理；
    *定时触发器线程：
	浏览器定时计数器并不是由 JavaScript 引擎计数的, 因为 JavaScript 引擎是单线程的,
	如果处于阻塞线程状态就会影响记计时的准确, 因此通过单独线程来计时并触发定时是更为合理的方案；
    *异步 HTTP 请求线程：
	XMLHttpRequest 在连接后是通过浏览器新开一个线程请求，将检测到状态变更时，如果设置有回调函数，
	异步线程就产生状态变更事件放到 JavaScript 引擎的处理队列中等待处理；

```