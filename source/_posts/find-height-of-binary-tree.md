---
title: 求二叉树的深度(递归和非递归两种)
date: 2020-05-07 11:11:29
tags:
    - algorithm
---

> 有两种定义二叉树深度的方式：
> 1. 从根节点到最长路径所经过的节点数
> 2. 从根节点到最长路径上的边数
>
> 在这里我们遵循第一条定义


<img src="http://xuheng.inject.top/images/tree122.gif" alt="tree" style='width: 200px'/>

如上图所示，树的深度为：3


## 递归方式

```
function maxDepth(node) {
    if(node === null) {
        return 0;
    }

    var lDepth = maxDepth(node.left);
    var rDepth = maxDepth(node.right);

    if(lDepth > rDepth) {
        return lDepth + 1;
    } else {
        return rDepth + 1;
    }
}
```

## 非递归方式

```
function maxLevel(node) {
    if(node === null) return 0;
    var queue = [];
    var height = 0;
    queue.push(node);

    while(true) {
        var nodeCount = queue.length;
        if(nodeCount === 0) {
            return height;
        }
        height++;
        while(nodeCount > 0) {
            var newNode = queue.shift();

            if(newNode.left !== null) {
                queue.push(newNode.left);
            }
            if(newNode.right !== null) {
                queue.push(newNode.right);
            }

            nodeCount--;
        }
    } 
}
```

