---
title: 用PM2优雅的停掉NodeJS服务器
date: 2020-02-11 21:32:09
tags:
    - nodeJS
---

> 现在你创建了个NodeJS的服务器，接收了成百上千的请求，你感到非常满足；正如其他的软件一样，你会发现bug，你会增加新的功能。显然，你需要停掉你的NodeJS进程重启你的服务，这样新的代码才会生效。那么问题来了：*你要怎样停掉服务，并且同时要继续服务好已经进来的请求呢？*。


## 开启一个HTTP服务器

我们在看怎样停止一个HTTP服务器之前让我们先看下怎样创建它。下面是一个简单的`ExpressJS` 服务器。

```
const express = require('express')

const expressApp = express()

// Responds with Hello World or optionally the name you pass as path param
expressApp.get('/hello/:name?', function (req, res) {
  const name = req.params.name

  if (name) {
    return res.send(`Hello ${name}!!!`)
  }

  return res.send('Hello World !!!')
})

// Start server
const server = expressApp.listen(3000, function () {
  console.log('App listening on port 3000!')
})
```

## 怎样停掉一个HTTP服务器？

停掉HTTP服务器的最佳方式是调用 `server.close()` 方法，它将停止服务器接收新的连接但同时保证现有的请求都得到回应才停止服务。

下面的代码新增加了个 `/close` 请求端点，一但被请求，就会停掉HTTP服务，然后退出应用（也就是停掉nodejs进程）：

```
app.get('/close', (req, res) => {
  console.log('Closing the server...')

  server.close(() => {
    console.log('--> Server call callback run !!')

    process.exit()
  })
})
```


## 优雅的关闭/重启服务，用或者不用PM2

优雅的关闭服务器，其实我们就是想关闭对新请求的通道，但是会保留正在处理的请求通道。

当我们用像PM2这样的进程管理器时，我们管理的是进程集群，每个进程都扮演着HTTP服务器的角色。而PM2实现优雅的重启是下面的原因：

* 发送一个 `SIGNINT` 信号给每个工作进程，
* 工作进程会捕获这个信号，清理或者释放被使用的资源，然后完结这个进程，
* 最后PM2新建一个进程

因为进程集群处理重启是连续进行的，因此用户不会受到影响，因为这期间总是有进程在工作和处理请求。

当我们部署新代码或者重启我们的服务器时，保证这些改变不会影响持续进来的请求是非常重要的。我们能通过下面的代码实现：

```
// Graceful shutdown
process.on('SIGINT', () => {
  const cleanUp = () => {
    // Clean up other resources like DB connections
  }

  console.log('Closing server...')

  server.close(() => {
    console.log('Server closed !!! ')

    cleanUp()
    process.exit()
  })

  // Force close server after 5secs
  setTimeout((e) => {
    console.log('Forcing server close !!!', e)

    cleanUp()
    process.exit(1)
  }, 5000)
})
```

当进程收到 `SINGINT` 信号时，它会执行 `server.close()` 函数去避免接收更多的请求，然后一旦进程被关闭，我们清理掉被应用使用的资源：比如关闭数据库连接，关闭打开的文件等等；执行完 `cleanUp()` 函数后我们就用 `process.exit()` 函数退出进程了。为了考虑特殊情况，为了避免代码花费太多的时间在关闭服务器上，我们强制服务器在一定时间后结束。

## 结束语

当创建HTTP服务器时，不管是服务页面的应用还是服务接口的应用，我们都需要考虑到，我们一定会去增加新功能和修改bug，因此我们需要以尽量减少对用户的影响为思考点。

以集群模式，运行nodejs进程是一个常用方式，这样我们能提高应用的性能，我们要想办法优雅的关闭所有进程，而且不会影响进来的请求。

当我们讨论的是 HTTP 服务器时，用 `process.exit()` 方法结束 node 进程是不够的，因为它会意外的结束所有的通信；而相反，我们要首先停止接收新的连接，释放被用的资源，最后才是停止这个进程。