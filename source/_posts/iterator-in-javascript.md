---
title: javascript 中的迭代器
date: 2020-05-13 21:02:56
tags:
    - js
    - util
---

> 迭代器模式是指提供一种顺序访问聚合对象中的各个对象的方法，而且不用暴露该对象的内部表示。


比如jquery中的$.each迭代器

```
$.each( [1, 2, 3], function( i, n ){ 
    console.log( '当前下标为: '+ i ); 
    console.log( '当前值为:' + n );
});
```

## 内部迭代器

我们下面就来实现jquery中的那个迭代器

```
function each(arr, callback) {
  var value;
  for (var i = 0; i < arr.length; i++) {
    value = callback.call(null, i, arr[i]);

    if (value === false) {
      break;
    }
  }
}

each([1, 2, 3], function (i, n) {
  console.log([i, n]);

  if (i === 1) {
    return false;
  }
});
```

我们刚刚定义的each函数属于内部迭代器，它的内部已经定义好了迭代的规则，它完全接收整个迭代的过程，外部只需要一次初始调用。

如果我们要比较两个数组是否相等，可以像下面这样。

```
var compare = function( ary1, ary2 ){ 
    if ( ary1.length !== ary2.length ){
        throw new Error ( 'ary1 和 ary2 不相等' ); 
    }
    each( ary1, function( i, n ){ 
        if ( n !== ary2[ i ] ){
            throw new Error ( 'ary1 和 ary2 不相等' ); 
        }
    });
    alert ( 'ary1 和 ary2 相等' ); 
};
compare( [ 1, 2, 3 ], [ 1, 2, 4 ] );
```

但是这个compare函数一点也不好看。

## 外部迭代器

外部迭代器必须显示的请求迭代下一个元素。

外部迭代器增加了一些调用的复杂度，但相对的也增加了迭代器的灵活性，我们可以手工控制迭代器的过程和顺序。

```
var Iterator = function (obj) {
  var current = 0;
  var next = function () {
    current += 1;
  };
  var isDone = function () {
    return current >= obj.length;
  }
  var getCurrItem = function () {
    return obj[current];
  };
  return {
    next: next,
    isDone: isDone,
    getCurrItem: getCurrItem
  }
};

var compare = function (iterator1, iterator2) {
  while (!iterator1.isDone() && !iterator2.isDone()) {
    if (iterator1.getCurrItem() !== iterator2.getCurrItem()) {
      throw new Error('iterator1 和 iterator2 不相等');
    }
    iterator1.next();
    iterator2.next();
  }
  console.log('iterator1 和 iterator2 相等');
}
var iterator1 = Iterator([1, 2, 3]);
var iterator2 = Iterator([1, 2, 3]);

compare(iterator1, iterator2);
```
