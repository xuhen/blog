---
title: JavaScript 中常用的工具函数
date: 2020-01-21 11:07:31
categories: JavaScript
tags:
    - js
    - util
---

> 在工作中会时不时遇到一些工具函数，这里记录下，方便下个项目用地，会优化。

## 产生随机长度的字符串

```
function generateString(n) {
    let str = 'abcdefghijklmnopqrstuvwxyz9876543210';
    let tmp = '',
        len = str.length;
    for (i = 0; i < n; i++) {
        tmp += str.charAt(Math.floor(Math.random() * len));
    }
    return tmp;
}

let str = generateString(10);  // 长度为10的随机字符串

```