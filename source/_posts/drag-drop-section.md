---
title: 拖动图片和文字
date: 2020-04-15 22:50:54
tags:
    - js
---

> 在一些后台管理项目中，经常会有拖动布局的需求，那让我们看下实现的原理。

在被拖动元素上触发的事件先后顺序：
1. dragstart 
2. drag
3. dragend

拖动到一个有效放置目标上会触发的事件顺序：

1. dragenter
2. dragover
3. dragleave 或 drop


dataTransfer 是事件对象的一个属性。它有两个方法

1. getData(); 可以获取两种数据类型： text 或 URL
2. setData();
下面是它们的使用方法：
```
setData 函数一般在dragstart事件处理程序中调用，设置传输的数据。
// 设置和接收文本
event.dataTransfer.setData('text', 'some text');
var text = event.dataTransfer.getData('text');

// 设置和接收URL
event.dataTransfer.setData('URL', 'http://www.baidu.com');
var url = event.dataTransfer.getData('URL');
```


```
// style.css
#droptarget {
    height: 300px;
    width: 300px;
    border: 1px solid red;
}

#dragtarget {
    margin-top: 20px;
    margin-bottom: 20px;
}

img {
    width: 100px;
}

// index.html

<div id="droptarget"></div>
<div id="dragtarget" draggable="true">helo world</div>

<img src='https://dn-coding-net-production-static.qbox.me/static/7a51352fa766f4176d7c4543339e0e98.png' />


// js
var droptarget = document.getElementById("droptarget");
var dragtarget = document.getElementById("dragtarget");

EventUtil.addHandler(droptarget, "dragover", function (event) {
    console.log('dragover');
    EventUtil.preventDefault(event);
});
EventUtil.addHandler(droptarget, "dragenter", function (event) {
    console.log('dragenter');
    EventUtil.preventDefault(event);
    // event.dataTransfer.dropEffect = 'copy';
});
EventUtil.addHandler(droptarget, "drop", function (event) {
    console.log('drop', event)
    EventUtil.preventDefault(event);
    var dataTransfer = event.dataTransfer;
    var text = dataTransfer.getData("Text");
    var url = dataTransfer.getData("url") || dataTransfer.getData("text/uri-list");
    console.log(url, text);
    var target = event.target;
    var childNode;
    if (text) {
        childNode = document.createTextNode(text);
    }
    if (url) {
        childNode = document.createElement("img");
        childNode.src = url;
    }
    target.appendChild(childNode);

});
EventUtil.addHandler(dragtarget, "dragstart", function (event) {
    event.dataTransfer.setData('text', 'some text')
    event.dataTransfer.effectAllowed = 'copy'
    console.log('dragstart');
    // EventUtil.preventDefault(event);
});
EventUtil.addHandler(dragtarget, "drag", function (event) {
    console.log('drag');
    // EventUtil.preventDefault(event);
});
EventUtil.addHandler(dragtarget, "dragend", function (event) {
    console.log('dragend', event)
    // EventUtil.preventDefault(event);
});

```