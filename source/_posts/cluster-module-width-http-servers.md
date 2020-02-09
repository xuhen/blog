---
title: 在http服务中使用cluster模块
date: 2020-02-09 11:15:31
tags:
    - nodeJS
---

> cluster模块让我们提高了多核CPU系统的应用性能。因为不管我们是写 APIs、或者用ExpressJS去写web服务，我们的最终目标都是利用好当前机器的所有CPUs。
>
> 此外，cluster模块可以让我们做到，在各个工作进程中负载平衡掉请求，进而提高应用的吞吐量。
>
> 在这篇文章中，我们将说明怎样使用cluster模块去创建HTTP服务。

## 使用cluster模块创建HTTP服务

下面让我们用cluster模块创建一个简单的HTTP服务。

```
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  masterProcess();
} else {
  childProcess();
}

function masterProcess() {
  console.log(`Master ${process.pid} is running`);

  for (let i = 0; i < numCPUs; i++) {
    console.log(`Forking process number ${i}...`);
    cluster.fork();
  }
}

function childProcess() {
  console.log(`Worker ${process.pid} started...`);

  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello World');
  }).listen(3000);
}
```

我们把整个代码分成了两个部分，一部分是关于主进程的，另一部分是关于初始化工作进程的。`masterProcess` 函数为每个CPU创建了一个工作进程。`childProcess`函数就是简单的创建了HTTP服务，并监听3000端口，然后返回了`Hello World`和一个200的状态码。

如果你跑上面的代码会得到下面的结果

```
$ node app.js

Master 1859 is running
Forking process number 0...
Forking process number 1...
Forking process number 2...
Forking process number 3...
Worker 1860 started...
Worker 1862 started...
Worker 1863 started...
Worker 1861 started...
```

上面的代码可以理解为：初始进程（主进程）为每个CPU创建了新的工作进程，然后每个工作进程跑了一个HTTP服务去处理请求。这样可以很大程度提高我们服务的性能，因为这样我们是用四个进程处理一百万的请求，而不是一个进程处理这么大的量。

## cluster模块和网络连接

上面的例子比较简单，但是其隐藏了一些棘手的问题，这也是NodeJS为了简化开发者的开发体验。

在任何OS系统中，一个进程能用一个端口和其他系统进行通信，这就意味着一个端口只能被一个进程使用。那么，被主进程生成的这么多的子进程是怎样共用一个端口的呢？

这个简化的答案是，主进程是唯一监听这个端口的进程，并且在进入子进程的请求之间负载平衡。下面是官方文档的截取：

> 这些工作进程是用 `child_process.fork()` 方法产生的，因此它们能和父进程通过 IPC 进行通信，来回传递信息。
>
> 这个cluster模块支持两种方式进行请求连接的分配。
>
> 1、第一种方式（这是默认方式除了在windows系统外），是 `round-robin` 方式，主进程监听端口，接受新的请求链接，然后以`round-robin`的方式分配给这些工作进程，当然也有内部的一些智能方式避免工作进程的过度负载。
>
> 2、第二种方式是主进程创建一个监听套接字(listen socket) 然后发送给相关的工作进程。然后这个工作进程直接，接收进来的请求链接。
>
> 只要有工作进程还活着，服务器就会继续接收请求链接。如果没有工作进程还活着，已经存在的连接将会被关闭，新的连接请求会被拒绝。

## cluster模块负载平衡的替代方案

cluster 模块允许主进程接收请求，并在工作进程直接平衡负载。这是提高性能的一种方式，但不是唯一的方式。
在[Node.js process load balance performance: comparing cluster module, iptables and Nginx](https://medium.com/@fermads/node-js-process-load-balancing-comparing-cluster-iptables-and-nginx-6746aaf38272 "comparing") 文章中，你能找到 node cluster module, iptables 和 nginx reverse proxy 的性能比较。

## 结束语

现如今，性能是任何网络应用必须考虑的点了，我们需要服务的高吞吐量和快速返回数据。
这个cluster模块是一种可能的解决方法，它允许我们用主进程为每个处理器创建一个工作进程，然后每个工作进程跑一个HTTP服务。cluster提供了两个比较好的特性：

* 通过创建IPC信道，和允许通过 `process.send()` 方法发送消息，大大简化了主进程和工作进程之间的通信机制。
* 允许工作进程共享同一个端口。这是通过主进程接收请求连接，然后负载平衡给其他工作进程实现的。