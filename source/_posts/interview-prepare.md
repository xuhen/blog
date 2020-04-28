---
title: 面试准备
date: 2020-04-28 10:02:47
tags:
    - interview
---

> 下面是一些在面试中常考的知识点总结

```
function debounce(f, wait, immediate) {
  let timer;

  return function () {
    let args = [...arguments];
    const ctx = this;

    const helper = function () {
      timer = null;

      if (!immediate) {
        f.apply(ctx, args)
      }
    }

    let callNow = immediate && !timer;

    clearTimeout(timer);

    timer = setTimeout(helper, wait);

    if (callNow) f.apply(ctx, args)
  }
}

const throttle = (func, wait) => {
  let timer;
  return function () {
    const args = arguments;
    const ctx = this;

    const helper = function () {
      timer = null;
    }

    if (!timer) {
      func.apply(ctx, args)
      timer = setTimeout(helper, wait)
    }
  }
}

function resize() {
  console.log('resize');
}

resize = throttle(resize, 200, true);

window.addEventListener('resize', resize);


// 生成 [m, n]范围内的随机整数，包括m和n
function random(m, n) {
  let factor = n - m + 1;
  return Math.floor(Math.random() * factor + m);
}
var r = random(10, 20);
console.log(r);


function upperCasetoLine(str) {
  return str.replace(/[A-Z]/g, (mc) => {
    return '-' + mc.toLowerCase();
  });
}

function lineToUpperCase(str) {
  return str.replace(/-(\w)/g, (mc, p1) => {
    console.log(mc);

    return p1.toUpperCase();
  });
}
// var r = upperCasetoLine('helloWorldNiceToMeetYou');
var r = lineToUpperCase('hello-world-nice-to-meet-you');

console.log(r);

// 接口数据缓存

const getPromise = (function () {
  const promiseCache = new Map();

  return function (url) {
    let promise = promiseCache.get(url);
    if (!promise) {
      promise = fetch(url).then(res => {
        return res.json();
      }).catch(error => {
        promiseCache.delete(url)
        return Promise.reject(error)
      })
      promiseCache.set(url, promise)
    }

    return promise
  }
})()


getPromise('http://hn.algolia.com/api/v1/search?query=redux').then((res) => {
  console.log('res', res);
}).catch((err) => {
  console.log('err', err);
})
getPromise('http://hn.algolia.com/api/v1/search?query=redux').then((res) => {
  console.log('res', res);
})

setTimeout(() => {
  getPromise('http://hn.algolia.com/api/v1/search?query=redux').then((res) => {
    console.log('res', res);
  })
}, 5000)


// 并发发起多个请求，并且缓存数据
const queryAll = (function () {
  const promiseCache = new Map();

  return function (queryApiName) {
    const queryIsArray = Array.isArray(queryApiName)
    const apis = queryIsArray ? queryApiName : [queryApiName]

    const promiseApi = []

    apis.forEach(api => {
      let promise = promiseCache.get(api)

      if (promise) {
        promiseApi.push(promise)
      } else {
        promise = fetch(api).then(res => {
          return res.json();
        }).catch(error => {
          promiseCache.delete(api)
          return Promise.reject(error)
        })
        promiseCache.set(api, promise)
        promiseApi.push(promise)
      }
    })
    return Promise.all(promiseApi).then(res => {
      return queryIsArray ? res : res[0]
    })
  }
})();



queryAll(['http://hn.algolia.com/api/v1/search?query=redux', 'http://hn.algolia.com/api/v1/search?query=react'])
  .then((res) => {
    console.log(res);
  }).catch((err) => {
    console.log('err', err);
  })



// 注意要点
// 1. 成员访问运算优先级比赋值运算高
// 2. 赋值运算会返回右边的值或者引用指针
// 3. 赋值运算从右往左算
var a = { n: 1 };
var b = a;
a.x = a = { n: 2 };

console.log(a); // { n: 2 }
console.log(b); // { n: 1, x: { n: 2 } }



// 扁平化嵌套数组

function flatten(arr) {
  var result = [];

  function helper(arr) {
    for (var i = 0; i < arr.length; i++) {
      if (Array.isArray(arr[i])) {
        helper(arr[i]);
      } else {
        result.push(arr[i]);
      }
    }
  }

  helper(arr);
  return result;
}

function flattenMd(arr) {
  var result = [];
  for (var i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      result = result.concat(flattenMd(arr[i]));
    } else {
      result.push(arr[i]);
    }
  }
  return result;
}

function flatten(arr) {
  return arr.reduce(function (plane, toBeFlatten) {
    console.log(Array.isArray(toBeFlatten), toBeFlatten);
    return plane.concat(Array.isArray(toBeFlatten) ? flatten(toBeFlatten) : toBeFlatten);
  }, []);
}

function flatten(arr) {
  return arr.toString().split(',').map(function (item) {
    return +item
  })
}

var arr = [1, [2, 3, [4, 5, [9, [10]]], 6], '7', -8];
var r = flatten(arr);
console.log(r);

// 考察运算符优先级，继承，this
function Foo() {
    getName = function () { alert (1); };
    return this;
}
Foo.getName = function () { alert (2);};
Foo.prototype.getName = function () { alert (3);};
var getName = function () { alert (4);};
function getName() { alert (5);}
 
//请写出以下输出结果：
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();



// 并发请求接口，但是顺序输出结果
function request(urls) {
  let promises = urls.map((url) => {
    return fetch(url).then((res) => {
      return res.json();
    }).catch((err) => {
      return Promise.reject(err);
    })
  });

  promises.reduce((chain, promise) => {
    return chain.then(() => {
      return promise;
    }).then((res) => {
      console.log('res', res);
    }).catch((err) => {
      console.error(err);
    })
  }, Promise.resolve())
}

request([
  'http://hn.algolia.com/api/v1/search?query=redux',
  'http://hn.algolia.com/api/v1/search?query=react',
  'http://hn.algolia.com/api/v1/search?query=immutable',
])


// compose
const compose = function(...fns) {
    return function (x) {
        return fns.reduceRight((y, f) => f(y), x);
    }
}

const g = n => n + 1;
const f = n => n * 2;
const b = n => n - 3;

const h = compose(f, g, b);

const r = h(20); 

console.log(r)

// curry
function curry(func) {
    return function curried(...args) {
        if (args.length >= func.length) {
            return func.apply(this, args);
        } else {
            return function (...args2) {
                return curried.apply(this, args.concat(args2));
            }
        }
    };
}

const sub = function (a, b, c, d) {
    return a + b + c + d;
}

const subCurry = curry(sub);

var r  = subCurry(1)(2)(3)(4);

console.log(r);


// aop
Function.prototype.aopBefore = function (fn) {
    const _this = this
    return function () {
        fn.apply(this, arguments)
        return _this.apply(this, arguments)
    }
}
Function.prototype.aopAfter = function (fn) {
    const _this = this
    return function () {
        let current = _this.apply(this, arguments);

        fn.apply(this, arguments) 
        return current
    }
}

let aopFunc = function (a, b, c) {
    console.log('aop')
    return a + b + c;
}

aopFunc = aopFunc.aopBefore((...rest) => {
    console.log('aop before', rest)
}).aopAfter((...rest) => {
    console.log('aop after', rest)
})
// 真正调用
var r = aopFunc(1, 2, 3);

console.log(r)

// mycall
var foo = {
    value: 1
};

function bar(a, b) {
    console.log(this.value);
    return {
        value: this.value,
        name: a,
        age: b
    }
}


Function.prototype.myCall = function (context) {
    var context = context || window;
    context.fn = this;
    var args = [];
    for (var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }

    console.log(args, 'context.fn(' + args + ')');
    var result = eval('context.fn(' + args + ')');
    delete context.fn;
    return result;
}

var r = bar.myCall(foo, 'edward', 23);
console.log(r);

// 发布订阅模式和观察者模式区别
https://www.cnblogs.com/onepixel/p/10806891.html

// 二分查找
function binarSearch(array, obj) {

    var low = 0, high = array.length;

    while (low < high) {
        var mid = Math.floor((low + high) / 2);
        if (array[mid] < obj) low = mid + 1;
        else high = mid;
    }

    return high;
};

console.log(binarSearch([10, 20, 30, 40, 50], 35))

// bind
Function.prototype.bind2 = function (context) {
    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);
    var fNOP = function () {};
    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}

圣杯布局
https://segmentfault.com/a/1190000008705541





```