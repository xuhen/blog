---
title: javascript 混入模式 (mixin)
date: 2020-11-21 11:05:10
tags:
  - js
  - design parttern
---

> 在这篇文章中我们将细致的探讨 _mixins_；本文不打算用常规的方式介绍，而是用比较直观自然的方式去介绍 _mixin_ 的一些策略。

## 函数复用

在 JavaScript 里，一个对象 (A) 隐藏的 `__proto__` 属性会以指针的方式指向一个 `prototype` 对象 (B)，利用这点 A 可以继承 B 对象上的方法和属性。_prototypes_ 对于代码复用来说是非常棒的：一个 _prototype_ 实例定义的属性可以被无限多个对象继承下来。_prototypes_ 也可以继承来自其他 _prototypes_ 的属性，这样就形成了 prototype 链。我们也可以利用这点去模拟其他语言(Java, C++)里的层级继承。对于描述物品的类型关系，继承是一种方式；如果我们的主要动机是函数的复用，那继承就变成迷宫一样无任何意义的子类了。里面会充斥着冗余代码和难以理解的逻辑（这个按钮是矩形框还是绑定了事件的控制组件？那就让我们 Button 继承自 Rectangle、Rectangle 再继承自 Control...）

幸运的是，对于函数的复用，在 JavaScript 里有更好的选择。对比更严格类型的语言，JavaScript 里的对象能够调用任何函数。方式就是代理 - 任何方法都能通过 `call` 和 `apply`去调用。这是一个强大的特性，在第三方的库里面能看到有大量的使用。但是，代理的便利性有时和代码库里结构化规则是相矛盾的；换句话说这种语法太冗长了。`Mixins` 就是一个很好的折中方案，允许整个函数代码被 "借用"，语法简洁而且最重要的是和 `prototypes` 很好的融合。它提供了有描述性的层级继承、避免了逻辑的混乱。

## 基础

在通用计算机科学上，_mixin_ 是一个 _class_，这个类定义了一套和类型（e.g. Person, Circle, Observer）有关的方法。`Mixins` 类通常也被认为是抽象类，即它们本身不会被实例化；相反它们上面的方法会被具体的类复制（也叫 “借用”）。这种方式可以只 “继承” 行为而不会和这个行为的提供者类有直接的关系。

好吧，在 JavaScript 里，我们没有类的概念。这实际上是个好事，这就意味着我们能用对象实例了；对象实例提供了清晰性和灵活性 - 我们的 `mixin` 可以是普通对象、 `prototype` 对象或函数，混入的过程就变得含义清楚了。

## 使用案例

我将讨论一些 `mixin` 的技巧，所有这些示例都是同一个案例：创建圆形、半圆和矩形按钮。下图是一个纲要性的图示；矩形里代表 `mixin` 对象，圆形里代表实际的按钮。

![mixin](https://javascriptweblog.files.wordpress.com/2011/05/mixin3.jpg?w=2216)

### 1. 经典混入

当你在百度或者谷歌里输入 _javascript mixin_ 关键字去搜索相关文章时，笔者注意到大部分定义的 mixin 是构造函数和在构造函数 `prototoype` 上定义一些方法。这可以看成是其他语言在 JavaScript 的自然延续 - 早期的 mixin 都是 class，这是在 JavaScript 里最接近其他语言“类”的定义了，下面是一个圆混入类的示例：

```js
var Circle = function () {};
Circle.prototype = {
  area: function () {
    return Math.PI * this.radius * this.radius;
  },
  grow: function () {
    this.radius++;
  },
  shrink: function () {
    this.radius--;
  }
};
```

在实际中，没有必要像上面的方式去定义 mixin。简单的对象字面量就足够了：

```js
var circleFns = {
  area: function () {
    return Math.PI * this.radius * this.radius;
  },
  grow: function () {
    this.radius++;
  },
  shrink: function () {
    this.radius--;
  }
};
```

**extend 函数**

怎样把 mixin 对象里的方法混入别的对象？答案是通过 extend 函数（也叫 augment）。通常 `extend` 只是简单的复制 mixin 里的方法到目标对象。也存在一些小的变形实现。比如 `Prototype.js` 就忽略 `hasOwnProperty` 的检查 （这个库假设了 mixin 在原型链上没有可遍历的属性）然而其他版本的实现假设我们只想复制 mixin 本身的属性。下面的实现兼具安全性和可扩展性：

```js
function extend(destination, source) {
  for (var k in source) {
    if (source.hasOwnProperty(k)) {
      destination[k] = source[k];
    }
  }
  return destination;
}
```

下面我们可以用 extend 函数去扩展 prototype 对象上的方法...

```js
var RoundButton = function (radius, label) {
  this.radius = radius;
  this.label = label;
};

extend(RoundButton.prototype, circleFns);
extend(RoundButton.prototype, buttonFns);
```

### 2. 函数式混入

如果 mixin 对象上定义的方法只是让其他对象来复制，为什么我们要大费周章的把 mixin 创建成普通的对象呢？换句话说，混入应该是一个过程而不是目的。比较好的方案是把 mixin 定义成函数，然后通过代理的方式注入到目标对象，这样就能去掉中间这层（extend 函数）:

```js
var asCircle = function () {
  this.area = function () {
    return Math.PI * this.radius * this.radius;
  };
  this.grow = function () {
    this.radius++;
  };
  this.shrink = function () {
    this.radius--;
  };
  return this;
};

var Circle = function (radius) {
  this.radius = radius;
};
asCircle.call(Circle.prototype);
var circle1 = new Circle(5);
circle1.area(); //78.54
```

这种方式感觉更好了，mixin 是动词而不是名称。这样写的另一个好处是更自然简洁了：this 总是指向目标对象而不是 mixin 对象本身。和传统混入的方式比起来，不会再粗心地复制继承的属性了；并且现在是拷贝 (clone) 而不是复制 (copy) 了。

下面是混入按钮的函数：

```js
var asButton = function () {
  this.hover = function (bool) {
    bool ? mylib.appendClass('hover') : mylib.removeClass('hover');
  };
  this.press = function (bool) {
    bool ? mylib.appendClass('pressed') : mylib.removeClass('pressed');
  };
  this.fire = function () {
    return this.action();
  };
  return this;
};
```

将两个 mixins 放在一起我们得到了圆形的按钮：

```js
var RoundButton = function (radius, label, action) {
  this.radius = radius;
  this.label = label;
  this.action = action;
};

asButton.call(RoundButton.prototype);
asCircle.call(RoundButton.prototype);

var button1 = new RoundButton(4, 'yes!', function () {
  return 'you said yes!';
});
button1.fire(); //'you said yes!'
```

### 3. 添加配置项

这种定义 mixin 为函数的方式，也允许我们通过传递配置项去参数化“被借用”的行为。下面让我我们通过示例了解这种用法：

```js
var asOval = function (options) {
  this.area = function () {
    return Math.PI * this.longRadius * this.shortRadius;
  };
  this.ratio = function () {
    return this.longRadius / this.shortRadius;
  };
  this.grow = function () {
    this.shortRadius += options.growBy / this.ratio();
    this.longRadius += options.growBy;
  };
  this.shrink = function () {
    this.shortRadius -= options.shrinkBy / this.ratio();
    this.longRadius -= options.shrinkBy;
  };
  return this;
};

var OvalButton = function (longRadius, shortRadius, label, action) {
  this.longRadius = longRadius;
  this.shortRadius = shortRadius;
  this.label = label;
  this.action = action;
};

asButton.call(OvalButton.prototype);
asOval.call(OvalButton.prototype, { growBy: 2, shrinkBy: 2 });

var button2 = new OvalButton(3, 2, 'send', function () {
  return 'message sent';
});
button2.area(); //18.84955592153876
button2.grow();
button2.area(); //52.35987755982988
button2.fire(); //'message sent'
```

### 4. 增加缓存

或许上面的实现混入的方式对性能的影响让我们感到担忧，因为我们每次调用 call 的时候都会在每个原型对象上重复定义一遍这些方法。考虑到这些 mixins 只有类型定义的时候才会被调用一次（对比金典模式中每次实例化被调用），这种低频次的创建我们不用太担心。

但是，如果你依然感到担忧，这里还有一个方法。通过在 mixin 上形成闭包我们能缓存初始化的结果。现在，在每个浏览器里，函数化的混入在性能上都大幅度优于经典模式了。

下面是 `asRectangle` 增加缓存的版本：

```js
var asRectangle = (function () {
  function area() {
    return this.length * this.width;
  }
  function grow() {
    this.length++, this.width++;
  }
  function shrink() {
    this.length--, this.width--;
  }
  return function () {
    this.area = area;
    this.grow = grow;
    this.shrink = shrink;
    return this;
  };
})();

var RectangularButton = function (length, width, label, action) {
  this.length = length;
  this.width = width;
  this.label = label;
  this.action = action;
};

asButton.call(RectangularButton.prototype);
asRectangle.call(RectangularButton.prototype);

var button3 = new RectangularButton(4, 2, 'delete', function () {
  return 'deleted';
});
button3.area(); //8
button3.grow();
button3.area(); //15
button3.fire(); //'deleted'
```

### 5. 增加柯粒化(curry)

世事充满了权衡，前面提到的缓存增强也不例外。我们失去了为每次混入创建克隆的能力了；更糟糕的是我们混入的时候也不能通过配置参数去自定义借用函数了。后面的问题可以通过为每个缓存的函数创建柯粒化副本去规避；从而提前把配置项赋值给每个函数调用。

下面是 `asRectangle` mixin 的函数被柯粒化允许 `grow` 和 `shrink` 函数被自定义：

```js
Function.prototype.curry = function () {
  var fn = this;
  var args = [].slice.call(arguments, 0);
  return function () {
    return fn.apply(this, args.concat([].slice.call(arguments, 0)));
  };
};

var asRectangle = (function () {
  function area() {
    return this.length * this.width;
  }
  function grow(growBy) {
    (this.length += growBy), (this.width += growBy);
  }
  function shrink(shrinkBy) {
    (this.length -= shrinkBy), (this.width -= shrinkBy);
  }
  return function (options) {
    this.area = area;
    this.grow = grow.curry(options['growBy']);
    this.shrink = shrink.curry(options['shrinkBy']);
    return this;
  };
})();

asButton.call(RectangularButton.prototype);
asRectangle.call(RectangularButton.prototype, { growBy: 2, shrinkBy: 2 });

var button4 = new RectangularButton(2, 1, 'add', function () {
  return 'added';
});
button4.area(); //2
button4.grow();
button4.area(); //12
button4.fire(); //'added'
```

## 结束语

JavaScript 是函数和状态的混合物。状态通常来说就是对象实例，而且函数总是会被跨实例共用。或许把这两者明显的隔离开是我们想要的结果，然后用混入的方式再把两者关联起来？

特别的，函数式混入模式提供了清晰的框架。对象实例就是充当状态，函数被组织在一起，就像树上的水果被挑拣一样。
