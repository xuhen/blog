---
title: 使用pm2去管理nodejs的cluster模块
date: 2020-02-10 09:20:59
tags:
    - nodeJS
---

> cluster模块能让我们创建工作进程去提高nodejs应用的性能。这对于网络应用是特别重要的，主进程接收所有的网络请求，然后往工作进程负载均衡。
>
> 但是这些管理工作必然会增加一定的消耗，我们的应用必须去管理进程之间的各种复杂的通信：如果一个工作进程意外退出，怎样优雅的退出这个工作进程呢，那又或者需要去重新气筒所有工作进程呢，等等...
>
> 在这个文章中我们要介绍下[PM2](https://pm2.keymetrics.io/ "pm2")这个工具。虽然它是一个通用的进程管理器，这意味着它能管理所有类型的进程，比如 python, ruby ... 而不仅只有nodejs进程，使用cluster模块的nodejs应用一般都是由这个工具管理的。

## 介绍PM2

正如之前介绍的，PM2是一个通用进程管理器，它也是一个程序，去控制其他进程的执行（就像一个 python 程序检查你是否有新邮件一样）和做一些管理工作：检查你的进程是否在运行；如果一些意外导致进程退出就重启进程；记录输出，等等

对我们来说PM2最重要的工作就是简化NodeJS作为一个集群的运行。是的，你写你的应用程序是不用担心cluster模块，因为PM2会创建一些数量的工作进程去运行你的应用程序。

## cluster模块最难的部分

下面让我们看一个示例，用cluster模块创建一个简单的HTTP服务器。主进程会创建CUPs个数的工作进程，然后也会在工作进程退出的情况下创建新的工作进程。

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

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    console.log(`Forking a new process...`);

    cluster.fork();
  });
}

function childProcess() {
  console.log(`Worker ${process.pid} started...`);

  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello World');

    process.exit(1);
  }).listen(3000);
}
```

这个工作进程是一个非常简单的HTTP服务器，监听3000端口，然后被硬编码返回一个 `Hello World`，最后退出（去模拟一个失败）

如果我们用 `$ node app.js` 跑下程序会看到下面的输出：

```
$ node app.js

Master 2398 is running
Forking process number 0...
Forking process number 1...
Worker 2399 started...
Worker 2400 started...
```

如果我们去浏览器打开 `http://localhost:3000` 链接，我们会看到 `Hello World` 然后控制台会打印下面的内容：

```
Worker 2400 died
Forking a new process...
Worker 2401 started...
```

那下面让我们看下PM2怎样简化我们的应用吧。

## PM2的方式

继续往下之前，请确保你的应用已经安装了PM2。它是被全局安装在你的系统的，比如 `$ npm install pm2 -g` 或者 `$ yarn global add pm2`

当使用PM2的时候，我们可以丢弃主进程相关的代码，因为那是PM2的工作，一个简单的HTTP服务就变成下面的样子了：

```
const http = require('http');

console.log(`Worker ${process.pid} started...`);

http.createServer((req, res) => {
  res.writeHead(200);
  res.end('Hello World');

  process.exit(1);
}).listen(3000);
```

现在我们跑下PM2 `$ pm2 start app.js -i 3`，你会看到下面的输出

> 这个 `-i` 选项是表明创建多少个实例。理想的情况下那个数字应该是和你的CPU核数相等。如果你不知道，你可以设置 `-i 0` 让PM2自动检测核数。

```
$ pm2 start app.js -i 3

[PM2] Starting /Users/blablabla/some-project/app.js in cluster_mode (3 instances)
[PM2] Done.

| Name      | mode    | status | ↺ | cpu | memory    |
| ----------|---------|--------|---|-----|-----------|
| app       | cluster | online | 0 | 23% | 27.1 MB   |
| app       | cluster | online | 0 | 26% | 27.3 MB   |
| app       | cluster | online | 0 | 14% | 25.1 MB   |
```

我们能通过`$ pm2 log` 看到应用的日志。现在让我们在浏览器打开 `http://localhost:3000`链接，在控制台会看到下面的输出：

```
PM2        | App name:app id:0 disconnected
PM2        | App [app] with id [0] and pid [1299], exited with code [1] via signal [SIGINT]
PM2        | Starting execution sequence in -cluster mode- for app name:app id:0
PM2        | App name:app id:0 online
0|app      | Worker 1489 started...
```

我们能看到PM2是怎样检测到我们的工作进程退出了，然后自动新开了个新的工作进程实例。

## 结束语

虽然NodeJS的cluster模块是个强大的途径去提高性能，但是它还是有一点复杂度的开销，因为需要管理应用可能遇到的所有场景：如果一个工作进程退出了，我们怎样不停机的情况下重启整个集群，等等。

PM2是一个进程管理器，特别为NodeJS集群量身打造的。它能让一个应用集群化；在没有代码的复杂度的情况下，重新开始或者重新加载应用，额外它还提供了工具去查看日志，和监控，等等。