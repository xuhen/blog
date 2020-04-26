---
title: 二叉树的遍历（广度优先遍历）
date: 2020-04-26 11:33:47
tags:
    - algorithm
---

> 提到二叉树遍历，还有一种常见的就是广度(bfs)优先的遍历

如果按照下面的示例，输出结果为：

1, 2, 3, 4, 5, 6, 7;

```
示例二叉树：


               1
          /          \
        2            3
      /    \         /    \
    4       5     6     7

```

二叉树的广度遍历需要用到队列(Queue) 这样的数据结构：

```
function Queue() {
    this.queue = [];
}

Queue.prototype.enqueue = function (item) {
    if (item) {
        this.queue.push(item);
    }
}
Queue.prototype.dequeue = function () {
    if (!this.isEmpty()) {
        return this.queue.shift();
    }
}
Queue.prototype.isEmpty = function () {
    return this.queue.length === 0;
}
```

节点我们定义成：
```
function Node(value) {
    this.value = value;
    this.left = null;
    this.right = null;
}

var n1 = new Node(1)
var n2 = new Node(2)
var n3 = new Node(3)
var n4 = new Node(4)
var n5 = new Node(5)
var n6 = new Node(6)
var n7 = new Node(7)

n1.left = n2;
n1.right = n3;
n2.left = n4;
n2.right = n5;
n3.left = n6;
n3.right = n7;
```


广度遍历：

```
function levelorderTraversal(root) {
    var q = new Queue(), node;
    q.enqueue(root);

    while (!q.isEmpty()) {
        node = q.dequeue();
        console.log(node.value);
        if (node.left) {
            q.enqueue(node.left);
        }
        if (node.right) {
            q.enqueue(node.right);
        }
    }
}

levelorderTraversal(n1);
```