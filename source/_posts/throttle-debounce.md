---
title: throttle和debounce函数实现
date: 2020-01-02 12:10:42
tags:
---

## throttle

`throttle` 函数的使用场景为每隔 delay（一般为XX ms） 时间执行下特定函数，类似于给函数执行的频次节流。

> throttle(1000, false, testFn) / throttle(1000, testFn)
> 场景1：
>    * onresize 开始时执行 `testFn`函数
>    * 每隔1s执行 `testFn` 函数
>    * 停止 onresize 时也执行下回调。



> throttle(1000, true, testFn)
> 场景2：
>    * onresize 开始时执行 `testFn`函数
>    * 每隔1s执行 `testFn` 函数
>    * 停止 onresize 时不执行下回调。


```
function testFn(e) {
    console.log('moved', e, this);
}

window.onresize = throttle(1000, false, testFn);
```

## debounce

`debounce` 的使用场景是动作执行期间不执行回调函数，只在开头或结束时执行回调

> debounce(1000, testFn);
> 场景1：
>    * onresize 开始时不执行 `testFn`函数
>    * 每隔1s执不执行 `testFn` 函数
>    * 停止 onresize 时执行下回调 `testFn`。

> debounce(1000, true, testFn);
> 场景2：
>    * onresize 开始时执行 `testFn`函数
>    * 每隔1s执不执行 `testFn` 函数
>    * 停止 onresize 时不执行下回调 `testFn`。

这个场景也可以改成开始和结束都执行下回调 `testFn`

```
window.onresize = debounce(1000, true, testFn);

function debounce(delay, atBegin, callback) {
    return callback === undefined ? throttle(delay, atBegin, false) : throttle(delay, callback, atBegin !== false);
}
```

## throttle 源码
方便学习，最后放上 throttle 的源码

```
function throttle(delay, noTrailing, callback, debounceMode) {
    var timeoutID;
    var cancelled = false;
    var lastExec = 0;

    function clearExistingTimeout() {
        if (timeoutID) {
            clearTimeout(timeoutID);
        }
    }

    function cancel() {
        clearExistingTimeout();
        cancelled = true;
    }


    if (typeof noTrailing !== 'boolean') {
        debounceMode = callback;
        callback = noTrailing;
        noTrailing = undefined;
    }

    function wrapper() {
        var self = this;
        var elapsed = Date.now() - lastExec;
        var args = arguments;

        if (cancelled) {
            return;
        }

        function exec() {
            lastExec = Date.now();
            callback.apply(self, args);
        }

        function clear() {
            timeoutID = undefined;
        }

        if (debounceMode && !timeoutID) {
            exec();
        }

        clearExistingTimeout();

        if (debounceMode === undefined && elapsed > delay) {
            exec();
        } else if (noTrailing !== true) {
            timeoutID = setTimeout(debounceMode ? clear : exec, debounceMode === undefined ? delay - elapsed : delay);
        }

    }

    wrapper.cancel = cancel;
    return wrapper;

}
```
