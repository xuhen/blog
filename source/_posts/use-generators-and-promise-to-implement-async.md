---
title: 用 generators 和 promise 模拟 async 的功能
date: 2019-11-10 16:06:55
tags:
    - async
---

想象一下，如果我们有一段async 写的函数，我们能用 promises 和 generator 去重写吗？


比如下面这段代码

``` js
    async function init() {
        const res1 = await doTask1();
        console.log(res1);

        const res2 = await doTask2(res1);
        console.log(res2);

        const res3 = await doTask3(res2);
        console.log(res3);

        return res3;
    }
```

上面的代码执行了三个异步任务，每个任务都依赖于前一个任务的完成；最终返回了最后一个任务的结果。


我们能用generator去实现上面的功能吗？

## 什么是 generator
generator 简单来说就是能退出执行环境，随后又能重新进入执行环境的函数。
比如下面的例子：

```js
    function* gen() {
        const a = yield 1;
        const b = yield true;
        const c = yield "foo";

        console.log(a, b, c);
    }
```

generator 函数有些有意思的特点

* 当一个generator函数被执行的时候，函数体不是马上执行的；而是返回了一个可遍历的对象。
* 唯一能执行 generator 函数体的方法是执行在可遍历对象上的 next 函数；每次 next 方法执行到下个 yield 表达式时停止。然后返回此处的结果作为value 值。
* next 方法也可以接收参数。执行next并传入参数会用传入的值替换掉当前停顿的yield表达式，然后继续执行到下个 yield 表达式。


## generator 的特性对异步编程有什么好处

这里的重点是generator 函数可以 yield promises：

generator 函数可以 yield 一个 promise （一个异步任务），它的遍历器（iterator）可以被控制停顿执行，直到 这个promise 被 resolve (or reject)；然后拿着 被resolved (or rejected) 值继续执行。

所以这种模式，可以帮助我们用 遍历器（iterator） 和 promises 去实现下面的场景：

```js
    function* init() {
        const res1 = yield doTask1();
        console.log(res1);

        const res2 = yield doTask2(res1);
        console.log(res2);

        const res3 = yield doTask3(res2);
        console.log(res3);

        return res3;
    }
```

看看上面的结构是不是很像开篇的 async 函数的版本呀。

但是我们需要去执行的是函数体啊；那我们需要一个wrapper函数，当一个 promise 被 yielded， 它能控制遍历器去停顿，一旦 promise 被 resolves (or rejects) 可以继续执行。

下面是这个wrapper函数：

```js
    function runner(genFn) {
        const itr = genFn();

        function run(arg) {
            const result = itr.next(arg);
            if (result.done) {
                return result.value;
            } else {
                return Promise.resove(result.value).then(run);
            }
        }

        run();
    }
```


现在就可以把这两个函数结合起来啦：

```js
    function* init() {
        const res1 = yield doTask1();
        console.log(res1);

        const res2 = yield doTask2(res1);
        console.log(res2);

        const res3 = yield doTask3(res2);
        console.log(res3);

        return res3;
    }

    runner(init);
```

嗯嗯，就这么多了，我们用 runner 函数和 generator 函数实现了 async 函数的功能了，开心 ：）