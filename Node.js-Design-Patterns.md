# Node.js-Design-Patterns
## Welcome to the Node.js Platform
### 最新ES语法
### reactor模式
`reactor模式`是`Node.js`异步编程的核心模块，其核心概念是：`单线程`、`非阻塞I/O`。

* `非阻塞I/O`：在这种机制下，后续代码块不会等到`I/O`请求数据的返回之后再执行。如果当前时刻所有数据都不可用，函数会先返回预先定义的常量值(如`undefined`)，表明当前时刻暂无数据可用。例如，在`Unix`操作系统中，`fcntl()`函数操作一个已存在的文件描述符，改变其操作模式为`非阻塞I/O`(通过`O_NONBLOCK`状态字)。一旦资源是非阻塞模式，如果读取文件操作没有可读取的数据,或者如果写文件操作被阻塞,读操作或写操作返回`-1`和`EAGAIN`错误。`非阻塞I/O`最基本的模式是通过轮询获取数据，这也叫做**忙-等模型**。看下面这个例子，通过`非阻塞I/O`和轮询机制获取`I/O`的结果。

### 事件多路复用
```javascript
let socketA, pipeB;
wachedList.add(socketA, FOR_READ);
wachedList.add(pipeB, FOR_READ);
while(events = demultiplexer.watch(wachedList)) {
  // 事件循环
  foreach(event in events) {
    // 这里并不会阻塞，并且总会有返回值（不管是不是确切的值）
    data = event.resource.read();
    if (data === RESOURCE_CLOSED) {
      // 资源已经被释放，从观察者队列移除
      demultiplexer.unwatch(event.resource);
    } else {
      // 成功拿到资源，放入缓冲池
      consumeData(data);
    }
  }
}
```

## Node.js Essential Patterns
### 回调模式
回调模式分为异步CPS风格和同步CPS风格、原生JS也可以实现回调模式
```javascript
function add(a, b) {
  return a + b;
}
```

改为CPS风格：

```javascript
function add(a, b, callback) {
  callback(a + b);
}
```

异步CPS：

```javascript
function additionAsync(a, b, callback) {
 setTimeout(() => callback(a + b), 100);
}
```

### Zalgo
解决方案：

* 使用同步API
* 延时处理

### Node.js的惯用风格
* 错误处理总在最前
* 错误传播
* 某些异常不太好捕获

### 模块系统及其模式
* 模块缓存原理

```javascript
const require = (moduleName) => {
  console.log(`Require invoked for module: ${moduleName}`);
  const id = require.resolve(moduleName);
  // 是否命中缓存
  if (require.cache[id]) {
    return require.cache[id].exports;
  }
  // 定义module
  const module = {
    exports: {},
    id: id
  };
  // 新模块引入，存入缓存
  require.cache[id] = module;
  // 加载模块
  loadModule(id, module, require);
  // 返回导出的变量
  return module.exports;
};
require.cache = {};
require.resolve = (moduleName) => {
  /* 通过模块名作为参数resolve一个完整的模块 */
};
```

* 模块循环依赖
* 模块寻找的算法

### 观察者模式
* EventEmitter类
如何让任意对象可观察，拓展EventEmitter类：

```javascript
const EventEmitter = require('events').EventEmitter;
const fs = require('fs');
class FindPattern extends EventEmitter {
  constructor(regex) {
    super();
    this.regex = regex;
    this.files = [];
  }
  addFile(file) {
    this.files.push(file);
    return this;
  }
  find() {
    this.files.forEach(file => {
      fs.readFile(file, 'utf8', (err, content) => {
        if (err) {
          return this.emit('error', err);
        }
        this.emit('fileread', file);
        let match = null;
        if (match = content.match(this.regex)) {
          match.forEach(elem => this.emit('found', file, elem));
        }
      });
    });
    return this;
  }
}
```

## Asynchronous Control Flow Patterns with Callbacks
如何写更优雅的回调：

* 避免回调地狱
* 迭代模式

```javascript
function iterate(index) {
  if (index === tasks.length) {
    return finish();
  }
  const task = tasks[index];
  task(function() {
    iterate(index + 1);
  });
}

function finish() {
  // 迭代完成的操作
}

iterate(0);
```


* 并发处理

## Asynchronous Control Flow Patterns with ES2015 and Beyond
这一章主要讲的是如何用`Promise`、`Generator`，以及`async await`简化异步。

几种方式各有优劣：

![](http://oczira72b.bkt.clouddn.com/17-11-7/55293366.jpg)

## Coding with Streams
### Streams和Buffer
* 空间效率更高
* 时间效率更高

### 实现可读的Streams

```javascript
const stream = require('stream');
const Chance = require('chance');

const chance = new Chance();

class RandomStream extends stream.Readable {
  constructor(options) {
    super(options);
  }

  _read(size) {
    const chunk = chance.string(); //[1]
    console.log(`Pushing chunk of size: ${chunk.length}`);
    this.push(chunk, 'utf8'); //[2]
    if (chance.bool({
        likelihood: 5
      })) { //[3]
      this.push(null);
    }
  }
}

module.exports = RandomStream;
```

### 实现可写的Streams

```javascript
const stream = require('stream');
const fs = require('fs');
const path = require('path');
const mkdirp = require('mkdirp');

class ToFileStream extends stream.Writable {
  constructor() {
    super({
      objectMode: true
    });
  }

  _write(chunk, encoding, callback) {
    mkdirp(path.dirname(chunk.path), err => {
      if (err) {
        return callback(err);
      }
      fs.writeFile(chunk.path, chunk.content, callback);
    });
  }
}
module.exports = ToFileStream;
```

### 双重的Streams

```javascript
const stream = require('stream');
const util = require('util');

class ReplaceStream extends stream.Transform {
  constructor(searchString, replaceString) {
    super();
    this.searchString = searchString;
    this.replaceString = replaceString;
    this.tailPiece = '';
  }

  _transform(chunk, encoding, callback) {
    const pieces = (this.tailPiece + chunk)         //[1]
      .split(this.searchString);
    const lastPiece = pieces[pieces.length - 1];
    const tailPieceLen = this.searchString.length - 1;

    this.tailPiece = lastPiece.slice(-tailPieceLen);     //[2]
    pieces[pieces.length - 1] = lastPiece.slice(0,-tailPieceLen);

    this.push(pieces.join(this.replaceString));       //[3]
    callback();
  }

  _flush(callback) {
    this.push(this.tailPiece);
    callback();
  }
}

module.exports = ReplaceStream;
```

### 异步
Streams在异步编程有很广泛的运用，实现一个无序并行的Streams：

```javascript
const stream = require('stream');

class ParallelStream extends stream.Transform {
  constructor(userTransform) {
    super({objectMode: true});
    this.userTransform = userTransform;
    this.running = 0;
    this.terminateCallback = null;
  }

  _transform(chunk, enc, done) {
    this.running++;
    this.userTransform(chunk, enc, this._onComplete.bind(this), this.push.bind(this));
    done();
  }

  _flush(done) {
    if(this.running > 0) {
      this.terminateCallback = done;
    } else {
      done();
    }
  }

  _onComplete(err) {
    this.running--;
    if(err) {
      return this.emit('error', err);
    }
    if(this.running === 0) {
      this.terminateCallback && this.terminateCallback();
    }
  }
}

module.exports = ParallelStream;
```

### 实现组合的Streams

使用诸如`multipipe`之类的库，我们可以通过组合一些核心库中已有的`Streams`（文件`combinedStreams.js`）来轻松地构建组合的`Streams`：

```javascript
const zlib = require('zlib');
const crypto = require('crypto');
const combine = require('multipipe');
module.exports.compressAndEncrypt = password => {
  return combine(
    zlib.createGzip(),
    crypto.createCipher('aes192', password)
  );
};
module.exports.decryptAndDecompress = password => {
  return combine(
    crypto.createDecipher('aes192', password),
    zlib.createGunzip()
  );
};
```

## Design Patterns
* 工厂模式（`Factory`）：通过stampit可以实现组合的工厂函数，在Node.js中这种模式有广泛应用，例如Node.js的核心库http，也是提供了工厂创建实例的方式。

* 揭示构造模式（`Revealing constructor`）：揭示构造函数模式接受执行函数executor作为参数，这个函数被提供给构造函数，并在内部调用。这种模式也在Node.js有广泛应用。最为显著的是原生Promise使用了这一种模式。也可以通过揭示构造函数模式创建只读的event emitter，可以较好地保证函数内部安全性。

* 代理模式（`Proxy`）、装饰者模式（`Decorator`）都常常使用对象增强和对象组合的方式书写。其各有优缺点，对象增强会改变主体对象，对象组合的写法又比较繁琐。他们都有广泛的应用，例如著名的Mongoose大量使用代理模式。AOP编程方式也是代理模式的应用。hooks这个库则是AOP的完美体现。装饰者模式的应用也很多，如levelup的许多插件则使用了装饰者模式。而由于JavaScript语言的动态性，实现装饰者模式则比较简单。

* 适配器模式（`Adapter`）：允许我们用不同的接口去访问对象的功能，它适配一个对象，以便于它可以被不同接口调用。适配器模式也有所应用场景，例如我们可以对核心库做上层封装，并且适配对应的核心库的功能，书上的fsAdapter则是这个模式的较好的体现。

* 策略模式（`Strategy`）、状态模式（`State`）和模板模式（`Template`）：使用模式来简化大量的条件代码的书写，状态模式类似于策略模式，状态模式Context的策略会根据State的变化而变化。而模板模式其实就是C++的类模板在JavaScript的体现。由于JavaScript语言本身没有类模板这样的功能。通过在一个类的方法中抛出异常来实现一个抽象类和虚函数。
* 中间件模式（`Middleware`）：思想源于拦截过滤器模式和责任链模式。常见的Web框架Express和Koa都广泛使用中间件模式。
* 命令模式（`Command`）：降低了对象之间的耦合度，设计命令也相对简单，代码解耦，但是使用命令模式可能导致系统命令类过多，这是命令模式的一大缺陷。

## Writing Modules
### 如何去定义一个模块
* 一个模块应该具有可读性和可理解性，因为它应该专注于一件事
* 一个模块被表示为一个单独的文件，使得其更容易被识别
* 模块可以更容易地在不同的应用程序中复用

### 依赖注入
依赖注入（DI）模式可能是软件设计中最容易被误解的概念之一。许多人将这个术语与框架和依赖注入容器相关联，例如`Spring`（用于`Java`和`C#`）或`Pimple`（用于`PHP`），但实际上它是一个很简单的概念。依赖注入模式背后的主要思想是由外部实体提供输入的组件的依赖关系。

这样的实体可以是客户端组件或全局容器，它集中了系统所有模块的关联。这种方法的主要优点是解耦，特别是对于取决于有状态实例的模块。使用DI，从外部接收每个依赖项，而不是硬编码到模块中。这意味着模块可以配置为其中的依赖关系，因此可以在不同的上下文中重用。

### 服务定位器
服务定位器核心原则是拥有一个中央注册中心，以便管理系统组件，并在模块需要加载依赖时作为中介。这个想法是要求服务定位器所连接的是依赖注入模块，而不是硬编码模块。通过使用服务定位器，我们引入了对它的依赖关系，它连接到模块的方式决定了它们的耦合程度，其可重用性较高。 在`Node.js`中，我们可以确定三种类型的服务定位器，区分它们的关键因素是它们连接到系统各个组件的方式：分为硬编码依赖服务定位器、依赖注入服务定位器和全局注入服务定位器。

服务定位器的基本模式：

```javascript
"use strict";

module.exports = () => {
  const dependencies = {};
  const factories = {};
  const serviceLocator = {};
  
  serviceLocator.factory = (name, factory) => {
    factories[name] = factory;
  };
  
  serviceLocator.register = (name, instance) => {
    dependencies[name] = instance;
  };
  
  serviceLocator.get = (name) => {
    if (!dependencies[name]) {
      const factory = factories[name];
      dependencies[name] = factory && factory(serviceLocator);
      if (!dependencies[name]) {
        throw new Error('Cannot find module: ' + name);
      }
    }
    return dependencies[name];
  };

  return serviceLocator;
};
```

## Advanced Asynchronous Recipes
这一章主要讲了三种书写异步的常见模型：

* 异步引入模块并初始化
* 在高并发的应用程序中使用批处理和缓存异步操作的性能优化
* 运行与`Node.js`处理并发请求的能力相悖的阻塞事件循环的同步`CPU`绑定操作

### 异步初始化模块
如果一个Node.js模块需要异步初始化，可以有以下两种解决方案：一是在开始使用模块之前之前确保模块已经初始化，否则则等待其初始化。二是使用预初始化队列进行初始化，在初始化之前的所有操作均放入预初始化队列中，等待初始化完成后取出队列中的任务。

* 等待初始化

```javascript
// 模块app.js
const db = require('aDb'); // aDb是一个异步模块
const findAllFactory = require('./findAll');
db.on('connected', function() {
  const findAll = findAllFactory(db);
  // 之后再执行异步操作
});


// 模块findAll.js
module.exports = db => {
  //db 在这里被初始化
  return function findAll(type, callback) {
    db.findAll(type, callback);
  }
}
```

* 预初始化队列

```javascript
const asyncModule = module.exports;

asyncModule.initialized = false;
asyncModule.initialize = callback => {
  setTimeout(() => {
    asyncModule.initialized = true;
    callback();
  }, 10000);
};

asyncModule.tellMeSomething = callback => {
  process.nextTick(() => {
    if(!asyncModule.initialized) {
      return callback(
        new Error('I don\'t have anything to say right now')
      );
    }
    callback(null, 'Current time is: ' + new Date());
  });
};
```

### 批处理和缓存
对于接口请求量较大的API，我们可以使用批处理或者缓存来提升接口性能：批处理指的是在调用异步函数的同时在队列中还有另一个尚未处理的回调，我们可以将回调附加到已经运行的操作上，而不是创建一个全新的请求。缓存不只可以是内存中的变量，还可以是数据库，甚至是专门的缓存服务器。此外，使用缓存需要有一定的策略对缓存进行淘汰，例如LRU。

### CPU-bound任务
对于运算量较大的同步的CPU-bound任务，可能造成接口的阻塞。解决方案有两种：一种是使用异步来包装同步的CPU-bound任务，二是使用多进程，一般来说使用Node.js子进程，因为Node.js自带子进程模块，并且可以使用相关接口进行父子进程的管道通信，而如果子进程不是Node.js进程，一般只能通过标准输入输入来进行父子进程的通信。

如何父子进程进行通信：

```javascript
const EventEmitter = require('events').EventEmitter;
const ProcessPool = require('./processPool');
const workers = new ProcessPool(__dirname + '/subsetSumWorker.js', 2);

class SubsetSumFork extends EventEmitter {
  constructor(sum, set) {
    super();
    this.sum = sum;
    this.set = set;
  }

  start() {
    workers.acquire((err, worker) => {  // [1]
      worker.send({sum: this.sum, set: this.set});

      const onMessage = msg => {
        if (msg.event === 'end') {  // [3]
          worker.removeListener('message', onMessage);
          workers.release(worker);
        }

        this.emit(msg.event, msg.data);  // [4]
      };

      worker.on('message', onMessage);  // [2]
    });
  }
}

module.exports = SubsetSumFork;
```

## Scalability and Architectural Patterns
### `cluster`模块
`cluster`模块主进程负责产生大量进程（`worker`），每个进程代表我们想要扩展的应用程序的一个实例。每个传入连接然后分布在克隆的`worker`，分散在他们的负载。

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

### 零宕机重启
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

### 粘性负载均衡
* 反向代理
* nginx的负载均衡

# Messaging and Integration Patterns
常见三类消息传递方式：

* 发布/订阅模式
* 管道和任务分配模式
* 请求/回复模式

三类模式都可以点对点通信或者是使用代理进行通信。

