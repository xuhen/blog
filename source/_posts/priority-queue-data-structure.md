---
title: 优先队列实现
date: 2020-04-26 15:03:51
tags:
    - algorithm
---

> 在数据结构大家庭中，优先队列也是一种重要的数据储存结构。

这里我们做一些假设
   1. 权重值是大于等于1的正整数
   2. 权重值越小，优先级越高
   3. 优先级越高的在队列前面，优先级低在队尾。

根据上面我们的假设知道，这种结构的入队逻辑比较复杂一点，每次入队，都根据当前条目的优先级插入队列，最终是一个根据优先级排好序的队列。

定义条目:
```
// 放条目和权重
function QElement(element, priority) {
    this.element = element;
    this.priority = priority
}
```

定义优先队列

```
function PriorityQueue() {
    this.items = [];
}

PriorityQueue.prototype.enqueue = function (element, priority) {
    var qElement = new QElement(element, priority);
    var contain = false;
    for (var i = 0; i < this.items.length; i++) {
        if (this.items[i].priority > qElement.priority) {
            this.items.splice(i, 0, qElement);
            contain = true;
            break;
        }
    }

    if (!contain) {
        this.items.push(qElement);
    }


}

PriorityQueue.prototype.dequeue = function () {
    if (this.isEmpty())
        return "Underflow";
    return this.items.shift();
}

PriorityQueue.prototype.front = function () {
    if (this.isEmpty())
        return "No elements in Queue";
    return this.items[0];
}

PriorityQueue.prototype.rear = function () {
    if (this.isEmpty())
        return "No elements in Queue";
    return this.items[this.items.length - 1];
}

PriorityQueue.prototype.isEmpty = function () {
    return this.items.length === 0;
}

PriorityQueue.prototype.printPQueue = function () {
    var str = "";
    for (var i = 0; i < this.items.length; i++)
        str += this.items[i].element + " ";
    return str;
}


var priorityQueue = new PriorityQueue();

console.log(priorityQueue.isEmpty());
console.log(priorityQueue.front());
priorityQueue.enqueue("Sumit", 2);
priorityQueue.enqueue("Gourav", 1);
priorityQueue.enqueue("Piyush", 1);
priorityQueue.enqueue("Sunny", 2);
priorityQueue.enqueue("Sheru", 3);


console.log(priorityQueue);
console.log(priorityQueue.isEmpty());
console.log(priorityQueue.front());
```