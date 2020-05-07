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
xhr.onreadystatechange = function() {
	if (xhr.readyState === 4 && xhr.status === 200) {
	  var data = JSON.parse(xhr.responseText);

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
下面说明影响重排的属性
![reflow](http://xuheng.inject.top/images/reflow.png)


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
## Q. 实现call方法

```
Function.prototype.myOwnCall = function(someOtherThis) {
  someOtherThis = someOtherThis || global;
  var uniqueID = "00" + Math.random();
  while (someOtherThis.hasOwnProperty(uniqueID)) {
    uniqueID = "00" + Math.random();
  }
  someOtherThis[uniqueID] = this;
  const args = [];
  // arguments are saved in strings, using args
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push("arguments[" + i + "]");
  }

  // strings are reparsed into statements in the eval method
  // Here args automatically calls the Array.toString() method.
  var result = eval("someOtherThis[uniqueID](" + args + ")");
  delete someOtherThis[uniqueID];
  return result;
};
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

## Q. 前端异常捕获都有哪些方式？

```
A.
1. 对于一般的同步任务，我们可以使用try...catch

try {
  undefined.map(v => v);
} catch(e) {
  console.log(e); // TypeError: Cannot read property 'map' of undefined
}

2. 对于setTimeout等异步异常，可以使用window.onerror捕获

setTimeout(() => {
    undefined.map(v => v);
}, 1000);

window.onerror = (msg, url, row, col, error) => {
    console.log({ msg, url, row, col, error });
};

Note: 此方法不能捕获img和css资源的加载异常，比如404；
而window.addEventListener('error')方式可以捕获；
addEventListener不能捕获js的异常。


3. 对于未捕获的Promise异常，可以使用window.onunhandledrejection捕获：

window.addEventListener("unhandledrejection", e => {
    e.preventDefault();
    console.log(e);
});

Promise.reject('promiseError');

通过以上的三种捕获方式，我们就可以捕获到我们程序中所有的异常！
注意，如果你的代码是async/await形式的， 可以通过babel转化为Promise的形式，这样
window.onunhandledrejection依然可以捕获到相应的异常。


```

## Q. requestAnimationFrame 和 requestIdleCallback

![reflow](http://xuheng.inject.top/images/lifeofaframe.png)

## Q. 模拟new 操作符

```
function objectFactory() {
    var obj = new Object(),
        Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    var ret = Constructor.apply(obj, arguments);
    return typeof ret === 'object' ? ret : obj;

};


function Otaku(name, age) {
    this.name = name;
    this.age = age;

    this.habit = 'Games';
}

Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}


var person = objectFactory(Otaku, 'Kevin', '18')

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); 
```


## Q. 格式化现金

```
function format(numberStr) {
    return numberStr.replace(/(\d)(?=(\d{3})+$)/g, function (mc) {
        console.log(mc)
        return mc + ',';
    })
}

console.log(format('1234567'))
```

## Q. Commonjs模块打包到前端

```
(function (modules) {
      var installedModules = {};

      function require(moduleName) {
        //如果模块已经导入，那么直接返回它的exports
        if (installedModules[moduleName]) {
          return installedModules[moduleName].exports;
        }

        //模块初始化
        var module = installedModules[moduleName] = {
          exports: {},
          name: moduleName,
          loaded: false
        };

        //执行模块内部的代码，这里的 modules 变量即为我们在上面写好的 modules 对象
        modules[moduleName].call(module.exports, module, module.exports, require);
        //模块导入完成
        module.loaded = true;
        //将模块的exports返回
        return module.exports;
      }

      return require("entry");
    })({
      "entry": function (module, exports, require, global) {
        //index.js
        var module1 = require("./module1");
        var module2 = require("./module2");
        module1.foo();
        module2.foo();

        function hello() {
          console.log("Hello!");
        }

        module.exports = hello;
      },
      "./module1": function (module, exports, require, global) {
        var module2 = require("./module2");
        console.log("initialize module1");

        console.log("this is module2.foo() in module1:");
        module2.foo();
        console.log("\n")

        module.exports = {
          foo: function () {
            console.log("module1 foo !!!");
          }
        };
      },

      "./module2": function (module, exports, require, global) {
        console.log("initialize module2");
        module.exports = {
          foo: function () {
            console.log("module2 foo !!!");
          }
        };
      }
    })

```

Q. 关于js函数中this的指向问题

```
1. 我们知道，this 对象是在运行时基于函数的执行环境绑定的: 在全局函数中，this 等于 window，而当函数被作为某个对象的方法调用时，this 等于那个对象
2. 匿名函数的执行环境具有全局性，因此其 this 对象通常指向 window。但有时候 由于编写闭包的方式不同，这一点可能不会那么明显。

下面来看一个例子:
var name = "The Window";
var object = {
  name: "My Object",
  getNameFunc: function () {
    return function () {
      return this.name;
    };
  }
};
console.log(object.getNameFunc()());  //"The Window"(在非严格模式下)

为什么匿名函数没 有取得其包含作用域(或外部作用域)的 this 对象呢?

答： 每个函数在被调用时都会自动取得两个特殊变量:this 和 arguments。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，因此永远不可能直接访问外部函数中的这两个变量。
  不过，把外部作用域中的 this 对象保存在一个闭包能够访问到的变量里，就可以让闭包访问该对象了

var name = "The Window";
var object = {
    name: "My Object",
    getNameFunc: function () {
      var that = this;
      return function () {
        return that.name;
      };
    }
};

```


Q. 获取嵌套对象的属性

```
function get(o, path) {
    var paths = path.split(/[.[\],]/).filter(Boolean);
    return paths.reduce((ac, cur) => {
        return (ac && ac[cur] !== 'undefined') ? ac[cur] : undefined;
    }, o)
}


var obj = {
    a: {
        b: {
            c: 1

        },
        d: [
            {
                e: 1,
                f: 2
            }
        ]
    }
};


var r = get(obj, 'a.d[0].e');

console.log(r);
```

Q. js中的数据类型

```
1. 简单类型

var a = undefined;
var b = 12;
var c = true;
var d = '';
var e = function () { }
var f = {};
var g = Symbol();

console.log(
    typeof a,
    typeof b,
    typeof c,
    typeof d,
    typeof e,
    typeof f,
    typeof g,

);

// undefined number boolean string function object symbol

2. 复杂类型
Object
```

## Q. 动态加载脚本的通用方法

```
  function loadScript(url){
    var script = document.createElement("script"); 
    script.type = "text/javascript";
    script.src = url; 
    document.body.appendChild(script);
  }
  loadScript("client.js");

  function loadScriptString(code) {
    var script = document.createElement("script");
    script.type = "text/javascript";

    try {
      // IE可能报错
      script.appendChild(document.createTextNode(code));
    } catch (ex) {
      // 在IE中
      script.text = code;
    }
    document.body.appendChild(script);
  }
  loadScriptString("function sayHi(){alert('hi');}");
  sayHi()

  function loadStyles(url) {
    var link = document.createElement("link");
    link.rel = "stylesheet";
    link.type = "text/css";
    link.href = url;
    var head = document.getElementsByTagName("head")[0];
    head.appendChild(link);

  }
  loadStyles("style.css");

  function loadStyleString(css) {
    var style = document.createElement("style");
    style.type = "text/css";
    try {
      // IE可能报错
      style.appendChild(document.createTextNode(css));
    } catch (ex) {
      // IE中的fallback
      style.styleSheet.cssText = css;
    }
    var head = document.getElementsByTagName("head")[0];
    head.appendChild(style);
  }

  loadStyleString("body{background-color:red}");

  
```

## Q. script标签的defer和async的区别

```
两者都会并行下载，不会影响页面的解析。

defer 会按照顺序在 DOMContentLoaded 前按照页面出现顺序依次执行。

async 当脚本下载完后立即执行。（两者执行顺序不确定，执行阶段不确定，可能在 DOMContentLoaded 事件前或者后 ）
```

![reflow](http://xuheng.inject.top/images/defer.jpg)


## CSS/JS 阻塞 DOM 解析和渲染

https://harttle.land/2016/11/26/static-dom-render-blocking.html


## uncurrying

```
Function.prototype.uncurrying = function () {
    var self = this;
    return function () {
        var obj = Array.prototype.shift.call(arguments);
        return self.apply(obj, arguments);
    };
};


for (var i = 0, fn, ary = ['push', 'shift', 'forEach']; fn = ary[i++];) {
    Array[fn] = Array.prototype[fn].uncurrying();
};

var obj = {
    "length": 3,
    "0": 1,
    "1": 2,
    "2": 3
};

Array.push(obj, 4);
console.log(obj.length);

var first = Array.shift(obj); // 截取第一个元素
console.log(first); // 输出:1
console.log(obj);

Array.forEach(obj, function (i, n) {
    console.log(i, n); // 分别输出:0, 1, 2
});
```


## 惰性加载函数

```
var addEvent = function (elem, type, handler) {
  if (window.addEventListener) {
    addEvent = function (elem, type, handler) {
      elem.addEventListener(type, handler, false);
    }
  } else if (window.attachEvent) {
    addEvent = function (elem, type, handler) {
      elem.attachEvent('on' + type, handler)
    }
  }

  addEvent(elem, type, handler);
}


var btn = document.getElementById('execute')

addEvent(btn, 'click', function () {
  console.log('click');
});
```

## 高阶函数定义

```
1. 函数可以作为参数被传递;
2. 函数可以作为返回值输出。
```

## this 的指向

```
this 的指向分4种

1. 作为对象的方法调用
2. 作为普通函数调用
3. 构造器调用
4. Function.prototype.call 和 Function.prototype.apply 调用
```