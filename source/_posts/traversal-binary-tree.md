---
title: 二叉树的遍历（前序，中序，后序）
date: 2020-04-26 10:51:57
tags:
    - algorithm
---

> 二叉树，这样的结构还是很常见的，二叉树的遍历更是数型算法的基础。那今天就介绍下二次数的非递归遍历。二叉树分前序遍历、中序遍历、后序遍历。

如果按照下面的树：
前序： 根 -> 左 -> 右     =>  1, 2, 4, 5, 3, 6, 7
中序： 左 -> 根 -> 右     =>  4, 2, 5, 1, 6, 3, 7
后序： 左 -> 右 -> 根     =>  4, 5, 2, 6, 7, 3, 1



```
示例二叉树：


               1
          /          \
        2            3
      /    \         /    \
    4       5     6     7

```

二叉树的三种遍历都要使用stack（栈）这样的数据结构。

```
function Stack() {
    this.stack = [];
}

Stack.prototype.isEmpty = function () {
    return this.stack.length === 0;
}

Stack.prototype.push = function (item) {
    if (item) {
        this.stack.push(item)
    }
}
Stack.prototype.pop = function () {
    if (!this.isEmpty()) {
        return this.stack.pop();
    }
}

Stack.prototype.peek = function () {
    if (!this.isEmpty()) {
        return this.stack[this.stack.length - 1]
    }
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

前序遍历
```
function preorderTraversal(root) {
    var node = root;
    var stack = new Stack();
    while (node !== null || !stack.isEmpty()) {
        while (node !== null) {
            console.log(node.value);
            stack.push(node);
            node = node.left;
        }
        if (!stack.isEmpty()) {
            node = stack.pop();
            node = node.right;
        }
    }
}

preorderTraversal(n1)
```

中序遍历：

```
function inorderTraversal(root) {
    var stack = new Stack();
    var node = root;
    while (node !== null || !stack.isEmpty()) {
        while (node !== null) {
            stack.push(node);
            node = node.left;
        }
        if (!stack.isEmpty()) {
            node = stack.pop();
            console.log(node.value);
            node = node.right;
        }
    }
}

inorderTraversal(n1)
```

后序遍历：

```
function postorderTraversal(root) {
    var stack = new Stack();
    var node = root;
    var lastVisit = root;
    while (node !== null || !stack.isEmpty()) {
        while (node !== null) {
            stack.push(node);
            node = node.left;
        }
        node = stack.peek();
        if (node.right === null || node.right === lastVisit) {
            console.log(node.value);
            stack.pop();
            lastVisit = node;
            node = null
        } else {
            node = node.right;
        }
    }

}

postorderTraversal(n1)
```