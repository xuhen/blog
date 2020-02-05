---
title: js中数组去重的方法
date: 2020-02-05 11:27:16
tags:
    - js
    - util
---

> 数组去重在js里有很多实现方式，有点方式时间复杂点为 O(n) 有的是 O(n^2)，有的空间复杂高是O(n^2)，有的底为O(1)，下面我们来看看实现方式

## 双层 for 循环去重

```
function distinct(arr) {
    for (let i=0, len=arr.length; i<len; i++) {
        for (let j=i+1; j<len; j++) {
            if (arr[i] == arr[j]) {
                arr.splice(j, 1);
                // splice 会改变数组长度，所以要将数组长度 len 和下标 j 减一
                len--;
                j--;
            }
        }
    }
    return arr;
}
```

时间复杂度为O(n^2)， 空间复杂度为O(1)

## Array.filter() 加 indexOf 去重

```
function distinct(arr) {
    return arr.filter((item, index)=> {
        return arr.indexOf(item) === index
    })
}
```

时间复杂度为O(n^2)， 空间复杂度为O(1)
note: 因为indexOf函数里的实现也有O(n)的时间复杂度

## Array.sort() 加一行遍历冒泡(相邻元素去重)

```
function distinct(array) {
    var res = [];
    var sortedArray = array.sort();
    var seen;
    for (var i = 0, len = sortedArray.length; i < len; i++) {
        // 如果是第一个元素或者相邻的元素不相同
        if (!i || seen !== sortedArray[i]) {
            res.push(sortedArray[i])
        }
        seen = sortedArray[i];
    }
    return res;
}
```

时间复杂度为O(n)， 空间复杂度为O(n)

## ES6 中的 Set 去重

```
function distinct(array) {
   return Array.from(new Set(array));
}
```

时间复杂度和空间复杂度后面研究再补充

## Object 键值对去重

```
function distinct(array) {
    var obj = {};
    return array.filter(function(item, index, array){
        return obj.hasOwnProperty(typeof item + item) ? false : (obj[typeof item + item] = true)
    })
}
```

时间复杂度为O(n)， 空间复杂度为O(n)