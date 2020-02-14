---
title: 阻止iphone页面的橡皮筋效果
date: 2020-02-14 18:09:24
tags:
    - mobile
---


> 移动端开发时，IOS微信浏览器，或者Safari浏览器，在滚动页面到顶部或底部都会看到橡皮筋效果。特别是在写Scroller这样的组件时，觉得体验太差了。
>
> 因为这是IOS手机加的特性，所以android手机不会有这个烦恼。

## 解决办法

首先我想到的是用 `e.preventDefault()` 这个方法，当滚动示阻止这个默认的行为，当加到body 上时整个页面就不会滚动了。而最好是只在两个临界值，去阻止默认行为较好，即页面的顶部和底部。

```
var startY,endY;
//记录手指触摸的起点坐标
document.body.addEventListener('touchstart', function (e) {
    startY = e.touches[0].pageY;
}, { passive: false });

document.body.addEventListener('touchmove', function (e) {
    endY = e.touches[0].pageY;     //记录手指触摸的移动中的坐标

    //手指下滑，页面到达顶端不能继续下滑
    if (endY > startY && self.el.scrollTop <= 0) {
        e.preventDefault();
    }
    //手指上滑，页面到达底部能继续上滑
    if (endY < startY && self.el.scrollTop + self.el.clientHeight >= self.el.scrollHeight) {
        e.preventDefault();
    }
}, { passive: false });

```

直接判断页面到达顶部或底部是不行的；因为到达顶部后，手指不论做什么方向的移动都会被阻止，所以我们要加个对移动方向的判断。