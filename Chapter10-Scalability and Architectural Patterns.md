# Scalability and Architectural Patterns
在早期，`Node.js`主要用于非阻塞的`Web`服务器，它的原名实际上是`web.js`。其创建者`Ryan Dahl`很快意识到了该平台的潜力，并开始使用工具对其进行扩展，以便在`JavaScript / non-blocking paradigm`之上创建任何类型的服务器端应用程序。`Node.js`的特点对于分布式系统的实现是完美的，由分布式系统组成的节点通过网络协调运作。`Node.js`诞生了。与其他网络平台不同的是，非阻塞这个词在应用程序的生命周期很早就进入了`Node.js`开发者的词汇表中，主要是因为它具有单线程特性，不能利用机器的所有资源，但通常还有更多深刻的原因。正如我们将在本章中看到的，扩展应用程序不仅意味着增加其容量，使其能够更快地处理更多的请求；这也是实现高可用性和容错性的关键途径。令人惊讶的是，它也可以将应用程序的复杂性分解为更易于管理的部分。可伸缩性是一个具有多个面的概念，其中六个是精确的，就像一个立方体的面 - 即多维数据集的面。

在本章中，我们将学习以下主题：

* `scale cube`是什么
* 如何通过运行同一应用程序的多个实例来进行扩展
* 如何在扩展应用程序时利用负载平衡器
* 什么是服务注册表，以及如何使用它
* 如何从单片应用程序设计微服务架构
* 如何通过使用一些简单的架构模式来集成大量的服务

## 介绍应用程序缩放
在我们深入探讨一些实际的模式和例子之前，我们应该说一下应用程序扩展的原因以及如何实现。

### 缩放`Node.js`应用程序
我们已经知道，典型的`Node.js`应用程序的大部分任务都是在单个线程的上下文中运行的。在`Chapter1-Welcome to the Node.js Platform`，我们了解到这不是一个限制，而是一个优点，因为它允许应用程序优化处理并发请求所需资源的使用情况，这要归功于非阻塞`I / O`范例。由非阻塞`I / O`充分利用的单线程对于每秒处理中等数量的请求（通常每秒几百次（这在很大程度上取决于应用程序））的应用程序奇妙地工作。假设我们使用的是商品硬件，那么无论服务器的功能如何强大，单个线程所能支持的容量都是有限的，因此，如果我们想要将`Node.js`用于高负载应用程序，唯一的方法就是将其扩展多个进程和机器。
但是，工作负载不是缩放`Node.js`应用程序的唯一原因;事实上，使用相同的技术，我们可以获得其他所需的属性，如可用性和容错性。可伸缩性也是适用于应用程序的大小和复杂性的概念;实际上，可拓展性是设计软件的另一个重要因素。 `JavaScript`是一个谨慎使用的工具，缺乏类型检查和许多陷阱可能是应用程序增长的一个障碍，但是通过纪律和精确的设计，我们可以把它变成一个优势。使用`JavaScript`，我们经常被迫使应用程序变得简单，并将其拆分成易于管理的部分，使其更易于扩展和分发。

### 可扩展性的三个维度
在谈到可伸缩性时，要理解的第一个基本原则是负载分布，这是将应用程序的负载分散到多个进程和机器上。有很多方法可以实现这一点，`Martin L. Abbott`和`Michael T. Fisher`提出的“可扩展性的艺术”一书提出了一个巧妙的模型来表示它们，称为`scale cube`。该模型描述了以下三个方面的可扩展性：

* x轴：克隆，或者说复制
* y轴：按服务/功能分解
* z轴：按数据分区分割

这三个维度可以表示为一个立方体，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-2-6/77680588.jpg)

多维数据集的左下角表示应用程序在单个代码库（单片应用程序）中具有所有功能和服务，并在单个实例上运行。对于处理小型工作负载的应用程序或处于开发的早期阶段，这是一种常见的情况。

单片非缩放应用程序最直观的发展是沿着`x`轴向右移动，这很简单，大部分时间价格便宜（在开发成本方面）并且非常有效。这个技术背后的原理是基本的，就是克隆相同的应用程序`n`次，并让每个实例处理工作负载的`1 / n`。

沿`y`轴缩放意味着根据其功能，服务或用例来分解应用程序。在这种情况下，分解意味着创建不同的，独立的应用程序，每个应用程序都有其自己的代码库，有时还有自己的专用数据库，甚至是独立的UI。例如，常见的情况是将负责管理的应用程序的一部分与面向公众的产品分开。另一个例子是提取负责用户认证的服务，创建一个专用的认证服务器。按照功能划分应用程序的标准主要取决于其业务需求，用例，数据以及其他因素，我们将在本章后面介绍。有趣的是，这不仅是应用程序的体系结构，还是从开发的角度来看，它是最大的影响。正如我们将看到的，微服务是一个术语，目前通常与细粒度的`y`轴缩放关联。

最后一个缩放维度是`z`轴，应用程序以这样一种方式分割，即每个实例只负责整个数据的一部分。这是一种主要用于数据库的技术，也是水平分区或分片的名称。在此设置中，同一个应用程序有多个实例，每个实例都在数据的一个分区上运行，这是使用不同的标准确定的。例如，我们可以根据他们的国家（列表分区）或者基于他们姓氏的起始字母（范围分区）划分应用程序的用户，或者让一个散列函数决定每个用户所属的分区（散列分区）。然后可以将每个分区分配给我们应用程序的特定实例。使用数据分区需要在每个操作之前进行查找步骤，以确定应用程序的哪个实例负责给定的数据。正如我们所说的，数据分区通常在数据库级应用和处理，因为它的主要目的是克服处理大型单一数据集（磁盘空间有限，内存和网络容量有限）的问题。在应用程序级别应用它仅仅适用于复杂的分布式体系结构或非常特殊的用例，例如在构建依赖于数据持久性定制解决方案的应用程序，使用不支持分区的数据库时，或者在`Google`上构建应用程序时规模。考虑到其复杂性，只有在尺度立方体的`x`轴和`y`轴被充分利用之后，才应该考虑沿着`z`轴缩放应用程序。

在下一节中，我们将重点介绍两种最常用和最有效的技术来扩展`Node.js`应用程序，即通过功能/服务进行克隆和分解。

## 克隆和负载平衡
传统的多线程`Web`服务器通常只在分配给一台机器的资源不能再升级的时候才进行扩展，否则这个服务器的成本将高于简单地启动另一台机器的成本。 通过使用多个线程，传统的Web服务器可以利用服务器的所有处理能力，使用所有可用的处理器和内存。 但是，使用单个`Node.js`进程很难做到这一点，它是单线程的，在`64`位计算机上默认具有`1.7 GB`的内存限制（这需要增加一个名为`--max_old_space_size`的特殊命令行选项）。这意味着`Node.js`应用程序通常比传统的`Web`服务器更快地缩放，即使在单个机器的情况下，也能够利用其所有资源。

> 在`Node.js`中，垂直缩放（向单个机器添加更多资源）和水平缩放（将更多机器添加到基础架构）几乎是等价的概念; 事实上这两种技术类似，都是增加服务器的负载能力。

不要被愚弄，把这看作是一个缺点。相反，几乎被迫扩展对应用程序的其他属性，特别是可用性和容错性具有有益的影响。实际上，通过克隆来扩展`Node.js`应用程序相对比较简单，即使不需要获取更多的资源，也只是为了具有冗余的容错设置的目的而实现。
这也促使开发人员从应用程序的早期阶段考虑可伸缩性，确保应用程序不依赖任何不能在多个进程或机器间共享的资源。实际上，扩展应用程序的绝对先决条件是每个实例不必将通用信息存储在无法共享的资源（通常是硬件，如内存或磁盘）上。例如，在`Web`服务器中，将会话数据存储在内存中或磁盘上是一种惯例，不适合缩放;相反，使用共享数据库将确保每个实例都可以访问相同的会话信息，无论它在哪里部署。
现在我们来介绍扩展`Node.js`应用程序的最基本机制：集群模块。

### `cluster`模块
在`Node.js`中，在单个机器上运行的不同实例之间分配应用程序负载的最简单模式是使用作为核心库一部分的`cluster`模块。 群集模块简化了相同应用程序的新实例的分叉，并自动将传入的连接分配到其中，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-2-6/58040629.jpg)

主进程负责产生大量进程（`worker`），每个进程代表我们想要扩展的应用程序的一个实例。每个传入连接然后分布在克隆的`worker`，分散在他们的负载。

#### 关于`cluster`模块行为的注意事项
在`Node.js 0.8`和`0.10`中，`cluster`模块在工作人员之间共享相同的服务器套接字，并离开操作系统，负载平衡跨可用工作者的传入连接。但是，这种方法存在问题。实际上，操作系统用于在工作人员之间分配负载的算法并不意味着对网络请求进行负载平衡，而是调度进程的执行。因此，在所有情况下，分配并不总是一致的；往往只有一小部分工人获得了大部分的工作量。这种类型的行为对于操作系统调度程序是有意义的，因为它着重于最小化不同进程之间的上下文切换。简而言之，`cluster`模块在`Node.js <= 0.10`中不能充分发挥其潜力。
但是，情况从版本`0.11.2`开始变化，在主进程中包含明确的循环负载平衡算法，这确保请求在所有工作者中均匀分布。新的负载均衡算法默认情况下在`Windows`以外的所有平台上启用，可以通过设置变量`cluster.schedulingPolicy`，使用常量`cluster.SCHED_RR`（循环）或`cluster.SCHED_NONE`（由操作系统处理）。

> 轮循算法轮流在可用服务器上均匀分配负载。第一个请求被转发到第一个服务器，第二个请求转发到列表中的下一个服务器，依此类推。 当列表结束时，迭代从头开始。 这是最简单和最常用的负载均衡算法之一；然而，这不是唯一的一个。 更复杂的算法允许分配优先级，选择负载最少的服务器或响应时间最快的服务器。 您可以在这两个`Node.js`问题中找到关于集群模块演变的更多细节： https://github.com/nodejs/node-v0.x-archive/issues/4435 和 https://github.com/nodejs/node-v0.x-archive/issues/3241

#### 建立一个简单的HTTP服务器
现在开始研究一个例子。 让我们构建一个小型的`HTTP`服务器，使用集群模块进行克隆和负载平衡。 首先，我们需要一个应用程序来扩展；对于这个例子我们不需要太多，只是一个非常基本的`HTTP`服务器。

我们创建一个名为`app.js`的文件，其中包含以下代码：

```javascript
const http = require('http');
const pid = process.pid;
http.createServer((req, res) => {
  for (let i = 1e7; i > 0; i--) {}
  console.log(`Handling request from ${pid}`);
  res.end(`Hello from ${pid}\n`);
}).listen(8080, () => {
  console.log(`Started ${pid}`);
});
```

我们刚刚构建的`HTTP`服务器通过发回包含`PID`的消息来响应任何请求; 这将有助于识别哪个应用程序实例正在处理请求。另外，为了模拟一些实际的`CPU`工作，我们执行一个空循环`1000`万次；没有这个，考虑到我们要为这个例子运行的小规模的测试，服务器负载几乎是没有的。

> 我们想扩展的`app`模块可以是任何东西，也可以使用Web框架来实现，例如`Express`。

现在，我们可以像往常一样运行应用程序，并使用浏览器或`curl`向`http://localhost:8080`发送请求，检查是否所有程序都按预期工作。

我们也可以尝试测量服务器每秒只能使用一个进程处理的请求；为此，我们可以使用网络基准测试工具，如[siege](http://www.joedog.org/siege-home)或[Apache ab](http://httpd.apache.org/docs/2.4/programs/ab.html)：

```bash
siege -c200 -t10S http://localhost:8080
```

用`ab`，命令行会非常相似：

```bash
ab -c200 -t10 http://localhost:8080/
```

上述命令将以`200`个并发连接加载服务器`10`秒钟。 作为参考，具有`4`个处理器的系统的结果是每秒`90`个事务的顺序，平均`CPU`利用率仅为`20％`。

> 请记住，我们将在本章中执行的负载测试故意做成最简单和最小的，仅供参考和学习之用。他们的结果不能提供我们正在分析的各种技术的性能的`100％`准确的评估。

![](http://oczira72b.bkt.clouddn.com/18-2-6/82010925.jpg)

#### 使用`cluster`模块进行扩展
现在让我们尝试使用集群模块来扩展我们的应用程序。 我们来创建一个名为`clusteredApp.js`的新模块：

```javascript
const cluster = require('cluster');
const os = require('os');

if(cluster.isMaster) {
  const cpus = os.cpus().length;
  for (let i = 0; i < cpus; i++) {  // [1]
    cluster.fork();
  }
} else {
  require('./app');  // [2]
}
```

正如我们所看到的，使用`cluster`模块只需要很少的努力。我们来分析一下发生的事情：

* 当我们从命令行启动`clusteredApp`时，我们实际上正在执行主进程。`cluster.isMaster`变量设置为`true`，我们需要做的唯一工作是使用`cluster.fork()`来`fork`当前进程。 在前面的示例中，我们启动的系统中的CPU数量与可用的所有处理能力相同。
* 当从主进程执行`cluster.fork()`时，当前主模块（`clusteredApp`）再次运行，但是这次是工作模式（`cluster.isWorker`设置为`true`，而`cluster.isMaster`为`false`）。当应用程序作为`worker`运行时，它可以开始做一些实际的工作。 在我们的例子中，我们加载了`app`模块，它实际上启动了一个新的`HTTP`服务器。

> 记住每个`worker`都是一个不同的`Node.js`进程，它有自己的事件循环，内存空间和加载的模块。

有趣的是，注意到集群模块的使用基于循环模式，这使得运行多个应用程序的实例变得非常简单：

```javascript
if (cluster.isMaster) {
  // fork()
} else {
  // do work
}
```

在底层，集群模块使用了`child_process.fork() API`（我们已经在`Chapter 9, Advanced Asynchronous Recipes`中已经遇到了这个`API`），因此我们也在`master`和`worker`之间有一个可用的通信通道。 工人的实例可以通过变量`cluster.workers`访问，所以向所有人发送消息就像运行下面几行代码一样简单：

```javascript
Object.keys(cluster.workers).forEach(id => {
  cluster.workers[id].send('Hello from the master');
});
```

现在，让我们尝试以集群模式运行我们的`HTTP`服务器。 我们可以像往常一样启动`clusteredApp`模块来做到这一点：

```bash
node clusteredApp
```

如果我们的机器有多个处理器，我们应该看到一些`worker`正在被主进程一个接一个地启动。例如，在一个有四个处理器的系统中，终端应该是这样的：

```
Started 14107
Started 14099
Started 14102
Started 14101
```

如果我们现在尝试使用`URL http://localhost：8080`再次访问我们的服务器，我们应该注意到每个请求都会返回一个带有不同`PID`的消息，这意味着这些请求已经由不同的`worker`处理，确认 负载正在其中分配。

现在我们可以尝试再次加载测试我们的服务器：

```bash
siege -c200 -t10S http://localhost:8080
```

这样，我们就能够发现通过在多个进程中扩展应用程序所获得的性能提升。 作为参考，通过在具有`4`个处理器的`Linux`系统中使用`Node.js 6`，在平均`CPU`负载为`90％`的情况下，性能提高应该是`3`倍（为`270 trans / sec`，比起`90 trans / sec`）。

#### `cluster`模块的可拓展性和可用性
正如我们已经提到的那样，扩展应用程序还带来了其他优点，特别是即使在出现故障或崩溃时也能保持一定的服务水平的能力。 这个属性也被称为弹性，它有助于系统的可用性。

通过启动同一应用程序的多个实例，我们正在创建一个冗余系统，这意味着如果一个实例由于某种原因而关闭，我们仍然有其他实例可以为请求提供服务。 这种模式使用集群模块非常简单。 让我们看看它是如何工作的！

我们以上一节的代码为起点。特别是，我们修改`app.js`模块，使其在随机时间间隔后崩溃：

```javascript
// 在app.js的最后
setTimeout(() => {
  throw new Error('Ooops');
}, Math.ceil(Math.random() * 3) * 1000);
```

在这种变化的情况下，我们的服务器在`1`到`3`之间的随机数字时间之后退出，出现错误。在真实的情况下，这会导致我们的应用程序停止工作，当然，服务请求，除非我们使用一些外部工具来监视其状态并自动重启。但是，如果我们只有一个实例，那么由应用程序的启动时间引起的重新启动之间可能会有一个不可忽略的延迟。 这意味着在这些重新启动期间，应用程序不可用。拥有多个实例会确保我们总是有一个备份系统来处理即将到来的请求，即使其中一个工作者失败。

使用`cluster`模块，只要我们检测到一个错误代码被终止，我们所要做的就是产生一个新的`worker`。 那么我们来修改`clusteredApp.js`模块来考虑这个问题：

```javascript
if (cluster.isMaster) {
  // ...
  cluster.on('exit', (worker, code) => {
    if (code != 0 && !worker.suicide) {
      console.log('Worker crashed. Starting a new worker');
      cluster.fork();
    }
  });
} else {
  require('./app');
}
```

在前面的代码中，一旦主进程收到`exit`事件，我们检查进程是有意终止的还是错误的结果；我们通过检查状态码和`worker.exitedAfterDisconnect`来实现这一点，这表明工作者是否被明确地终止了。 如果我们确认过程因错误而终止，我们启动一个新的`worker`。有意思的是，当崩溃的`worker`重新启动时，其他`worker`仍然可以提供请求，从而不会影响应用程序的可用性。

为了测试这个假设，我们可以试着用`siege`再次重启我们的服务器。当压力测试完成时，我们注意到`siege`产生的各种指标中还有一个衡量应用程序可用性的指标。 预期的结果会是这样的：

```
Transactions: 3027 hits
Availability: 99.31%
Failed transactions: 21
```

![](http://oczira72b.bkt.clouddn.com/18-2-6/17382904.jpg)

请记住，这个结果可能会有很大的变化。它在很大程度上取决于正在运行的实例的数量以及它们在测试期间崩溃的次数，但是它应该很好地指出我们的解决方案是如何工作的。 前面的数字告诉我们，尽管我们的应用程序不断崩溃，但是在超过了`3027`次请求中只有`21`次失败的请求。 在我们构建的示例场景中，大部分失败的请求将由崩溃期间已建立连接的中断引起。

事实上，当发生这种情况时，`siege`将会打印出如下错误：

```
[error] socket: read error Connection reset by peer sock.c:479: Connection reset by peer
```

不幸的是，为了防止这类类型的错误，我们能够做的不多，特别是当应用程序因崩溃而终止时。尽管如此，我们的解决方案证明是可行的，对于经常崩溃的应用程序，使用`cluster`，其可拓展性性并不差。

#### 零宕机重启
当代码需要更新时，`Node.js`应用程序也可能需要重新启动。因此，在这种情况下，拥有多个实例可以帮助维护我们应用程序的可用性。
当我们不得不故意重新启动一个应用程序来更新它时，会出现一个小窗口，在这个窗口中应用程序将重新启动并且无法为请求提供服务。如果我们正在更新我们的个人博客，这是可以接受的，但对于具有服务水平协议（`SLA`）的专业应用程序就不行了，或者作为持续交付过程的一部分经常更新的专业应用程序。解决方案是实现零宕机重新启动，更新应用程序的代码而不影响其可用性。

使用`cluster`模块，这又是一项非常简单的任务；该模式包括一次重启一个`worker`。这样，剩余的`worker`可以继续操作和维护可用应用程序的服务。

然后，让我们将这个新模块添加到我们的集群服务器；我们所要做的就是添加一些由主进程执行的新代码（看`clusteredApp.js`文件）：

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const cpus = os.cpus().length;
  for (let i = 0; i < cpus; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code) => {
    if (code != 0 && !worker.exitedAfterDisconnect) {
      console.log('Worker crashed. Starting a new worker');
      cluster.fork();
    }
  });
  
  process.on('SIGUSR2', () => {
    console.log('Restarting workers');
    const workers = Object.keys(cluster.workers);
    
    function restartWorker(i) {
      if (i >= workers.length) return;
      const worker = cluster.workers[workers[i]];
      console.log(`Stopping worker: ${worker.process.pid}`);
      worker.disconnect();
      
      worker.on('exit', () => {
        if (!worker.suicide) return;
        const newWorker = cluster.fork();
        newWorker.on('listening', () => {
          restartWorker(i + 1);
        });
      });
    }
    restartWorker(0);
  });
} else {
  require('./app');
}
```

这是前面的代码的工作原理：

1. 一旦接收到`SIGUSR2`信号，则触发`worker`重新启动。
2. 我们定义一个名为`restartWorker()`的迭代器函数。异步迭代`cluster.workers`的每一项。
3. `restartWorker()`函数的第一个任务是通过调用`worker.disconnect()`来优雅地停止工作。
4. 当终止的进程退出时，我们可以产生一个新的`worker`。
5. 只有当新的`worker`准备好并且正在侦听新的连接时，我们才可以通过调用迭代的下一步来重新启动下一个`worker`。

> 由于我们的程序使用了`UNIX`信号，因此在`Windows`系统上无法正常工作（除非您在`Windows 10`中使用最新的`Windows`子系统）。信号是实现我们的解决方案的最简单的机制。但是，这不是唯一的；实际上，其他方法包括侦听来自套接字，管道或标准输入的命令。

现在我们可以通过运行`clusteredApp`模块然后发送一个`SIGUSR2`信号来测试我们的零宕机重启。但是，首先我们需要获取主进程的`PID`；以下命令可用于从所有正在运行的进程的列表中识别它：

```bash
ps af
```

主进程应该是一组节点进程的父节点。一旦我们有我们正在寻找的`PID`，我们可以发送信号给它：

```bash
kill -SIGUSR2 <PID>
```

现在，`clusteredApp`应用程序的输出应该显示如下所示：

```
Restarting workers
Stopping worker: 19389
Started 19407
Stopping worker: 19390
Started 19409
```

我们可以尝试再次使用`siege`来验证我们在重新启动`worker`时对应用程序的可用性没有太大的影响。

> [pm2](https://github.com/Unitech/pm2)是一个基于`cluster`的小型实用程序，它提供负载平衡，过程监控，零宕机重启等功能。

### 处理有状态的通信
`cluster`模块不适用于有状态通信，应用程序维护的状态在各个实例之间不共享。这是因为属于相同有状态会话的不同请求可能会由应用程序的不同实例处理。这不是一个仅限于`cluster`模块的问题，但通常它适用于任何种类的无状态负载均衡算法。例如，考虑下图所描述的情况：

![](http://oczira72b.bkt.clouddn.com/18-2-11/21685002.jpg)

用户John最初发送一个请求到我们的应用程序来验证自己身份，但是操作的结果是在本地注册的（例如在内存中），所以只有接收到认证请求的应用程序实例（实例`A`）知道`John`已成功通过身份验证。当`John`发送一个新的请求时，负载平衡器可能会将它转发给应用程序的另一个实例，实际上它不具有`John`的认证细节，因此拒绝执行该操作。我们刚刚描述的应用程序不能按比例缩放，但幸运的是，我们可以通过两个简单的解决方案来解决问题。

#### 跨多个实例共享状态
要实现在所有实例之间共享状态，我们必须使用有状态通信来扩展应用程序。这可以通过共享数据存储容易地实现，例如像[PostgreSQL](http://www.postgresql.org)，[MongoDB](http://www.mongodb.org)或[CouchDB](http://couchdb.apache.org)，或者甚至更好，我们可以使用内存存储，如[Redis](http://redis.io)或[Memcached](http://memcached.org)。

下图概述了这个简单有效的解决方案：

![](http://oczira72b.bkt.clouddn.com/18-2-11/30499145.jpg)

在通信状态中使用共享存储的唯一缺点是，这并不总是容易实现的，例如，我们可能会使用现有的库在内存中保持通信状态；无论如何，如果我们有一个现有的应用程序，那要在现有应用程序上增加共享数据存储则需要更改应用程序的代码（如果它尚未支持）。正如我们接下来会看到的那样，看接下来这个解决方案。

#### 粘性负载均衡
我们必须支持有状态通信的另一种方法是使负载均衡器始终将与会话相关的所有请求都路由到应用程序的同一实例。这种技术也被称为粘性负载均衡。

下图说明了涉及此技术的简化方案：

![](http://oczira72b.bkt.clouddn.com/18-2-11/8087704.jpg)

从上图可以看出，当负载均衡器接收到与`session`相关的请求时，它会创建一个映射，其中包含由负载平衡算法选择的一个特定实例。负载平衡器下一次接收到来自同一个会话的请求时，会绕过负载平衡算法，选择之前与会话关联的应用程序实例。我们刚刚描述的特定技术涉及检查与请求相关的`session ID`（通常由应用程序或负载平衡器本身包含在`cookie`中）。

将有状态连接关联到单个服务器的更简单的替代方法是记住执行请求的客户端的`IP`地址。通常，将`IP`提供给一个`hash`函数，该函数生成一个代表指定接收请求的应用程序实例的`ID`。这种技术的优点是不需要负载均衡器记住关联。但是，对于频繁更换IP的设备，例如在不同网络上漫游时，它不起作用。

> `cluster`模块默认不支持粘性负载均衡；不过，它可以添加一个名为[sticky-session](https://www.npmjs.org/package/sticky-session)的`npm`库来实现这一点。

粘性负载均衡的一个大问题是，它使得拥有冗余系统的大部分优点失效，其中应用程序的所有实例都是相同的，并且实例可以最终替代另一个停止工作的实例。出于这些原因，建议避免在共享存储中维护任何会话状态使用粘性负载均衡，在根本不需要有状态通信的应用程序（例如，通过在请求中包含状态）使用粘性负载均衡。

> 对于需要粘性负载平衡的库的一个真实例子，可以看看[socket.io](https://socket.io/blog/introducing-socket-io-1-0/#scalability)

### 使用反向代理进行缩放
`cluster`模块不是我们必须扩展`Node.js Web`应用程序的唯一选项。事实上，更多的传统技术往往是首选，因为它们在生产环境中更易于使用。

替代`cluster`的另一种方法是启动在不同端口或计算机上运行的同一应用程序的多个独立实例，然后使用反向代理（或网关）提供对这些实例的访问权限，从而将流量分配到这些实例。在这个配置中，我们没有一个主进程将请求分发给一组工作者，而是在同一台机器上运行的一组不同的进程（使用不同的端口），或者分散在网络内的不同机器上。为了向我们的应用程序提供单一的访问点，我们可以使用一个反向代理，放置在客户端和应用程序的实例之间的一个特殊的设备或服务，它接受任何请求并将其转发到目标服务器，并将结果返回给 客户端，而这些对客户端来说都是透明的。在这种情况下，反向代理也用作负载平衡器，将请求分发到应用程序的实例中。

> 有关反向代理和转发代理之间差异的明确说明，可以参阅[Apache HTTP服务器文档](http://httpd.apache.org/docs/2.4/mod/mod_proxy.html#forwardreverse)

下图显示了一个典型的多进程多机配置，其中一个反向代理充当负载均衡器的前端：

![](http://oczira72b.bkt.clouddn.com/18-2-11/90415875.jpg)

对于Node.js应用程序，选择此方法取代`cluster`模块的原因有很多：

* 反向代理可以将负载分布到多个机器上，而不仅仅是几个进程；
* 市场上最流行的反向代理支持粘性负载均衡；
* 反向代理可以将请求路由到任何可用的服务器，而不管其编程语言或平台；
* 我们可以选择更强大的负载均衡算法；
* 许多反向代理还提供其他服务，例如URL重写，缓存，SSL终止点，甚至可以使用的完全成熟的Web服务器的功能，例如，为静态文件提供服务。

也就是说，如果需要，`cluster`模块也可以很容易地与反向代理结合使用；例如，使用`cluster`在单个机器内部垂直缩放，然后使用反向代理在不同节点之间水平缩放。

> 模式：使用反向代理来平衡在不同端口或机器上运行的多个实例之间的应用程序负载。

对于反向代理实现负载均衡器，我们有很多选择；一些流行的解决方案如下：

* [Nginx](http://nginx.org)：这是一个基于`非阻塞I/O`模型的`Web`服务器，反向代理和负载均衡器。
* [HAProxy](http://www.haproxy.org)：这是一个用于`TCP/HTTP`流量的快速负载均衡器。
* 基于`Node.js`的代理：有很多解决方案可以直接在`Node.js`中实现反向代理和负载均衡器。这可能有优点和缺点，我们将在后面看到。
* 基于云的代理服务器：在云计算时代，利用负载均衡器作为服务并不罕见。这可能很方便，因为它基本不需要维护，通常具有高度的可扩展性，有时它可以支持动态配置以实现按需扩展。

在本章接下来的几节中，我们将分析一个使用`Nginx`的配置示例，接下来我们还将使用`Node.js`来构建我们自己的负载均衡器。

#### 使用`Nginx`进行负载平衡
为了说明专用反向代理如何工作，我们现在将构建基于[Nginx](http://nginx.org)的可扩展架构，但首先我们需要安装它。 我们可以按照 http://nginx.org/en/docs/install.html 上的说明来做到这一点。

在最新的`Ubuntu`系统上，您可以使用以下命令快速安装`Nginx`：

```bash
sudo apt-get install nginx
```

在`Mac OSX`上，您可以使用[brew](http://brew.sh)：

```bash
brew install nginx
```

由于我们不打算使用`cluster`来启动服务器的多个实例，因此我们需要稍微修改应用程序的代码，以便我们可以使用命令行参数指定侦听端口。这将允许我们在不同的端口上启动多个实例。我们再来考虑我们的示例应用程序（`app.js`）的主要模块：

```javascript
const http = require('http');
const pid = process.pid;

http.createServer((req, res) => {
  for (let i = 1e7; i > 0; i--) {}
  console.log(`Handling request from ${pid}`);
  res.end(`Hello from ${pid}\n`);
}).listen(process.env.PORT || process.argv[2] || 8080, () => {
  console.log(`Started ${pid}`);
});
```

另一个不使用`cluster`的原因是其在发生崩溃时无法自动重启。幸运的是，这很容易通过使用专用的管理程序来解决，该管理程序监视我们的应用程序并在必要时重新启动的外部进程。可能的选择如下：

* 基于`Node.js`的`supervisors`，如[forever](https://npmjs.org/package/forever)或[pm2](https://npmjs.org/package/pm2)
* 基于`OS`的`supervisors`，例如[upstart](http://upstart.ubuntu.com)，[systemd](http://freedesktop.org/wiki/Software/systemd)或者[runit](http://smarden.org/runit/)
* 更高级的`supervisors`解决方案，如[monit](http://mmonit.com/monit)或[supervisor](http://supervisord.org)。

对于这个例子，我们将使用`forever`，这是我们使用最简单，最直接的。 我们可以通过运行以下命令来全局安装它：

```bash
npm install forever -g
```

下一步是启动我们的应用程序的四个实例，全部在不同的端口上，使用`forever`：

```bash
forever start app.js 8081
forever start app.js 8082
forever start app.js 8083
forever start app.js 8084
```

我们可以使用以下命令检查已启动进程的列表：

```bash
forever list
```

现在需要将`Nginx`服务器配置为负载平衡器。

首先，我们需要根据你的系统来确定`nginx.conf`文件的位置。一般是在`/usr/local/nginx/conf`，`/etc/nginx`，或者`/usr/local/etc/nginx`。

接下来，我们打开`nginx.conf`文件并应用以下配置，这是获得实现负载均衡所需的最基础的配置：

```
http {
  # [...]
  
  upstream nodejs_design_patterns_app {
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
    server 127.0.0.1:8083;
    server 127.0.0.1:8084;
  }

  # [...]
  
  server {
      listen       80;

      location / {
        proxy_pass         http://nodejs_design_patterns_app;
      }
  }
  
  # [...]
}
```

对于配置文件，基本不用解释。在`upstream nodejs_design_patterns_app`部分，我们定义了用于处理网络请求的后端服务器列表，然后在`server`部分中指定了`proxy_pass`指令，这本质上告诉`Nginx`将任何请求转发给我们之前定义的服务器组（`nodejs_design_patterns_app`）。就是这样，现在我们只需要用以下命令重新加载`Nginx`配置：

```bash
nginx -s reload
```

我们的系统现在应该已经启动并且正在运行，已经准备好接受请求并且平衡`Node.js`应用程序的四个实例的流量。 只需在您的浏览器打开地址http://localhost，查看我们的`Nginx`服务器如何平衡流量。

### 使用服务注册表
现代基于云的基础架构的一个重要优势是能够基于当前的运行情况，预测的流量动态调整应用的容量；这也被称为动态缩放。如果实施得当，这种做法可以极大地降低`IT`基础架构的成本，同时保持应用程序的高可用性和响应能力。

这个想法很简单：如果我们的应用程序正在经历由流量高峰造成的性能下降，我们会自动产生新的服务器来应对增加的负载。我们也可以决定在某些时间关闭一些服务器，例如晚上，当我们知道流量将会减少时，在早上再次重新启动它们。该机制要求负载均衡器随时了解当前的网络拓扑结构，随时了解哪台服务器处于运行状态。

解决此问题的常见模式是使用称为服务注册中心的中央存储库，该中心存储库跟踪正在运行的服务器及其提供的服务。下图显示了前端具有负载平衡器的多服务架构，使用服务注册表进行动态配置：

![](http://oczira72b.bkt.clouddn.com/18-2-11/86308058.jpg)

上述架构假定存在两个服务`API`和`WebApp`。负载均衡器将到达`/ api`节点的请求分发给实现`API`服务的所有服务器，而其余请求分布在实现WebApp服务的服务器上。负载均衡器获取使用服务注册表的服务器列表。

为了使其完全自动化运行，每个应用程序实例在联机时必须自己注册到服务注册表，并在其停止时取消注册。通过这种方式，负载均衡器可以始终拥有最新的服务器视图和网络上可用的服务。

> 模式（服务注册表）：使用中央资源库来存储和管理服务器的最新视图以及系统中可用的服务。

这种模式不仅可以应用于负载平衡，还可以更普遍地作为从提供服务的服务器分离服务类型的一种方式。我们可以将其视为适用于网络服务的服务定位器的设计模式。

#### 使用`http-proxy`和`Consul`实现动态负载均衡器
为了实现粘性负载均衡，我们可以使用反向代理，例如`Nginx`或`HAProxy`；我们所需要做的就是使用自动服务更新其配置，然后强制负载均衡器选择更改。 对于`Nginx`，可以使用以下命令行完成：

```bash
nginx -s reload
```

使用基于云的解决方案可以获得相同的结果，但我们有第三种更熟悉的替代方案，可以使用我们最喜欢的平台。

我们都知道`Node.js`是构建任何网络应用程序的好工具；正如我们所说，这正是其主要设计目标之一。那么，为什么不建立一个只使用`Node.js`的负载均衡器呢？ 这将给我们更多的自由，并允许我们直接在我们的定制负载平衡器中实现任何类型的模式或算法，包括我们现在要探索的负载平衡器，使用服务注册表的动态负载平衡。在这个例子中，我们将使用[Consul](https://www.consul.io)作为服务注册表。

在这个例子中，我们想要复制我们在上一节中看到的多服务体系结构，为此，我们将主要使用三个`npm`包：

* [http-proxy](https://npmjs.org/package/http-proxy)：这是一个库，用于简化`Node.js`中代理和负载均衡器的创建
* [portfinder](https://npmjs.com/package/portfinder)：这是一个允许发现系统中的自由端口的库
* [consul](https://npmjs.org/package/consul)：这是一个图书馆，允许服务在`consul`登记

让我们开始实施我们的服务。 它们是简单的`HTTP`服务器，就像我们迄今用来测试`cluster`和`Nginx`的`HTTP`服务器一样，但是这次我们希望每个服务器都在服务注册表启动的时候注册自己。

让我们看看这看起来如何（文件`app.js`）：

```javascript
const http = require('http');
const pid = process.pid;
const consul = require('consul')();
const portfinder = require('portfinder');
const serviceType = process.argv[2];

portfinder.getPort((err, port) => {
  const serviceId = serviceType+port;
  consul.agent.service.register({
    id: serviceId,
    name: serviceType,
    address: 'localhost',
    port: port,
    tags: [serviceType]
  }, () => {

    const unregisterService = (err) => {
      consul.agent.service.deregister(serviceId, () => {
        process.exit(err ? 1 : 0);
      });
    };

    process.on('exit', unregisterService);
    process.on('SIGINT', unregisterService);
    process.on('uncaughtException', unregisterService);

    http.createServer((req, res) => {
      for (let i = 1e7; i > 0; i--) {}
      console.log(`Handling request from ${pid}`);
      res.end(`${serviceType} response from ${pid}\n`);
    }).listen(port, () => {
      console.log(`Started ${serviceType} (${pid}) on port ${port}`);
    });
  });
});
```

在前面的代码中，有一些部分值得我们关注：

* 首先，我们使用`portfinder.getPort`来发现系统中的一个空闲端口（默认情况下，`portfinder`从`8000`端口开始搜索）。
* 接下来，我们使用`Consul`库在注册表中注册一项新服务。服务定义需要几个属性：`id`（服务的唯一名称），`name`（标识服务的通用名称），`address`和`port`（用于标识如何访问服务），`tags`（可选的标签数组用于过滤和分组服务）。我们使用`serviceType`（我们将其作为命令行参数）来指定服务名称并添加标签。这将允许我们识别集群中可用的相同类型的所有服务。
* 此时我们定义了一个名为`unregisterService`的函数，它允许我们在集群中定义相同类型的服务。
* 我们使用`unregisterService`作为清理函数，以便程序运行时关闭（无论是人为关闭还是意外关闭），从取消注册。
* 最后，我们为`portfinder`发现的端口上的服务启动`HTTP`服务器。

现在是实施负载均衡器的时候了。 我们通过创建一个名为`loadBalancer.js`的新模块来实现这一点。首先，我们需要定义一个路由表来将`URL`路径映射到服务：

```javascript
const routing = [{
  path: '/api',
  service: 'api-service',
  index: 0
}, {
  path: '/',
  service: 'webapp-service',
  index: 0
}];
```

`routing`数组中的每个项目都包含用于处理到达映射路径的请求的服务。`index`属性将用于循环给定服务的请求。

让我们通过实现`loadbalancer.js`的第二部分来看看它是如何工作的：

```javascript
const proxy = httpProxy.createProxyServer({});
http.createServer((req, res) => {
  let route;
  routing.some(entry => {
    route = entry;
    //Starts with the route path?
    return req.url.indexOf(route.path) === 0;
  });

  consul.agent.service.list((err, services) => {
    const servers = [];
    Object.keys(services).filter(id => {
      if (services[id].Tags.indexOf(route.service) > -1) {
        servers.push(`http://${services[id].Address}:${services[id].Port}`)
      }
    });

    if (!servers.length) {
      res.writeHead(502);
      return res.end('Bad gateway');
    }

    route.index = (route.index + 1) % servers.length;
    proxy.web(req, res, {target: servers[route.index]});
  });
}).listen(8080, () => console.log('Load balancer started on port 8080'));

```

这就是我们如何实现基于`Node.js`的负载均衡器：

1. 首先，我们需要`consul`，以便我们可以访问注册表。接下来，我们实例化一个`http-proxy`对象并启动一个普通的`web`服务器。
2. 在服务器的请求处理程序中，我们所做的第一件事是将`URL`与我们的路由表进行匹配。 结果将是一个包含服务名称的描述符。
3. 我们从`consul`获得实施所需服务的服务器清单。如果这个列表是空的，我们会向客户端返回一个错误。我们使用Tag属性来过滤所有可用的服务，并查找实现当前服务类型的服务器的地址。最后，我们可以将请求路由到它的目的地。 我们根据循环法更新route.index以指向列表中的下一个服务器。然后，我们使用索引从列表中选择一个服务器，并将它与请求（`req`）和响应（`res`）对象一起传递给`proxy.web()`。 这将简单地将请求转发到我们选择的服务器。

现在很清楚，仅使用`Node.js`和服务注册表来实现负载均衡器是多么简单，以及我们可以通过这种方式实现多大的灵活性。现在，我们应该准备好了，但首先，请通过以下官方文档安装`Consul`服务器： https://www.consul.io/intro/getting-started/install.html 。

这使我们能够通过这个简单的命令行在我们的开发机器中启动`consul`服务注册表：

```bash
consul agent -dev
```

现在我们准备启动负载平衡器：

```bash
node loadBalancer
```

现在，如果我们尝试访问负载平衡器公开的某些服务，我们会注意到它返回一个`HTTP 502`错误，因为我们还没有启动任何服务器。亲自尝试一下：

```bash
curl localhost:8080/api
```

上述命令应返回以下输出：

```
Bad Gateway
```

如果我们产生一些服务实例，情况将会发生变化，例如，两个`api-service`和一个`webapp-service`：

```
forever start app.js api-service
forever start app.js api-service
forever start app.js webapp-service
```

现在负载平衡器应该自动查看新服务器并开始在它们之间分配请求。 让我们尝试使用以下命令：

```bash
curl localhost:8080/api
```

上述命令现在应该返回：

```bash
api-service response from 6972
```

通过再次运行它，我们现在应该从另一台服务器收到一条消息，确认请求正在不同服务器之间负载均衡：

```bash
api-service response from 6979
```

![](http://oczira72b.bkt.clouddn.com/18-2-25/55815960.jpg)

这种模式的优点是显而易见的。我们现在可以动态，按需或基于时间表调整我们的基础架构，我们的负载均衡器将自动根据新配置进行调整，无需任何额外的工作！

## 点对点负载平衡
当我们想要将一个复杂的内部网络架构暴露给公共网络（如`Internet`）时，使用反向代理几乎是必需的。它有助于隐藏复杂性，提供外部应用程序可轻松使用和依赖的单一访问点。但是，如果我们需要扩展仅供内部使用的服务，则我们可以拥有更多的灵活性和控制力。

假设有一个服务A依靠服务B来实现其功能。服务B在多台机器上进行缩放，并且只能在内部网络中使用。我们迄今为止所了解到的是，服务A将使用反向代理连接到服务B，反向代理会将流量分发到实施服务B的所有服务器。

但是，还有一个选择。我们可以从图中删除反向代理，并直接从客户端（服务A）分发请求，该客户端现在直接负责跨服务B的各种实例负载平衡其连接。只有服务器A知道详细信息关于暴露服务B的服务器，并且在内部网络中，这通常是已知信息。通过这种方法，我们基本上实现了对等负载均衡。

下图比较了我们刚刚描述的两种替代方案：

![](http://oczira72b.bkt.clouddn.com/18-2-25/98621619.jpg)

这是一种非常简单而有效的模式，可以实现真正的分布式通信，而不会出现瓶颈或单点故障。除此之外，它还执行以下操作：

* 通过删除网络节点来降低基础设施的复杂性
* 更快的通信，因为消息将通过更少的节点
* 规模更好，因为性能不受负载平衡器可以处理的限制

另一方面，通过删除反向代理，我们实际上暴露了其底层基础架构的复杂性。此外，通过实施负载平衡算法，每个客户端都必须变得更加智能，并且可能也是保持其基础架构最新的一种方式。

> 点对点负载均衡是[ØMQ](http://zeromq.org)库中广泛使用的一种模式。

### 实现可以跨多台服务器平衡请求的HTTP客户端
