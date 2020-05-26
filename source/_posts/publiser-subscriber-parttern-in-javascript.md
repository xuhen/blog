---
title: javaScript中的发布订阅模式
date: 2020-05-16 11:27:01
tags:
    - design parttern
    - js
---

> 观察者模式定义了对象间的一种一对多的依赖关系，当一个对象状态发生改变时，所有依赖它的对象都将得到通知，并自动更新。
>
> 观察者模式还有另一个别名，“发布-订阅模式”，或者“订阅-发布模式”，订阅者和订阅目标是联系在一起的，当订阅目标发生改变时，逐个通知订阅者。
>
> 其实24种基本的设计模式中并没有发布订阅模式，它只是观察者模式的别称。

## 两种模式的区别

![different sub and pub](http://xuheng.inject.top/images/pub-sub.png)

从上图可以看出，发布订阅模式比观察者模式多了个事件通道，事件通道作为调度中心，管理事件的订阅和发布工作，彻底隔绝了订阅者和发布者的依赖关系。

下面看下它们的代码实现

## 发布-订阅模式

```
class PubSub {
  constructor() {
    this.subscribers = {};
  }

  subscrib(type, callback) {
    if (!this.subscribers[type]) {
      this.subscribers[type] = [];
    }

    this.subscribers[type].push(callback);
  }

  publish(type, ...args) {
    var callbacks = this.subscribers[type] || [];
    callbacks.forEach((callback) => {
      callback(...args);
    })
  }
}

let pubSub = new PubSub();

pubSub.subscrib('SMS', function (...p) {
  console.log('订阅者1');
  console.log(...p);
});
pubSub.subscrib('SMS', function (...p) {
  console.log('订阅者2');
  console.log(...p);
});

pubSub.publish('SMS', 'I published `SMS` event');
```

## 观察者模式

```
class Subject {
  constructor() {
    this.observers = [];
  }

  add(observer) {
    if (observer) {
      this.observers.push(observer);
    }
  }
  notify(...args) {
    this.observers.forEach((observer) => {
      observer.update(...args);
    })
  }
}

class Observer {
  update(...args) {
    console.log(...args);
  }
}

var observer1 = new Observer();
var observer2 = new Observer();
var sub = new Subject();
sub.add(observer1);
sub.add(observer2);
sub.notify('I fired `SMS` event');
```

## 完整的发布-订阅模式代码
```
class Event {
    constructor() {
        this.clientList = {};
    }
    listen(key, fn) {
        if (!this.clientList[key]) {
            this.clientList[key] = [];
        }
        this.clientList[key].push(fn);
    }
    trigger() {
        var key = Array.prototype.shift.call(arguments),
            fns = this.clientList[key];
        if (!fns || fns.length === 0) {
            return false;
        }
        for (var i = 0, fn; fn = fns[i++];) {
            fn.apply(this, arguments);
        }
    }
    remove(key, fn) {
        var fns = this.clientList[key];
        if (!fns) {
            return false;
        }

        if (!fn) {
            fns && (fns.length = 0);
        } else {
            for (var l = fns.length - 1; l >= 0; l--) {
            var _fn = fns[l];
            if (_fn === fn) {
                fns.splice(l, 1);
            }
            }
        }
    }

}

var event = new Event();

```

## 总结
从代码实现可以看出，发布-订阅者模式是面向调度中心编程的，而观察者模式则是面向目标和观察者编程的。前者用于解耦发布者和订阅者，后者用于耦合目标和观察者。