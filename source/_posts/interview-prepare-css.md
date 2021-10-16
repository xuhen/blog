---
title: 面试CSS部分的准备
date: 2020-05-26 20:13:45
tags:
    - CSS
---


## 画三角形

```
html
<div class="arrow"></div>

css
.arrow {
    width: 0;
    height: 0;
    border-top: 40px solid transparent;
    border-left: 40px solid transparent;
    border-bottom: 40px solid transparent;
    border-right: 40px solid green;
}
```

## CSS 的选择器有哪些

```
1. id选择器(#myid)
2. 类选择器(.myclass)
3. 标签选择器(div, h1, p)
4. 相邻选择器(h1 + p)
5. 子选择器(ul > li)
6. 后代选择器(li a)
7. 属性选择器(a[rel="external"])
8. 通配符选择器 (*)
9. 伪类选择器(a:hover, li:nth-child)
10.伪元素选择器 (div:after)

可继承的属性： font-size, font-family, color
不可继承的属性： border, padding, margin, width, height
```

## CSS优先级如何计算

```
!important(10000) > 内联样式(1000) > id(100) > class|伪类|属性选择(10) > 标签|伪元素(1) > 通配符(0) > 继承(无)
```

## CSS的伪类有哪些

```
p:first-of-type 选择属于其父元素的首个元素
p:last-of-type 选择属于其父元素的最后元素
p:only-of-type 选择属于其父元素唯一的元素
p:only-child 选择属于其父元素的唯一子元素
p:nth-child(2) 选择属于其父元素的第二个子元素
:enabled :disabled 表单控件的禁用状态。
:checked 单选框或复选框被选中。
```

## margin重合的问题

在重合元素外包裹一层容器，并触发该容器生成一个BFC。

```
html
<div class="aside"></div>
<div class="text">
    <div class="main"></div>
</div>

css
.aside {
    margin-bottom: 100px;
    width: 100px;
    height: 150px;
    background: #f66;
}

.main {
    margin-top: 100px;
    height: 200px;
    background: #fcc;
}

.text {
    /*盒子main的外面包一个div，通过改变此div的属性使两个盒子分属于两个不同的BFC，以此来阻止margin重叠*/
    overflow: hidden;
    /*此时已经触发了BFC属性。*/
}

```

## 等比列缩放的盒子

```
<div class="father">
    <div class="child"></div>
</div>

.father {
    width: 100%;
}

.child {
    width: 40%;
    height: 0;
    padding-top: 30%;
    background: black;
}
```

## 浏览器是怎样解析CSS选择器的？

CSS选择器的解析是从右向左解析的。若从左向右的匹配，发现不符合规则，需要进行回溯，会损失很多性能。若从右向左匹配，先找到所有的最右节点，对于每一个节点，向上寻找其父节点直到找到根元素或满足条件的匹配规则，则结束这个分支的遍历。

两种匹配规则的性能差别很大，是因为从右向左的匹配在第一步就筛选掉了大量的不符合条件的最右节点（叶子节点），而从左向右的匹配规则的性能都浪费在了失败的查找上面。

而在 CSS 解析完毕后，需要将解析的结果与 DOM Tree 的内容一起进行分析建立一棵 Render Tree，最终用来进行绘图。在建立 Render Tree 时（WebKit 中的「Attachment」过程），浏览器就要为每个 DOM Tree 中的元素根据 CSS 的解析结果（Style Rules）来确定生成怎样的 Render Tree。

## 怎么让Chrome支持小于12px 的文字？

```
p{ font-size:10px;-webkit-transform:scale(0.8); } //0.8是缩放比例
```

## style标签写在body后与body前有什么区别？

页面加载自上而下 当然是先加载样式。

写在body标签后由于浏览器以逐行方式对HTML文档进行解析，当解析到写在尾部的样式表（外联或写在style标签）会导致浏览器停止之前的渲染，等待加载且解析样式表完成之后重新渲染，在windows的IE下可能会出现FOUC现象（即样式失效导致的页面闪烁问题）

## 浏览器重绘(repaint)和重排(reflow)

引发重排的事件

* 改变视图窗口的大小
* 改变文字的大小
* 改变元素边距
* 元素的位置重新定位（脱离文档流）
* 操作DOM
* 添加/删除样式表，操作class属性，设置style属性
* CSS动画属性

![reflow](http://xuheng.inject.top/images/reflow_props.jpg)

引发重绘的事件

* 修改元素外观属性，如修改文字颜色/图片背景色等
* 所有重排事件都会引发重绘

![reflow](http://xuheng.inject.top/images/repaint_props.jpg)

如何减少重排

* 对DOM进行多次添加删除操作时，使用documentFragment对象。
* 操作元素时使用定位，脱落文档流或者display:none隐藏该元素
* 避免逐项更改样式，将样式列表定义为class并一次更改class属性
* 避免循环读取offsetLeft等属性，在循环之前将它们缓存起来
* 避免使用table布局