---
title: 理解nodejs中的cluster模块
date: 2020-02-08 21:38:36
tags:
    - nodeJS
---

> NodeJS 的进程是单进程的，这就意味着它没有利用完多核系统的资源。如果你有个8核的CPU然后跑个命令 `node app.js` 它只会跑在单进程里，浪费了剩下的CPUs。
>
> 幸运的是，NodeJS 提供了 cluster 模块去帮助我们创建程序去利用所有的CUPs。而cluster模块利用所有这些CUP是通过 fork 进程实现的，和Unix 的 fork() 差不多。


## 介绍cluster模块

当引入cluster模块后，parent/master进程分出任意多的child/worker进程，父进程和子进程之间是通过IPC (Inter Process Communication)进行通信的。请记住，这些进程之间没有共享内存。

下面的这段话是从NodeJS官网摘取的一段：


* Node.js 的单实例跑在单线程中的。为了利用多核的系统，开发者就得发起一组Node.js进程去处理负载。
* 而这个cluster模块让创建子进程变得相对容易，而且这些子进程可以共用同一个port.
* 子进程是用`child_process.fork()`方法创建的，所以它们能通过IPC同父进程进行通信。而这个 `child_process.fork()` 方法是 `child_process.spawn()` 方法的特殊例子，专门用于创建新的 Node.js进程的。就像 `child_process.spawn()` 一样，通过`child_process.fork()`方法返回了 `ChildProcess` 的实例。这个 `ChildProcess` 有内在的沟通渠道允许消息在父进程和子进程之间来回传递。而这个传递的方法就是 `send()`。
* 我们得记住一点就是，创建的子进程除了和父进程有个IPC进行沟通以外，它们之间其他的部分都是独立的。每个进程有它们自己的内存、自己的V8实例。因为需要花费额外的资源，产生大量子进程是不明智的。

## 一个简单的示例

下面的示例：
1、创建了个主进程，并且得到了CPU的个数，然后为每个CPU生成了一个工作进程
2、每个工作进程打印了一条日志后退出

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

  process.exit();
}

function childProcess() {
  console.log(`Worker ${process.pid} started and finished`);

  process.exit();
}
```

下面是执行这个文件的结果

```
$ node app.js

Master 8463 is running
Forking process number 0...
Forking process number 1...
Forking process number 2...
Forking process number 3...
Worker 8464 started and finished
Worker 8465 started and finished
Worker 8467 started and finished
Worker 8466 started and finished
```

## 父子进程间通信

当子进程创建成功后，这个子进程和父进程之间就创建了IPC信道，允许我们用 `send()` 方法在它们之间进行通信，这个方法也接收JavaScript的对象作为参数。这里依然要强调下，这个父进程和子进程是不同的进程（而不是不同的线程），因此我们不能共用内存去作为通信的方法。

从父进程的角度，我们能通过进程引用向子进程发送消息，i.e. `someChild.send({ ... })`。而站在子进程的角度，我们能用当前的 `process` i.e. `process.send() `引用向父进程发送消息。

基于上面的代码我们加上了子进程向父进程通信的代码

```
function childProcess() {
  console.log(`Worker ${process.pid} started`);

  process.on('message', function(message) {
    console.log(`Worker ${process.pid} recevies message '${JSON.stringify(message)}'`);
  });

  console.log(`Worker ${process.pid} sends message to master...`);
  process.send({ msg: `Message from worker ${process.pid}` });

  console.log(`Worker ${process.pid} finished`);
}
```

下面是主进程向子进程发起的通信

```
let workers = [];

function masterProcess() {
  console.log(`Master ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    console.log(`Forking process number ${i}...`);

    const worker = cluster.fork();
    workers.push(worker);

    // Listen for messages from worker
    worker.on('message', function(message) {
      console.log(`Master ${process.pid} recevies message '${JSON.stringify(message)}' from worker ${worker.process.pid}`);
    });
  }

  // Send message to the workers
  workers.forEach(function(worker) {
    console.log(`Master ${process.pid} sends message to worker ${worker.process.pid}...`);
    worker.send({ msg: `Message from master ${process.pid}` });
  }, this);
}
```

这个 `masterProcess` 函数我们分成两块说明。在第一个循环中，我们生成了CPUs个数的工作进程。`cluster.fork()` 方法返回了 `worker` 代表的工作进程，我们把所有的子进程引用放入一个数组，并且注册了消息事件函数用于接收工作进程发来的消息。

在第二个循环中，我们从主进程分别向每个子进程发送了消息。

下面是运行的结果

```
$ node app.js

Master 4045 is running
Forking process number 0...
Forking process number 1...
Master 4045 sends message to worker 4046...
Master 4045 sends message to worker 4047...
Worker 4047 started
Worker 4047 sends message to master...
Worker 4047 finished
Master 4045 recevies message '{"msg":"Message from worker 4047"}' from worker 4047
Worker 4047 recevies message '{"msg":"Message from master 4045"}'
Worker 4046 started
Worker 4046 sends message to master...
Worker 4046 finished
Master 4045 recevies message '{"msg":"Message from worker 4046"}' from worker 4046
Worker 4046 recevies message '{"msg":"Message from master 4045"}'
```

## 结束语

cluster 模块提供了 NodeJS 利用系统所有CPUs的能力。虽然在这里没有看到 `Child Process` 模块，但它和 `cluster` 模块是个很好的互补，并且提供了很多工具用于操作进程，比如 `start`, `stop` 和 `pipe` `input/out`, 等等。