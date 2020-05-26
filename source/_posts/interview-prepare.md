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

function throttle(fn, wait) {
  let inThrottle = false;
  return function () {
    const args = arguments;
    const ctx = this;
    if (!inThrottle) {
      fn.apply(ctx, args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
      }, wait)
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


function camelCaseToDash(str) {
  return str.replace(/([a-z])([A-Z])/g, '$1-$2').toLowerCase();
}

function dashToCamelCase(str) {
  return str.replace(/-([a-z])/g, function (a, m) {
    console.log(a, m);
    return m.toUpperCase();
  })
}
// var r = camelCaseToDash('helloWorldNiceToMeetYou');
var r = dashToCamelCase('hello-world-nice-to-meet-you');

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

// example1
const sub = function (a, b, c, d) {
    return a + b + c + d;
}

const subCurry = curry(sub);

var r  = subCurry(1)(2)(3)(4);

console.log(r);

// example2
function checkByRegExp(regExp, string) {
    return regExp.test(string);
}
let _check = curry(checkByRegExp);

let checkCellPhone = _check(/^1\d{10}$/);
let checkEmail = _check(/^(\w)+(\.\w+)*@(\w)+((\.\w+)+)$/);

console.log(checkCellPhone('18642838455'));
console.log(checkEmail('test@qq.com'));

// example3

let list = [
    {
        name: 'lucy'
    },
    {
        name: 'jack'
    }
]
const map = function (key, obj) {
    return obj[key];
};
let prop = curry(map)
let names = list.map(prop('name'));

console.log(names);

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
// https://www.cnblogs.com/onepixel/p/10806891.html

// 二分查找
function binarSearch(arr, target) {
  var lo = 0;
  var hi = arr.length - 1;

  while (lo <= hi) {
    var mid = lo + Math.floor((hi - lo) / 2);

    if (arr[mid] > target) {
      hi = mid - 1;
    } else if (arr[mid] < target) {
      lo = mid + 1;
    } else {
      return mid;
    }
  }

  return -1;
}


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
// https://segmentfault.com/a/1190000008705541


Promise.all 实现代码

Promise.all = function (promises) {
    let results = [];
    let promiseCount = 0;
    let promisesLength = promises.length;
    return new Promise(function (resolve, reject) {
        for (let i = 0; i < promisesLength; i++) {
            let val = promises[i];
            Promise.resolve(val).then(function (res) {
                promiseCount++;
                results[i] = res;
                if (promiseCount === promisesLength) {
                    return resolve(results);
                }
            }).catch((err) => {
                return reject(err);
            });
        }
    });
};

let promise1 = new Promise(function (resolve) {
    setTimeout(() => {
        resolve(1);
    }, 1000)
});
let promise2 = new Promise(function (resolve) {
    resolve(2);
});
let promise3 = new Promise(function (resolve) {
    resolve(3);
});

let promiseAll = Promise.all([promise1, promise2, promise3]);
promiseAll.then(function (res) {
    console.log(res);
});

```


jsonp 函数实现

```
var jsonp = function (url, callback) {
  var cbname = 'jsonpRequest_' + Date.now();

  window[cbname] = function (data) {
    callback(data);

    document.body.removeChild(script);
    delete window[cbname];
  }
  url += url.indexOf('?') == -1 ? '?' : '&';
  url += 'callback=' + cbname;
  var script = document.createElement('script');
  script.src = url;
  script.type = 'text/javascript';
  document.body.appendChild(script);
}

jsonp('http://api.douban.com/v2/movie/in_theaters', function (data) {
  console.log(data);

});
```

es5的继承

```
function Child() {
  Parent.call(this)
}

function Parent() {

}
Child.prototype = Object.create(Super.prototype, {
  constructor: {
    value: Child,
    enumerable: false,//不能枚举
    writable: true,
    configurable: true
  }
});
```

```
// 浏览器渲染原理
https://juejin.im/entry/59e1d31f51882578c3411c77

HTML 页面有三个重要生生命周期函数

DOMContentLoaded :
浏览器完全加载了HTML文件，DOM树构建完成，但是外部资源像<img> 和样式表可能还没有完全加载
load:
HTML加载完成，并且所有其他外部资源也加载完毕(比如img和link)
beforeunload/unload
用户离开页面

css加载不会阻塞DOM树解析，但是会阻塞DOM树渲染
参考如下文章:
https://juejin.im/post/5b88ddca6fb9a019c7717096
```


```
bfc知识点:
https://juejin.im/post/5c860afd6fb9a049fd10ab9d
```

// 红绿灯问题
```

function creatLight(type) {
  return function (time) {
    return new Promise((resolve) => {
      setTimeout(() => {
        console.log(type);
        resolve(type);

      }, time)
    })
  }
}

const setRedLight = creatLight('red');
const setYellowLight = creatLight('yellow');
const setGreenLight = creatLight('green');

async function setLight() {
  await setRedLight(2000);
  await setYellowLight(1000);
  await setGreenLight(3000);
  await setLight();
}

setLight();
```

继承问题

```
function Ofo() { }
function Bick() {
  this.name = 'mybick'
}

var myBick = new Ofo()      // 此时实例的__proto__指向原来的Ofo的prototype 而不是Bick的实例，也就没有name属性
Ofo.prototype = new Bick()
var youbick = new Bick()
console.log(myBick.name)  // undefined
console.log(youbick.name) // mybick
```

## 队列问题

```
async function async1() {
  console.log('async1 start')
  var r = await async2();
  // 上面的await等价下面的这句表达
  // Promise.resolve(async2promise).then(next)
  console.log(r);
  console.log('async1 end')
}
async function async2() {
  console.log('async2')
}
console.log('script start')
setTimeout(function () {
  console.log('setTimeout')
}, 0)
async1();
new Promise(function (resolve) {
  console.log('promise1')
  resolve();
}).then(function () {
  console.log('promise2')
})
console.log('script end')

```


## 什么是内存泄漏

```
内存泄漏就是由于疏忽或错误造成程序未能释放那些已经不再使用的内存，造成内存浪费。
```

## 什么是闭包

```
一个内部函数，有权访问包含其的外部函数中的变量。
```

## js的内存管理

```
https://juejin.im/post/5d116a9df265da1bb47d717b
```


## generator的自动运行

```
function* gen(p) {
  console.log('p=', p);
  const a = yield 1;
  console.log('a=', a);
  const b = yield 2;
  console.log('b=', b);
  const c = yield 3;
  console.log('c=', c);
  return 4;
}
function spawn(genF) {
  return new Promise((resolve, reject) => {
    const gen = genF();

    function step(nextF) {
      let next;

      try {
        next = nextF();
      } catch (e) {
        reject(e);
      }

      if (next.done) {
        return resolve(next.value);
      }

      Promise.resolve(next.value).then(function (v) {
        step(function () {
          return gen.next(v);
        })
      }, function (e) {
        step(function () {
          return gen.throw(e);
        })
      })

    }

    step(function () {
      return gen.next();
    })

  })
}

spawn(gen).then((res) => {
  console.log('res', res);
});
```