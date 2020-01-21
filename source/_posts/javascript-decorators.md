---
title: javascript decorators 它们是什么，使用场景
date: 2018-10-05 21:26:49
tags:
  - decorator
---
随着 ES2015+ 的完善，还有编译器的广泛使用，许多人都会在项目代码或教程片段中遇到一些ES语言的新特性。当JS程序员第一次遇到这些特性中的一个（decorators）都会发蒙，这是什么语法呢？


说起装饰器能在ES中流行得多亏了Angular2+，那在Angular中能用装饰器，又得多亏了TypeScript, 因为在ES目前的版本中，这个特性正处在[stage 2 proposal](https://github.com/tc39/proposals#stage-2) 的阶段，那这也意味着这个特性是以后才会加到ES的稳定版本的。那让我们言归正传看下装饰器是啥东东，它怎样使我们的代码更整洁，和易懂的。


### 装饰器是啥呢？
简单的理解，装饰器就是包裹一段代码在一个函数中--也就是字面意思‘修饰’它。这个思想可能之前你接触‘函数组合’，或者高阶函数时比较熟悉。

用一个函数去包裹另一个函数，在目前的ES中已有广泛的应用了。

``` js
function doSomething(name) {
  console.log('Hello, ' + name);
}

function loggingDecorator(wrapped) {
  return function() {
    console.log('Starting');
    const result = wrapped.apply(this, arguments);
    console.log('Finished');
    return result;
  }
}

const wrapped = loggingDecorator(doSomething);
```

上面的例子只是产生了一个新函数--在变量`wrapped`中。这个新函数可以像调用`doSomething`一样，它俩都干一样的事。它们的区别是新函数的调用会输出一些logging日志。

``` js
doSomething('Graham');
// Hello, Graham

wrapped('Graham');
// Starting
// Hello, Graham
// Finished
```

### 怎样使用javascript装饰器呢？
在JS中装饰器用了一种特殊的语法，那就是用@符合作为前缀，并且放在所有装饰的代码前面。


在同一个代码片段上，你爱用多少个装饰器都可以，并且都会安你声明的顺序起作用。

比如下面的例子：

``` js
@log()
@immutable()
class Example {
  @time('demo')
  doSomething() {
    //
  }
}
```

这个例子定义了一个类，并且声明了三个装饰器--两个是类的，还有一个是类的属性的。

*   `@log` 可能输出类的所用日志
*   `@immutable` 可能让类保持不可串改-- 或许对实例使用了 `Object.freeze`
*   `@time` 将记录一个方法执行的花费的时间，并用一个唯一的标签以日志的方式输出。


目前来看，要用装饰器的话得用编译器（babel），因为目前的浏览器和Node最新版本还没支持这个新特性。如果你用babel的话用[ transform-decorators-legacy plugin](https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy)插件就能支持。

### 为啥使用装饰器呢？

目前函数组合已经在Javascript广泛使用，把同样的技巧运用给其他代码（比如：类和类的属性）是非常困难或者说不可能的。

装饰器也运用更优雅的方式去包装你的代码，结果就是让你更集中于你的业务代码，不被冗长的包装代码分心。

### 不同类型的装饰器
目前，只有类和类的成员才能用装饰器。这些成员有 properties, methods, getters, and setters

装饰器其实就是返回包装函数的函数而已，返回的函数被精心装饰了下。这些装饰器函数在程序第一次执行时被算值一次。被装饰的代码被返回值替换掉。

### 类的成员装饰器
属性装饰器只作用于类的单个成员--也就是properties, methods, getters, 和 setters。这个装饰器函数用三个参数去执行。

*   `target` 成员所在的目标类
*   `name` 此成员在类上的名字
*   `descriptor` 成员描述，这是传给 [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 方法的对象


经典的例子就是`@readonly`装饰器。下面是它的简单实现：

``` js
function readonly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}
```

上面的例子其实就是把描述对象的`writable`属性设置为`false`

下面是使用readonly的例子

``` js
class Example {
  a() {}
  @readonly
  b() {}
}

const e = new Example();
e.a = 1;
e.b = 2;
// TypeError: Cannot assign to read only property 'b' of object '#<Example>'
```

我们能丰富下上面的例子。我们能让这个装饰器有不一样的行为。比如输出所有的输入输出日志。

``` js
function log(target, name, descriptor) {
  const original = descriptor.value;
  if (typeof original === 'function') {
    descriptor.value = function(...args) {
      console.log(`Arguments: ${args}`);
      try {
        const result = original.apply(this, args);
        console.log(`Result: ${result}`);
        return result;
      } catch (e) {
        console.log(`Error: ${e}`);
        throw e;
      }
    }
  }
  return descriptor;
}
```


上面的例子用新的方法替换了旧的并输出了arguments参数，调用旧方法，和输出日志。


注意，上面的例子我们用了[spread operator](https://www.sitepoint.com/es6-destructuring-assignment/)操作符，
去参数里取到数组，这也是比`arguments`的方式更时髦的选择。

我们也看下如何使用：

``` js
class Example {
  @log
  sum(a, b) {
    return a + b;
  }
}

const e = new Example();
e.sum(1, 2);
// Arguments: 1,2
// Result: 3
```

你或许注意到了，我们必须用晦涩的语法去执行这段被装饰的方法。简单说来`apply`函数让我们在`this`作用域上调用旧函数，并且传入参数。

更进一步，我们能让装饰器接收一些参数，比如下面这样重写下`log`装饰器：

``` js
function log(name) {
  return function decorator(t, n, descriptor) {
    const original = descriptor.value;
    if (typeof original === 'function') {
      descriptor.value = function(...args) {
        console.log(`Arguments for ${name}: ${args}`);
        try {
          const result = original.apply(this, args);
          console.log(`Result from ${name}: ${result}`);
          return result;
        } catch (e) {
          console.log(`Error from ${name}: ${e}`);
          throw e;
        }
      }
    }
    return descriptor;
  };
}
```

这个例子变得有点复杂了，让我们仔细看会得到下面的结论：

*   `log`  函数接收唯一的一个参数：`name`
*   这个函数然后返回一个本身是装饰器的函数


这个log装饰器，除了使用最外层的参数`name`之外几乎等同于之前的那个 `log`装饰器。

下面是复杂`log`的使用：

``` js
class Example {
  @log('some tag')
  sum(a, b) {
    return a + b;
  }
}

const e = new Example();
e.sum(1, 2);
// Arguments for some tag: 1,2
// Result from some tag: 3
```

很明显，我们能用自己提供的 tag去区分不同的日志输出。

上面的代码能工作，是因为 `log('some tag')` 函数是被JavaScript runtime立即执行的
返回值作为`sum`方法的装饰器。


### 类装饰器

类装饰器是在整个类定义时一并起作用的。这个装饰器函数的参数只有一个，即被装饰的 `constructor` 函数。

注意，这是作用于 `constructor` 函数的，而不是基于类创建的实例。这就意味着如果你想操纵实例，你必须返回一个被包裹的 `constructor` 版本。

总的来说，这种用法比起类成员装饰器来比较少用到，因为能用这种方式实现的事，也能用一个简单的函数调用实现。
如果你实在需要用到，你需要返回一个新 `constructor` 函数去换掉类 `constructor`.

回到我们的日志函数例子，让我们打印下，`constructor` 的参数。

``` js
function log(Class) {
  return (...args) => {
    console.log(args);
    return new Class(...args);
  };
}
```

这里我们接受一个类作为参数，并且返回了新的函数作为 `constructor`。
这只是简单的打印了下参数，返回了旧 `constructor` 的实例。


如下调用：

```js
@log
class Example {
  constructor(name, age) {
  }
}

const e = new Example('Graham', 34);
// [ 'Graham', 34 ]
console.log(e);
// Example {}
```

我们能看到构造的 `Example` 类打印了提供的参数日志，构造的 `value` 也
确实是 `Example` 的实例。正如我们要的一样。

向类装饰器传参，就像类成员传参一样：

``` js

function log(name) {
  return function decorator(Class) {
    return (...args) => {
      console.log(`Arguments for ${name}: args`);
      return new Class(...args);
    };
  }
}

@log('Demo')
class Example {
  constructor(name, age) {}
}

const e = new Example('Graham', 34);
// Arguments for Demo: args
console.log(e);
// Example {}
```

### 结束语

类成员装饰器提供了包装一段在类中的代码的好方法。就像我们包装函数到另一个函数一样。这样我们就能写些
帮助代码复用在许多地方，这样的代码既优雅，也易懂。

翻译 [JavaScript Decorators: What They Are and When to Use Them](https://www.sitepoint.com/javascript-decorators-what-they-are/)



