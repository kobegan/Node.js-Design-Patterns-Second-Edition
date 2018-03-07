# Messaging and Integration Patterns
如果应用程序涉及到分布式系统。在前一章中，我们学习了如何如何通过使用一些简单的架构模式来集成大量的服务，将其分割到多个机器上。为了使其正常工作，所有机器都必须以某种方式进行交互，因此必须整合它们的交互方式。

有两种主要的技术来集成分布式应用程序：一种是使用共享存储，另一种是使用消息在系统节点上传播数据，这里涉及事件和命令模式。后者在扩展分布式系统时确实有用，这也是后一种方式被广泛运用的原因。

消息被用于软件系统的每一层。我们交换消息以在互联网上进行通信，我们可以使用消息将信息发送到使用管道的其他进程，我们可以使用应用程序中的消息作为直接函数调用（命令模式）的替代方法，甚至也可以使用消息与硬件直接交互。用作在组件和系统之间交换信息的方式的任何离散和结构化数据都可以看作是一条消息。但是，在处理分布式体系结构时，消息传递系统用于描述旨在促进网络信息交换的特定类别的解决方案，模式或者说体系结构。

正如我们将看到的，有几种特征表征这些类型的系统。我们可以选择使用代理模式或点对点结构，我们可以使用请求/回复模式或单向通信，也可以使用队列来更可靠地传递消息；消息整合模式的使用范围非常广泛。本章从`Node.js`及其生态系统的角度探讨了这些众所周知的模式中最重要的模式。

总而言之，在本章中，我们将学习以下主题：

* 消息传递系统的基本原理
* 发布/订阅模式
* 管道和任务分配模式
* 请求/回复模式

## 消息传递系统的基本原理
在谈论消息和消息传递系统时，需要考虑四个基本要素，如下：
* 通信的方向，可以是单向的，也可以是双向的
* 消息的目的地，这也决定了消息的内容
* 消息的时间，这决定了消息是否可以被立即发送和接收（同步），也可以在将来接收（异步）
* 信息的传递方式，直接传递或通过一个中介者进行传递

在接下来的部分中，我们将把这些方面正式化，以便为我们稍后的讨论奠定基础。

### 单向通信和请求/回复模式
消息传递系统中最基本的方面是通信的传递方向，这个方向通常也表示了这条消息的含义。

最简单的消息传递模式是消息从源到目的地单向推送; 这是一个简单的情况，并不需要太多解释：

![](http://oczira72b.bkt.clouddn.com/18-3-6/85270514.jpg)

单向通信的一个典型例子是使用`WebSockets`向连接的浏览器或`Web`服务器发送消息的电子邮件，或将任务分配给一组工作人员的系统。

然而，请求/回复模式比单向通信更受欢迎；一个典型的例子就是调用Web服务。下图显示了这个简单且众所周知的场景：

![](http://oczira72b.bkt.clouddn.com/18-3-6/77969595.jpg)

请求/回复模式可能看起来是一个简单的模式; 但是，当通信异步或涉及多个节点时，我们将看到它变得更加复杂。看看下图中的例子：

![](http://oczira72b.bkt.clouddn.com/18-3-6/78681245.jpg)

通过上图所示的设置，我们可以理解一些请求/回复模式的复杂性。如果我们考虑任何两个节点之间的通信方向，我们可以肯定地说它是单向的。但是，从全局角度来看，发起者发送一个请求，然后接收一个关联的响应，即使来自不同的节点。在这些情况下，真正区分请求/响应模式与单向消息传递模式的区别在于请求和响应之间的关系，它保存在发起者中。回复通常在请求的相同上下文中处理。

### 消息类型
一条消息本质上是连接不同软件组件的一种方式，这样做的原因有很多：这可能是因为我们想要获得由另一个系统或组件持有的某些信息，或远程执行某项操作，或向某个组件通知某操作刚刚发生。消息内容也会因通信原因而异。 一般来说，我们可以根据消息的目的来确定三种类型的消息：

* 命令消息
* 事件消息
* 文档消息

#### 命令消息
命令消息对我们来说已经很熟悉；它本质上是一个序列化的`command`对象，正如我们在`Chapter 6-Design Patterns`中所描述的那样。 这种类型的消息的目的是触发`recevier`上的动作或任务的执行。为了做到这一点，我们的信息必须包含运行任务的基本信息，这通常是操作的名称和执行时提供的参数列表。 命令消息可用于实现远程过程调用（`RPC`）系统，分布式计算或更简单地用于请求某些数据。`RESTful HTTP`调用是命令消息的简单示例; 每个`HTTP`请求都有一个特定的含义，并与一个精确的操作相关联：例如`GET`表示检索资源；`POST`表示创建一个新的资源；`PUT`表示更新一个资源；`DELETE`表示删除一个资源。

#### 事件消息
事件消息用于通知另一个组件发生了某些事件。它通常包含事件的类型，有时还包含一些细节，如`context`，`subject`或`actor`。 在`Web`开发中，当使用长轮询或`WebSocket`接收来自服务器的刚刚发生的事件的通知时，我们在浏览器中使用事件消息，例如数据的变化导致一个时间的发生。事件的使用是分布式应用程序中非常重要的机制，因为它使我们能够将系统的所有节点保持在同一状态上。

#### 文档消息
文档消息主要用于在组件和机器之间传输数据。区分文档消息和命令消息（可能还包含数据）的主要特点是该消息不包含告诉接收方如何处理数据的任何信息。另一方面，与事件消息的主要区别主要是缺少与特定事件的关联。通常，对命令消息的回复是文档消息，因为它们通常只包含请求的数据或操作的结果。

### 异步消息传递和队列
作为`Node.js`开发人员，我们应该已经知道执行异步操作的优势。对于消息和通信而言，这是一回事。

我们可以将同步通信与电话进行比较：两个对等设备必须同时连接到同一个通道，并且它们应该实时交换消息。通常情况下，如果我们想打电话给其他人，我们可能需要另一部手机或关闭正在进行的通信以便开始新的通话。

异步通信类似于`SMS`：它不要求收件人在我们发送邮件时连接到网络，我们可能会立即收到回复或者收到未知延迟后的回复，或者我们可能根本没有收到回复。我们可能会将多个`SMS`一个接一个地发送给多个收件人，并以任何顺序收到他们的回复（如果有）。简而言之，我们使用更少的资源可以获得更好的并行性。

异步通信的另一个重要优点是可以将消息存储并尽快或稍后发送。当接收器太忙而无法处理新消息或我们希望保证传送时，这可能很有用。在消息传递系统中，这可以使用消息队列实现，该消息队列调解发送者和接收者之间的通信，在将消息传递到其目标之前存储任何消息，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-3-6/54869165.jpg)

如果出于任何原因接收机崩溃，与网络断开连接或速度变慢，则消息会在队列中累积并在接收机联机并且完全正常工作时才可以让发送者继续请求并调度。队列可以位于发送者中，也可以在发送者和接收者之间分开，或者存储在充当通信中间件的专用外部系统中。

### 点对点或基于代理的消息传递
消息可以以对等方式直接传送给接收方，也可以通过称为消息代理的集中式中介系统传送。代理的主要作用是将发件人的信息接收者分离出来。下图显示了两种方法之间的架构差异：

![](http://oczira72b.bkt.clouddn.com/18-3-6/78511743.jpg)

在对等体系结构中，每个节点都直接负责将消息传递给接收方。这意味着节点必须知道接收方的地址和端口，他们必须就协议和消息格式达成一致。代理从等式中消除了这些复杂性：每个节点都可以完全独立，并且可以与未定义数量的对等进行通信，而无需直接了解其详细信息。 代理还可以充当不同通信协议之间的桥梁，例如，[RabbitMQ broker](http://www.rabbitmq.com)支持高级消息队列协议（`AMQP`），消息队列遥测传输（`MQTT`）和 简单/流式文本定向消息协议（`STOMP`），支持不同消息协议的多个应用程序进行交互。

> [MQTT](http://mqtt.org)是一种轻量级消息传递协议，专为机器间通信（物联网）设计。 [AMQP](http://www.amqp.org)是一个更复杂的协议，旨在成为专有消息中间件的开源替代品。[STOMP](http://stomp.github.io)是一个轻量级的基于文本的协议，来自`HTTP school of design`。 这三个都是应用层协议，并且基于`TCP / IP`。

除了解耦和互操作性外，代理还可以提供更多高级功能，如持久队列，路由，消息转换和监控，而不提及许多代理支持的广泛的消息传递模式。当然，没有任何东西可以阻止我们使用对等体系结构实现所有这些功能，但不幸的是，还需要付出更多努力。尽管如此，避免使用代理的原因可能有所不同：

* 代理可能发生故障
* 代理必须扩展，而在对等体系结构中，我们只需要扩展单个节点
* 在没有代理的情况下交换消息可以大大减少传输的延迟

如果我们想要实现一个对等消息传递系统，我们也拥有更多的灵活性和能力，因为我们不受任何特定技术，协议或体系结构的约束。 [ØMQ](http://zeromq.org)是一个构建消息传递系统的库，其流行性很好地证明了我们可以通过构建定制的对等或混合体系结构获得灵活性。

## 发布/订阅模式
发布/订阅（通常缩写为`pub / sub`）可能是最着名的单向消息传递模式。我们应该已经熟悉它了，因为它不过是一个分布式的观察者模式。就观察者而言，我们有一组用户注册他们对接收特定类别的消息的兴趣。另一方面，发布者产生分布在所有相关用户中的消息。下图显示了发布/订阅模式的两个主要变体，第一个是点对点，第二个使用代理来调解通信：

![](http://oczira72b.bkt.clouddn.com/18-3-6/30922433.jpg)

让pub / sub如此特别的是，发布者不知道邮件的收件人是谁。正如我们所说的那样，用户必须注册它的监听器才能收到特定的消息，从而允许发布者与未知数量的接收者一起工作。换句话说，`pub / sub`模式的两边是松散耦合的，这使得它成为一个理想模式来集成不断发展的分布式系统的节点。

代理的存在进一步改善了系统节点之间的解耦，因为订阅者仅与代理交互，不知道哪个节点是消息发布者。正如我们稍后将看到的，代理还可以提供消息队列系统，即使在节点之间存在连接问题的情况下也可以实现可靠的传送。

现在，让我们以一个示例来演示这种模式。

### 构建一个简单的实时聊天应用程序
为了展示`pub / sub`模式如何帮助我们集成分布式体系结构的实例，现在我们将使用纯`WebSockets`构建一个非常基本的实时聊天应用程序。然后，我们将尝试通过运行多个实例并使用消息传递系统进行通信来扩展它。

#### 实现服务器端
现在，让我们一次一步。 首先构建我们的聊天应用程序; 为此，我们将依赖[ws](https://npmjs.org/package/ws)，它是`Node.js`的纯`WebSocket`实现。我们知道，在`Node.js`中实现实时应用程序非常简单，我们的代码将证实这一假设。然后让我们创建聊天的服务器端; 其内容如下（在`app.js`文件中）：

```javascript
const WebSocketServer = require('ws').Server;

// 静态的文件服务器
const server = require('http').createServer( //[1]
  require('ecstatic')({
    root: `${__dirname}/www`
  })
);

const wss = new WebSocketServer({
  server: server
}); //[2]
wss.on('connection', ws => {
  console.log('Client connected');
  ws.on('message', msg => { //[3]
    console.log(`Message: ${msg}`);
    broadcast(msg);
  });
});

function broadcast(msg) { //[4]
  wss.clients.forEach(client => {
    client.send(msg);
  });
}

server.listen(process.argv[2] || 8080);
```

这就是我们需要在服务器上实现聊天应用程序的全部内容。这是它的工作方式：

1. 我们首先创建一个`HTTP`服务器并附上名为`ecstatic`的中间件（ https://npmjs.org/package/ecstatic ）来提供静态文件。 这需要为我们的应用程序（`JavaScript`和`CSS`）的客户端资源提供服务。
2. 我们创建一个`WebSocket`服务器的新实例，并将其附加到我们现有的`HTTP`服务器上。然后，我们通过附加连接事件的事件侦听器来开始监听传入的`WebSocket`连接。
3. 每当新客户端连接到我们的服务器时，我们就开始监听收到的消息。当新消息到达时，我们将它广播给所有连接的客户端
4. `broadcast()`函数是对所有连接客户端进行广播，`send()`函数在其中的每一个客户端上被调用。

这是`Node.js`的魔力！ 当然，我们实现的服务器的功能非常少，仅仅实现了基本的功能，但正如我们将看到的，它能够工作。

#### 实现客户端
接下来，是时候实施我们聊天的客户端了;这也是一个非常小而简单的代码片段，基本上是一个包含一些基本`JavaScript`代码的最少的`HTML`页面。让我们在一个名为`www/index.html`的文件中创建这个页面，如下所示：

```html
<!DOCTYPE html>
<html>
  <head>
    <script>
      var ws = new WebSocket('ws://' + window.document.location.host);
      ws.onmessage = function(message) {
        var msgDiv = document.createElement('div');
        msgDiv.innerHTML = message.data;
        document.getElementById('messages').appendChild(msgDiv);
      };
      
      function sendMessage() {
        var message = document.getElementById('msgBox').value;
        ws.send(message);
      }
    </script>
  </head>
  <body>
    Messages:
    <div id='messages'></div>
    <input type='text' placeholder='Send a message' id='msgBox'>
    <input type='button' onclick='sendMessage()' value='Send'>
  </body>
</html>
```

我们创建的HTML页面并不需要太多解释; 它只是一个简单的`Web`页面。 我们使用本地`WebSocket`对象初始化与`Node.js`服务器的连接，然后开始监听来自服务器的消息，并在它们到达时将它们显示在新的`div`元素中。相反，我们使用简单的文本框和按钮来发送消息。

在停止或重新启动聊天服务器时，`WebSocket`连接将关闭，并且不会自动重新连接（如果要实现此则需要使用高级库，例如`Socket.io`）。 这意味着在服务器重新启动后重新刷新浏览器以重新建立连接（或实现重新连接机制，这里我们不会介绍）。

#### 运行和扩展聊天应用程序
我们可以尝试立即运行应用程序; 只需使用以下命令启动服务器即可：

```bash
node app 8080
```

> 要运行这个demo，您需要支持本机`WebSocket`的最新浏览器。这里有一个兼容的浏览器列表： http://caniuse.com/#feat=websockets 

打开浏览器，访问 http://localhost:8080 ：

![](http://oczira72b.bkt.clouddn.com/18-3-6/56502781.jpg)

我们现在要展示的是当我们尝试通过启动多个实例来扩展应用程序时发生的情况。让我们尝试这样做，让我们在另一个端口上启动另一台服务器：

```bash
node app 8081
```

缩放我们的聊天应用程序的理想结果应该是连接到两个不同服务器的两个客户端应该能够交换聊天消息。不幸的是，这不如我们所愿。 我们可以通过打开另一个浏览器选项卡来尝试打开 http://localhost:8081 。

在一个实例上发送聊天消息时，我们在本地广播一条消息，仅将其分发给连接到该特定服务器的客户端。实际上，两台服务器不会互相通话。 我们需要整合它们。

![](http://oczira72b.bkt.clouddn.com/18-3-6/27320700.jpg)

在实际的应用程序中，我们将使用负载平衡器来分配实例中的负载，但对于此演示，我们不会使用它。这使我们能够以确定性的方式访问每台服务器，以验证它与其它实例交互的方式。

### 使用Redis作为消息代理
我们通过引入[Redis](http://redis.io)开始分析最重要的`pub / sub`实现，这是一个非常快速和灵活的键/值存储，也被许多人定义为数据结构服务器。

Redis比消息代理更像是一个数据库；然而，在其众多功能中，有一对专门用于实现集中式发布/订阅模式的命令。
当然，与更先进的面向消息的中间件相比，这种实现非常简单和基本，但这是其受欢迎的主要原因之一。通常，实际上，`Redis`已经在现有基础架构中广泛使用，例如，作为缓存服务器或会话存储；它的速度和灵活性使其成为在分布式系统中共享数据的非常流行的选择。因此，只要项目中出现对发布/订阅代理的需求，最简单直接的选择就是重用`Redis`本身，避免安装和维护专用的消息代理。让我们以一个例子来展示它的功能。

> 这个例子需要安装`Redis`，监听它的默认端口。你可以在这里查看： https://redis.io/topics/quickstart 

我们计划使用`Redis`来作为聊天服务器的消息代理。每个实例都将从其客户端接收到的任何消息发布给代理，并同时订阅来自其他服务器实例的消息。正如我们所看到的，我们架构中的每个服务器都是订阅者和发布者。下图显示了我们想要获得的体系结构的表示形式：

![](http://oczira72b.bkt.clouddn.com/18-3-6/56153519.jpg)

通过查看上图，我们可以总结一条消息的经历如下：

1. 将消息输入到网页的文本框中并发送到聊天服务器的连接实例。
2. 邮件然后发布给代理。
3. 代理将消息分派给所有订阅者，在我们的体系结构中，所有订阅者都是聊天服务器的实例。
4. 在每种情况下，都会将消息分发给所有连接的客户端。

`Redis`允许发布和订阅由字符串标识的频道，例如`chat.nodejs`。它还允许我们使用`glob`风格的模式来定义可能匹配多个频道的订阅，例如`chat.*`。

我们在实践中看看它是如何工作的。让我们通过添加发布/订阅逻辑来修改服务器代码：

```javascript
const WebSocketServer = require('ws').Server;
const redis = require("redis");
const redisSub = redis.createClient();
const redisPub = redis.createClient();

// 静态文件服务器
const server = require('http').createServer(
  require('ecstatic')({root: `${__dirname}/www`})
);

const wss = new WebSocketServer({server: server});
wss.on('connection', ws => {
  console.log('Client connected');
  ws.on('message', msg => {
    console.log(`Message: ${msg}`);
    redisPub.publish('chat_messages', msg);
  });
});

redisSub.subscribe('chat_messages');
redisSub.on('message', (channel, msg) => {
  wss.clients.forEach((client) => {
    client.send(msg);
  });
});

server.listen(process.argv[2] || 8080);
```

我们对原始聊天服务器所做的更改在前面的代码中突出显示；下面来解释其工作原理：

1. 要将我们的`Node.js`应用程序连接到`Redis`服务器，我们使用[redis](https://npmjs.org/package/redis)，它是一个支持所有可用`Redis`命令的完整客户端。 接下来，我们实例化两个不同的连接，一个用于订阅`channel`，另一个用于发布消息。 这在`Redis`中是必需的，因为一旦连接进入用户模式，就只能使用与订阅相关的命令。 这意味着我们需要第二个连接来发布消息。

2. 当从连接的客户端收到新消息时，我们会在`chat_messages`通道中发布消息。我们不直接向客户广播该消息，因为我们所有的服务器订阅了同一个`channel`（我们稍后会看到），所以它会通过`Redis`返回给我们。 对于这个例子的范围来说，这是一个简单而有效的机制。
3. 正如我们所说的，我们的服务器还必须订阅`chat_messages`通道，因此我们注册一个侦听器来接收发布到该通道的所有消息（通过当前服务器或任何其他聊天服务器）。当收到消息时，我们只是将它广播给所有连接到当前`WebSocket`服务器的客户端。

这些少许的改变足以让聊天服务器信息互通。为了证明这一点，我们可以尝试启动我们应用程序的多个实例：

```bash
node app 8080
node app 8081
node app 8082
```

然后，我们可以将多个浏览器的选项卡连接到每个实例，并验证我们发送到一台服务器的消息是否被连接到不同服务器的所有其他客户端成功接收。恭喜！我们只使用发布/订阅模式集成了分布式实时应用程序。

![](http://oczira72b.bkt.clouddn.com/18-3-7/85762682.jpg)

![](http://oczira72b.bkt.clouddn.com/18-3-7/75060663.jpg)

### 使用ØMQ进行点对点发布/订阅
代理的存在可以大大简化消息传递系统的体系结构；但是，在某些情况下，它不是最佳解决方案，例如，当不能接受延时的情况下，扩展复杂的分布式系统时，或者当代理节点失败或发生异常的情况。

#### 介绍ØMQ
如果我们的项目可选择点对点消息交换模式，那最佳解决方案应该是[ØMQ](http://zeromq.org)，也称为`zmq`、`ZeroMQ`或`0MQ`）；我们在本书前面已经提到过这个库。`ØMQ`是一个网络库，提供构建各种消息模式的基本工具。它是低级的，速度非常快，并且具有简约的`API`，但它提供了消息传递系统的所有基本构建模块，例如原子消息，负载平衡，队列等等。它支持许多类型的传输，例如进程内通道（`inproc://`），进程间通信（`ipc://`），使用PGM协议（`pgm://`或`epgm://`）的多播，当然，经典的`TCP`（`tcp://`）。
在`ØMQ`的功能中，我们还可以找到实现发布/订阅模式的工具，这正是我们的例子所需要的。因此，我们现在要做的是从聊天应用程序的体系结构中删除代理（`Redis`），并让各个节点以对等方式进行通信，利用`ØMQ`的发布/订阅套接字。

> `ØMQ`套接字可以被视为类固化网络套接字，它提供了很多方法来帮助实现最常见的消息传递模式。例如，我们可以找到实现发布/订阅，请求/回复或单向通信的套接字。

#### 为聊天设计一个对等体系结构的服务器
当我们从架构中移除代理时，聊天应用程序的每个实例都必须直接连接到其他可用实例，以便接收他们发布的消息。 在ØMQ中，我们有两种专门为此设计的套接字：`PUB`和`SUB`。典型的模式是将`PUB`套接字绑定到一个端口，该端口将开始侦听来自其他`SUB`套接字的订阅。

订阅可以有一个过滤器，指定将传递到`SUB`套接字的消息。该过滤器是一个简单的二进制缓冲区（所以它也可以是一个字符串），它将与消息的开头（这也是一个二进制缓冲区）相匹配。当通过`PUB`套接字发送一条消息时，它将被广播到所有连接的`SUB`套接字，但仅在应用了它们的订阅过滤器之后。仅当使用连接的协议时，过滤器才会应用到发布方，例如`TCP`。

下图显示了应用于我们的分布式聊天服务器体系结构的模式（为简单起见，仅有两个实例）：

![](http://oczira72b.bkt.clouddn.com/18-3-7/11976098.jpg)

> 要运行本节中的示例，您需要在系统上安装本地`ØMQ`二进制文件。 你可以在 http://zeromq.org/intro:get-the-software 找到更多信息。注意：此示例已针对`ØMQ`的`4.0`分支进行了测试。

#### 使用`ØMQ`的`PUB / SUB`套接字
让我们通过修改我们的聊天服务器来看看它是如何工作的：

```javascript
const WebSocketServer = require('ws').Server;
const args = require('minimist')(process.argv.slice(2));
const zmq = require('zmq');

//static file server
const server = require('http').createServer(
  require('ecstatic')({root: `${__dirname}/www`})
);

const pubSocket = zmq.socket('pub');
pubSocket.bind(`tcp://127.0.0.1:${args['pub']}`);

const subSocket = zmq.socket('sub');
const subPorts = [].concat(args['sub']);
subPorts.forEach(p => {
  console.log(`Subscribing to ${p}`);
  subSocket.connect(`tcp://127.0.0.1:${p}`);
});
subSocket.subscribe('chat');

subSocket.on('message', msg => {
  console.log(`From other server: ${msg}`);
  broadcast(msg.toString().split(' ')[1]);
});

const wss = new WebSocketServer({server: server});
wss.on('connection', ws => {
  console.log('Client connected'); 
  ws.on('message', msg => {
    console.log(`Message: ${msg}`);
    broadcast(msg);
    pubSocket.send(`chat ${msg}`);
  });
});

function broadcast(msg) {
  wss.clients.forEach(client => {
    client.send(msg);
  });
}

server.listen(args['http'] || 8080);
```

前面的代码清楚地表明，我们的应用程序的逻辑变得稍微复杂一些；然而，考虑到我们正在实施分布式和点对点的发布/订阅模式，它仍然很简单。让我们看看所有的部分是如何结合在一起的：

1. 我们需要[zmq](https://npmjs.org/package/zmq)，它基本上是`ØMQ`库的`Node.js`版本。我们还需要[minimist](https://npmjs.org/package/minimist)，它是一个命令行参数解析器；我们需要这个能够轻松接受命名参数。
2. 我们立即创建我们的`PUB`套接字并将其绑定到 - `pub`命令行参数中提供的端口。
3. 我们创建`SUB`套接字，并将它连接到应用程序其他实例的`PUB`套接字。目标`PUB`套接字的端口在--`sub`命令行参数中提供（可能有多个）。然后，我们通过提供`chat`作为过滤器来创建实际订阅，这意味着我们只会收到以`chat`开始的消息。
4. 当我们的`WebSocket`接收到新消息时，我们将它广播给所有连接的客户端，但我们也通过`PUB`套接字发布它。 我们使用`chat`作为前缀，然后是空格，因此该消息将作为过滤器发布到所有使用`chat`的订阅者。
5. 我们开始监听到达我们`SUB`套接字的消息，我们对消息做一些简单的解析以删除聊天前缀，然后我们将它广播给所有连接到当前`WebSocket`服务器的客户端。

我们现在已经构建了一个简单的分布式系统，使用点对点发布/订阅模式进行集成！

让我们开始吧，让我们通过确保正确连接它们的`PUB`和`SUB`插槽来启动我们的应用程序的三个实例：

```bash
node app --http 8080 --pub 5000 --sub 5001 --sub 5002
node app --http 8081 --pub 5001 --sub 5000 --sub 5002
node app --http 8082 --pub 5002 --sub 5000 --sub 5001
```

第一个命令将启动一个`HTTP`服务器侦听端口`8080`的实例，同时在端口`5000`上绑定`PUB`套接字，并将`SUB`套接字连接到端口`5001`和`5002`，这是其他两个实例的`PUB`套接字应该侦听的端口。其他两个命令以类似的方式工作。

现在，我们可以看到的第一件事情是，如果与`PUB`套接字对应的端口不可用，`ØMQ`不会崩溃。例如，在第一个命令执行时，端口`5001`和`5002`仍然不可用；但是，`ØMQ`不会引发任何错误。这是因为`ØMQ`具有重连机制，它会自动尝试定期与这些端口建立连接。如果任何节点出现故障或重新启动，此功能特别适用。相同的逻辑适用于`PUB`套接字：如果没有订阅者，它将简单地删除所有消息，但它将继续工作。

此时，我们可以尝试使用浏览器导航到我们启动的任何服务器实例，并验证这些消息是否适当地向所有聊天服务器广播。

![](http://oczira72b.bkt.clouddn.com/18-3-7/85762682.jpg)

在前面的例子中，我们假设了一个静态体系结构，其中实例的数量和地址是事先已知的。我们可以引入一个服务注册表，如前一章所述，动态连接我们的实例。同样重要的是要指出`ØMQ`可以用来实现代理模式。

### 持久订阅者
消息传递系统中的一个重要抽象是消息队列（`MQ`）。对于消息队列，消息的发送者和接收者不需要同时处于活动状态和连接状态以建立通信，因为排队系统负责存储消息直到目的地能够 接收他们。 这种行为与`set and forget`范式相反，订户只能在消息系统连接期间才能接收消息。

一个能够始终可靠地接收所有消息的订阅者，即使是在没有收听这些消息时发送的消息，也被称为持久订阅者。

> `MQTT`协议为发送方和接收方之间交换的消息定义了服务质量（QoS）级别。这些级别对描述任何其他消息系统（不仅仅是`MQTT`）的可靠性也非常有用。如下描述：

`QoS0`，最多一次：也被称为“设置并忘记”，消息不会被保留，并且传送未被确认。这意味着在接收机崩溃或断开的情况下，信息可能会丢失。
`QoS1`，至少一次：保证至少收到一次该消息，但如果在通知发件人之前接收器崩溃，则可能发生重复。这意味着消息必须在必须再次发送的情况下持续下去。
`QoS2`，正好一次：这是最可靠的`QoS`; 它保证该消息只被接收一次。 这是以用于确认消息传递的更慢和更数据密集型机制为代价的。

> 请在MQTT规范中了解更多信息 http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt /mqtt-v3r1.html#qos-flows 

正如我们所说的，对于持久订阅者，我们的系统必须使用消息队列来在用户断开连接时累积消息。队列可以存储在内存中，也可以保存在磁盘上以允许恢复其消息，即使代理重新启动或崩溃。下图显示了由消息队列支持的持久订阅者：

![](http://oczira72b.bkt.clouddn.com/18-3-7/84465601.jpg)

持久订阅者可能是消息队列所支持的最重要的模式，但它肯定不是唯一的模式，我们将在本章后面看到。

`Redis`的发布/订阅命令实现了一个设置和遗忘机制（`QoS0`）。但是，`Redis`仍然可以用于使用其他命令的组合来实现持久订阅者（不直接依赖其发布/订阅实现）。您可以在以下博客文章中找到关于此技术的说明：

* https://davidmarquis.wordpress.com/2013/01/03/reliable-delivery-message-queues-with-redis/
* http://www.ericjperry.com/redis-message-queue/

`ØMQ`定义了一些支持持久订阅者的模式，但实现这种机制主要取决于我们。

#### 介绍`AMQP`
消息队列通常用于不能丢失消息的情况，其中包括任务关键型应用程序，如银行或金融系统。这通常意味着典型的企业级消息队列是一个非常复杂的软件，它使用`bulletproof protocols`和持久存储来保证即使在出现故障时也能传送消息。由于这个原因，企业消息传递中间件多年来一直是`Oracle`和`IBM`等巨头的特权，它们中的每一个通常都实施自己的专有协议，导致强大的客户锁定。幸运的是，由于诸如`AMQP`，`STOMP`和`MQTT`等开放协议的增长，邮件系统进入主流已经有几年了。为了理解消息队列系统的工作原理，现在我们将概述`AMQP`；这是了解如何使用基于此协议的典型`API`的基础。

`AMQP`是许多消息队列系统支持的开放标准协议。除了定义通用通信协议外，它还提供了描述路由，过滤，排队，可靠性和安全性的模型。在`AMQP`中，有三个基本组成部分：

* `Queue（队列）`：负责存储客户端消费的消息的数据结构。我们的应用程序推送消息到队列，供给一个或多个消费者。如果多个使用者连接到同一个队列，则这些消息会在它们之间进行负载平衡。 队列可以是以下之一：
  * `Durable（持久队列）`：这意味着如果代理重新启动，队列会自动重新创建。一个持久的队列并不意味着它的内容也被保留下来；实际上，只有标记为持久性的消息才会保存到磁盘，并在重新启动的情况下进行恢复。
  * `Exclusive（专有队列）`：这意味着队列只能绑定到一个特定的用户连接。当连接关闭时，队列被销毁。
  * `Auto-delete（自动删除队列）`：这会导致队列在最后一个用户断开连接时被删除。
* `Exchange（交换机）`：这是发布消息的地方。交换机根据它实现的算法将消息路由到一个或多个队列：
  * `Direct exchange（直接交换机）`：通过匹配路由键（例如，`chat.msg`）整个消息来路由消息。
  * `Topic exchange（主题交换机）`：它使用与路由密钥相匹配的类似`glob`的模式分发消息（例如，`chat.#`匹配以`chat`开始的所有路由密钥）。
  * `Fanout exchange（扇出交换机）`：它向所有连接的队列广播消息，忽略提供的任何路由密钥。
* `Binding（绑定）`：这是交换机和队列之间的链接。它还定义了路由键或用于过滤从交换机到达的消息的模式。

这些组件由代理管理，该代理公开用于创建和操作它们的`API`。当连接到代理时，客户端创建一个到连接的通道，负责维护与代理的通信状态。

> 在`AMQP`中，可以通过创建任何类型的非排他性或自动删除的队列来获得持久用户模式。

下图将所有这些组件放在一起：

![](http://oczira72b.bkt.clouddn.com/18-3-7/99221788.jpg)

`AMQP`模型比我们目前使用的消息系统（`Redis`和`ØMQ`）更复杂；但是，比起只使用原生发布/订阅机制，它提供了一系列功能和可靠性的保证。

> 您可以在`RabbitMQ`网站上找到`AMQP`模型的详细介绍： https://www.rabbitmq.com/tutorials/amqp-concepts.html 

#### 使用`AMQP`和`RabbitMQ`的持久订阅者
现在让我们练习一下我们了解持久订阅者和`AMQP`的内容，并开发一个小例子。不丢失任何消息很重要的典型场景是，我们希望保持微服务体系结构的不同服务同步；我们在前一章已经描述了这种集成模式。如果我们想要使用经纪商将所有服务保留在同一页面上，那么我们不会丢失任何信息是非常重要的，否则我们可能会处于不一致的状态。

##### 为聊天应用程序设计一个历史记录服务
现在让我们使用微服务方法扩展我们的小聊天应用程序。让我们添加一个历史记录服务，将我们的聊天消息保存在数据库中，这样当客户端连接时，我们可以查询服务并检索整个聊天记录。我们将使用[RabbitMQ broker](https://www.rabbitmq.com)和`AMQP`将历史记录服务器与聊天服务器相集成。

下图显示了我们的架构：

![](http://oczira72b.bkt.clouddn.com/18-3-7/63012877.jpg)

如前面的体系结构所述，我们将使用单个扇出交换机；我们不需要任何特定的路由，所以我们的场景不需要任何更复杂的交换。接下来，我们将为聊天服务器的每个实例创建一个队列。这些队列是排他性的；当聊天服务器处于脱机状态时，我们无意收到任何遗漏的消息，都会传送给历史记录服务器记录，最终还可以针对存储的消息实施更复杂的查询。实际上，这意味着我们的聊天服务器不是持久订阅者，并且只要连接关闭，它们的队列就会被销毁。

相反，历史记录服务器不能丢失任何信息; 否则，它不会达到其目的。我们要为它创建的队列必须耐用，以便在历史记录服务断开连接时发布的任何消息将保留在队列中，并在联机时交付。

我们将使用熟悉的`LevelUP`作为历史记录服务的存储引擎，而我们将使用[amqplib](https://npmjs.org/package/amqplib)，并通过`AMQP`协议连接到`RabbitMQ`。

> 以下示例需要工作的`RabbitMQ`服务器，侦听其默认端口。 有关更多信息，请参阅其官方安装指南： http://www.rabbitmq.com/download.html 

##### 使用`AMQP`实现可靠的历史记录服务
现在让我们实施我们的历史记录服务器！我们将创建一个独立的应用程序（典型的微服务），它在模块`historySvc.js`中实现。该模块由两部分组成：向客户端展示聊天记录的`HTTP`服务器，以及负责捕获聊天消息并将其存储在本地数据库中的`AMQP`使用者。

让我们来看看下面代码中的内容：

```javascript
const level = require('level');
const timestamp = require('monotonic-timestamp');
const JSONStream = require('JSONStream');
const amqp = require('amqplib');
const db = level('./msgHistory');

require('http').createServer((req, res) => {
  res.writeHead(200);
  db.createValueStream()
    .pipe(JSONStream.stringify())
    .pipe(res);
}).listen(8090);

let channel, queue;
amqp
  .connect('amqp://localhost')  // [1]
  .then(conn => conn.createChannel())
  .then(ch => {
    channel = ch;
    return channel.assertExchange('chat', 'fanout');  // [2]
  })
  .then(() => channel.assertQueue('chat_history'))  // [3]
  .then((q) => {
    queue = q.queue;
    return channel.bindQueue(queue, 'chat');  // [4]
  })
  .then(() => {
    return channel.consume(queue, msg => {  // [5]
      const content = msg.content.toString();
      console.log(`Saving message: ${content}`);
      db.put(timestamp(), content, err => {
        if (!err) channel.ack(msg);
      });
    });
  })
  .catch(err => console.log(err))
;
```

我们可以立即看到`AMQP`需要一些设置，这对创建和连接模型的所有组件都是必需的。 观察`amqplib`默认支持`Promises`也很有趣，所以我们大量利用它们来简化应用程序的异步步骤。让我们详细看看它是如何工作的：

1. 我们首先与`AMQP`代理建立连接，在我们的例子中是`RabbitMQ`。然后，我们创建一个`channel`，该`channel`类似于保持我们通信状态的会话。
2. 接下来，我们建立了我们的会话，名为`chat`。正如我们已经提到的那样，这是一种扇出交换机。 `assertExchange()`命令将确保代理中存在交换，否则它将创建它。
3. 我们还创建了我们的队列，名为`chat_history`。默认情况下，队列是持久的；不是排他性的，也不会自动删除，所以我们不需要传递任何额外的选项来支持持久订阅者。
4. 接下来，我们将队列绑定到我们以前创建的交换机。在这里，我们不需要任何其他特殊选项，例如路由键或模式，因为交换机是扇出类型的交换机，所以它不执行任何过滤。
5. 最后，我们可以开始监听来自我们刚创建的队列的消息。我们将使用时间戳记作为密钥（ https://npmjs.org/package/monotonic-timestamp ）在`LevelDB`数据库中收到的每条消息保存，以保持消息按日期排序。看到我们使用`channel.ack(msg)`来确认每条消息，并且只有在消息成功保存到数据库后，也很有趣。如果代理没有收到`ACK`（确认），则该消息将保留在队列中以供再次处理。这是`AMQP`将服务可靠性提升到全新水平的另一个重要特征。如果我们不想发送明确的确认，我们可以将选项`{noAck:true}`传递给`channel.consume() API`。

##### 将聊天应用程序与`AMQP`集成
要使用`AMQP`集成聊天服务器，我们必须使用与我们在历史记录服务器中实现的设置非常相似的设置，因此我们不打算在此重复。 但是，看看队列是如何创建的以及如何将新消息发布到交换中仍然很有趣。新的`app.js`文件的相关部分如下：

```javascript
const WebSocketServer = require('ws').Server;
const amqp = require('amqplib');
const JSONStream = require('JSONStream');
const request = require('request');
let httpPort = process.argv[2] || 8080;

const server = require('http').createServer(
  require('ecstatic')({root: `${__dirname}/www`})
);

let channel, queue;
amqp
  .connect('amqp://localhost')
  .then(conn => conn.createChannel())
  .then(ch => {
    channel = ch;
    return channel.assertExchange('chat', 'fanout');
  })
  .then(() => {
    return channel.assertQueue(`chat_srv_${httpPort}`, {exclusive: true});
  })
  .then(q => {
    queue = q.queue;
    return channel.bindQueue(queue, 'chat');
  })
  .then(() => {
    return channel.consume(queue, msg => {
      msg = msg.content.toString();
      console.log('From queue: ' + msg);
      broadcast(msg);
    }, {noAck: true});
  })
  .catch(err => console.log(err))
;

const wss = new WebSocketServer({server: server});
wss.on('connection', ws => {
  console.log('Client connected');
  //query the history service
  request('http://localhost:8090')
    .on('error', err => console.log(err))
    .pipe(JSONStream.parse('*'))
    .on('data', msg => ws.send(msg))
  ;
    
  ws.on('message', msg => {
    console.log(`Message: ${msg}`);
    channel.publish('chat', '', new Buffer(msg));
  }); 
}); 

function broadcast(msg) {
  wss.clients.forEach(client => client.send(msg));
}

server.listen(httpPort);
```

正如我们所提到的，我们的聊天服务器不需要成为持久的订阅者。 所以当我们创建我们的队列时，我们传递选项`{exclusive:true}`，指示队列被限制到当前连接，因此一旦聊天服务器关闭，它就会被销毁。

发布新消息也很容易; 我们只需要指定目标交换机（聊天）和一个路由键，在我们的情况下这是空的（''），因为我们正在使用扇出交换。

我们现在可以运行我们改进的聊天应用程序架构；为此，我们开始两个聊天服务器和历史服务：

```bash
node app 8080
node app 8081
node historySvc
```

现在看看我们的系统，特别是历史服务如何在停机的情况下运行，这一点很有意思。如果我们停止历史记录服务器并继续使用聊天应用程序的`Web UI`发送消息，我们将会看到，当历史记录服务器重新启动时，它将立即收到所有错过的消息。

![](http://oczira72b.bkt.clouddn.com/18-3-7/85762682.jpg)

## 管道和任务分配模式
在`Chapter9-Advanced Asynchronous Recipes`中，我们学习了如何将高耗能的任务委派给多个本地进程，但即使这是一种有效的方法，但它也无法在单个机器的边界之外进行缩放。在本节中，我们将看到如何在分布式架构中使用类似的模式，使用位于网络中任何位置的远程`worker`。

这个想法是有一个消息传递模式，允许我们跨多台机器传播任务。 这些任务可能是单独的工作块或者使用分而治之技术分割的更大任务。

如果我们看看下图所示的逻辑架构，我们应该能够认识到一种熟悉的模式：

![](http://oczira72b.bkt.clouddn.com/18-3-7/42776259.jpg)

从上图我们可以看到，发布/订阅模式不适合这种类型的应用程序，因为我们绝对不希望多个`worker`接收任务。 我们需要的是一种类似于负载均衡器的消息分发模式，它将每条消息分派给不同的消费者（在本例中也称为worker）。在消息传递系统术语中，这种模式被称为竞争消费者。

与上一章中我们看到的`HTTP`负载均衡器的一个重要区别是，在这里，消费者扮演着更积极的角色。事实上，我们将在后面看到，大多数情况下，生产者不是连接到消费者，而是连接到任务生产者或任务队列的消费者本身，以便接收新的工作。这对于可扩展系统来说是一个很大的优势，因为它可以在不修改生产者或采用服务注册表的情况下无缝增加`worker`数量。

另外，在通用消息传递系统中，我们不一定需要生产者和`worker`之间的请求/回复通信。相反，大多数情况下，首选的方法是使用单向异步通信，这可以实现更好的并行性和可伸缩性。在这样的体系结构中，消息可能总是以一个方向行进，创建管道，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-3-7/5900362.jpg)

管道允许我们构建非常复杂的处理体系结构，而不需要同步请求/应答通信的负担，通常导致更低的延迟和更高的吞吐量。 在上图中，我们可以看到消息如何在一组`worker`分布，并被转发到其他处理单元，然后聚合到通常称为接收器的单个节点（扇入）中。

在本节中，我们将通过分析两个最重要的变体，即点对点通信和代理模式为基础，来关注这些架构的构建。

> 管道与任务分配模式的组合也称为并行管道。

### ØMQ扇出/扇出模式
我们已经发现了`ØMQ`在构建点对点分布式体系结构方面的一些优势。在前一节中，我们使用`PUB`和`SUB`套接字向多个消费者传播单个消息；现在我们将看到如何使用称为`PUSH`和`PULL`的另一对套接字来构建并行管道。

#### PUSH/PULL套接字
直观地说，我们可以说`PUSH`套接字用于发送消息，而`PULL`套接字是用于接收的。这似乎是一个微不足道的组合；然而，它们有一些很好的特性，使它们成为构建单向通信系统的完美选择：

* 两者都可以在`connet`模式或`bind`模式下工作。换句话说，我们可以构建一个`PUSH`套接字并将其绑定到本地端口，以监听来自`PULL`套接字的传入连接，反之亦然，`PULL`套接字可以监听来自`PUSH`套接字的连接。消息总是以相同的方向传播，从`PUSH`到`PULL`；它只是连接的发起者可能是不同的。绑定模式是耐用节点（例如任务生产者和接收器）的最佳解决方案，而连接模式对于瞬态节点（例如任务工作者）来说是完美的。这使得瞬时节点的数量可以任意变化，而不会影响其它正在使用的节点。

* 如果有多个`PULL`套接字连接到单个`PUSH`套接字，则消息均匀分布在所有的`PULL`套接字中；在实践中，它们是负载均衡的（点对点负载平衡！）。另一方面，从多个`PUSH`套接字接收消息的`PULL`套接字将使用公平排队系统处理消息，这意味着它们将从所有负载是均衡的。

* 通过没有任何连接的`PULL`套接字的`PUSH`套接字发送的消息不会丢失；他们排队等待生产者，直到一个节点联机并开始提取消息。

我们现在开始了解`ØMQ`与传统`Web`服务的不同之处，它如何成为构建任何类型的消息传递系统的理想工具。

#### 使用ØMQ构建分布式hashsum cracker
现在是时候构建一个示例应用程序来查看我们刚刚描述的`PUSH / PULL`套接字的属性。
一个简单而迷人的应用程序可能是一个`hashsum cracker`，一个使用暴力破解技术来尝试将给定的`hashsum（MD5，SHA1等）`与给定字母表中每个可能的字符变体进行匹配的系统。 这个算法的负载量是很高的（ http://en.wikipedia.org/wiki/Embarrassingly_parallel ），它非常适合构建演示并行管道功能的示例。

对于我们的应用程序，我们希望通过一个节点来实现典型的并行管道，以在多个`worker`之间创建和分配任务，以及一个节点来收集所有结果。我们刚刚描述的系统可以使用以下体系结构在`ØMQ`中实现：

![](http://oczira72b.bkt.clouddn.com/18-3-7/32787784.jpg)

在我们的体系结构中，我们有一个`ventilator`，用于生成给定字母表中所有可能的字符变体，并将它们分发给一组`worker`，然后计算每个给定变体的哈希函数并尝试将其与输入的哈希函数进行匹配。如果找到匹配项，则结果将发送到结果收集器节点（`sink`）。

重点是`ventilator`和`sink`，而`worker`节点是随时在变化中的。这意味着每个`worker`将其`PULL`套接字连接到`ventilator`，并将其`PUSH`套接字连接到`ventilator`；通过这种方式，我们可以在不改变`ventilator`和`sink`中的任何参数的情况下，启动和停止我们想要的`worker`数量。

##### 实现ventilator
现在，让我们开始通过在名为`ventilator.js`的文件中为`ventilator`创建一个新模块来实现我们的系统：

```javascript
const zmq = require('zmq');
const variationsStream = require('variations-stream');
const alphabet = 'abcdefghijklmnopqrstuvwxyz';
const batchSize = 10000;
const maxLength = process.argv[2];
const searchHash = process.argv[3];

const ventilator = zmq.socket('push');  // [1]
ventilator.bindSync("tcp://*:5016");

let batch = [];
variationsStream(alphabet, maxLength)
  .on('data', combination => {
    batch.push(combination);
    if (batch.length === batchSize) {  // [2]
      const msg = {searchHash: searchHash, variations: batch};
      ventilator.send(JSON.stringify(msg));
      batch = [];
    }
  })
  .on('end', () => {
    //send remaining combinations
    const msg = {searchHash: searchHash, variations: batch};
    ventilator.send(JSON.stringify(msg));
  })
;
```

为避免产生太多变化，我们的生成器只使用英文字母的小写字母，并对生成的单词的大小设置限制。这个限制在输入中作为命令行参数（`maxLength`）与`hashsum`匹配（`searchHash`）一起提供。 我们使用名为[variation-stream的库](https://npmjs.org/package/variations-stream)来使用流式接口生成所有变体。

但是我们最感兴趣分析的部分是我们如何给`worker`分配任务：

1. 我们首先创建一个`PUSH`套接字，并将其绑定到本地端口`5000`；这是`worker`的`PULL`套接字将连接以接收任务的地方。
2. 我们将每个批次生成的变体进行分组，然后制作一条消息，其中包含匹配的散列和要检查的一批单词。这实质上是`worker`将接受的任务对象。当我们通过`ventilator`套接字调用`send()`时，消息将按循环分配传递给下一个可用的`worker`。

##### 实现worker
现在是实现`worker`（`worker.js`）的时候了：

```javascript
const zmq = require('zmq');
const crypto = require('crypto');
const fromVentilator = zmq.socket('pull');
const toSink = zmq.socket('push');

fromVentilator.connect('tcp://localhost:5016');
toSink.connect('tcp://localhost:5017');

fromVentilator.on('message', buffer => {
  const msg = JSON.parse(buffer);
  const variations = msg.variations;
  variations.forEach( word => {
    console.log(`Processing: ${word}`);
    const shasum = crypto.createHash('sha1');
    shasum.update(word);
    const digest = shasum.digest('hex');
    if (digest === msg.searchHash) {
      console.log(`Found! => ${word}`);
      toSink.send(`Found! ${digest} => ${word}`);
    }
  });
});
```

正如我们所说的，我们的`worker`在我们的体系结构中代表了一个临时节点，因此，它的套接字应连接到远程节点，而不是侦听传入连接。这正是我们在`worker`中所做的，我们创建了两个套接字：

* 连接到`ventilator`的`PULL`套接字
* 用于接收任务连接到接收器的`PUSH`套接字，用于传播结果

除此之外，我们的`worker`完成的工作非常简单：对于收到的每条消息，我们迭代它包含的一批单词，然后对每个单词计算`SHA1`校验和，并尝试将其与针对消息传递的`searchHash`进行匹配。当找到匹配时，结果被转发到接收器。

##### 实现sink
对于我们的例子来说，接收器是一个非常基本的结果收集器，它只是将`worker`接收的消息打印到控制台。 文件`sink.js`的内容如下所示：

```javascript
const zmq  = require('zmq');
const sink = zmq.socket('pull');
sink.bindSync("tcp://*:5017");

sink.on('message', buffer => {
  console.log('Message from worker: ', buffer.toString());
});
```

##### 运行应用
我们现在准备运行我们的应用程序；让我们开始几个`worker`和`sink`：

```bash
node worker
node worker
node sink
```

然后启动`ventilator`，指定要生成的单词的最大长度以及我们希望匹配的`SHA1`校验和。以下是参数的示例列表：

```bash
node ventilator 4 f8e966d1e207d02c44511a58dccff2f5429e9a3b
```

当运行上述命令时，`ventilator`将开始生成所有可能的单词，其长度至多为四个字符，并将它们分配给我们开始的工作人员，以及我们提供的校验和。计算结果（如果有的话）将显示在接收器应用程序的终端中。

## 请求/回复模式
处理消息传递系统通常意味着使用单向异步通信;发布/订阅就是一个很好的例子。

单向通信可以在并行性和效率方面给我们带来巨大的优势，但单靠它们无法解决我们所有的集成和通信问题。有时候，一个很好的请求/回复模式可能只是这项工作的完美工具。因此，在所有那些我们拥有异步单向通道的情况下，知道如何构建一个允许我们以请求/回复方式交换消息的模式是很重要的。这正是我们接下来要学习的内容。

### 关联ID
我们将要学习的第一个请求/回复模式称为关联ID，它表示在单向通道之上构建请求/回复模式的基本内容。

该模式包括标记每个请求的标识符，然后由接收方附加到响应中；通过这种方式，请求的发送者可以关联这两个消息并将响应返回给正确的处理程序。这优雅地解决了存在单向异步通道的问题，消息可以随时在任何方向传播。我们来看看下图中的例子：

![](http://oczira72b.bkt.clouddn.com/18-3-7/21513795.jpg)

前面的场景显示了如何使用关联`ID`使我们能够将每个响应与正确的请求进行匹配，即使这些响应是以不同的顺序发送和接收的。

#### 使用关联实现请求/答复模式
现在让我们开始通过选择最简单类型的单向通道（一个是点对点（它直接连接系统的两个节点）和一个全双工（消息可以双向传输））来进行尝试。

关于管道连接，我们可以找到例如`WebSockets`：它们在服务器和浏览器之间建立点对点连接，并且消息可以以任何方向传播。另一个例子是使用`child_process.fork()`生成子进程时创建的通信通道。我们应该已经知道了，我们在`Chapter9-Advanced Asynchronous Recipes`中看到了这个API。这个通道也是异步的：它只将父进程连接到子进程，并允许消息以任何方向传播。这可能是这个类别的最基本的渠道，所以这就是我们下一个例子中要用到的。

下一个应用程序的计划是构建一个抽象，以包装在父进程和子进程之间创建的通道隧道。这个抽象应该提供一个请求/回复通信隧道，通过用一个关联`ID`自动标记每个请求，然后将任何传入回复的`ID`与等待响应的请求处理程序列表进行匹配。

从`Chapter9-Advanced Asynchronous Recipes`中，我们应该记住父进程可以使用两个方法访问带有子进程的通道：

* `child.send(message)`
* `child.on('message',callback)`

以类似的方式，子进程可以使用以下方式访问父进程的通道：

* `process.send(message)`
* `process.on('message',callback)`

这意味着父进程中可用的隧道的`API`与子进程中可用的隧道的`API`相同；这将允许我们建立一个通用的方法，以便可以从通道的两端发送请求。

##### 抽象request
我们通过考虑负责发送新请求的部分开始构建这个抽象请求；让我们创建一个名为`request.js`的新文件：

```javascript
const uuid = require('node-uuid');

module.exports = channel => {
  const idToCallbackMap = {};  // [1]
  
  channel.on('message', message => {  // [2]
    const handler = idToCallbackMap[message.inReplyTo];
    if(handler) {
      handler(message.data);
    }
  });
  
  return function sendRequest(req, callback) {  // [3]
    const correlationId = uuid.v4();
    idToCallbackMap[correlationId] = callback;
    channel.send({
      type: 'request',
      data: req,
      id: correlationId
    });
  };
};
```

这就是我们的抽象请求的工作原理：

1. 看`request()`函数。该模式的神奇之处在于`idToCallbackMap`变量，它存储了传出请求与其回复处理程序之间的关联。
2. 一旦工厂被调用，我们所做的第一件事就是开始监听收到的消息。如果消息的关联`ID`（包含在`inReplyTo`属性中）与`idToCallbackMap`变量中包含的任何`ID`相匹配，我们知道我们刚收到一个回复，因此我们获得了对相关响应处理程序的引用，并且用 消息中包含的数据。
3. 最后，我们返回我们将用来发送新请求的函数。 其工作是使用[node-uuid](https://npmjs.org/package/node-uuid)生成关联`ID`，然后将请求数据包装起来，并指定关联ID`correlationId`和消息类型`type`。

这就是`request`模块；让我们转到下一部分。

##### 抽象reply
我们距实现完整的`request/reply`模式只有一步之遥，所以让我们看看`request.js`模块的对应的模块是如何工作的。我们创建另一个名为`reply.js`的文件，它将包含答复处理程序：

```javascript
module.exports = channel =>
{
  return function registerHandler(handler) {
    channel.on('message', message => {
      if (message.type !== 'request') return;
      handler(message.data, reply => {
        channel.send({
          type: 'response',
          data: reply,
          inReplyTo: message.id
        });
      });
    });
  };
};
```

我们的`reply`模块又是一个工厂，它返回一个函数来注册新的答复处理程序。这是在注册新处理程序时发生的情况：

1. 我们开始监听传入的请求，当我们收到请求时，我们立即通过传递消息的数据和回调函数来收集处理程序的回复来调用处理程序。
2. `handler`程序完成其工作后，它将调用我们提供的回调，并返回其答复。然后我们通过附加请求的关联`ID`（`inReplyTo`属性）来构建，然后我们将所有内容都放回到隧道中。

关于这种模式的惊人之处在于，在`Node.js`中，它非常容易；我们所有的东西都是异步的，所以建立在单向通道之上的异步请求/回复通信与其他任何异步操作并没有太大的不同，特别是当我们构建一个抽象方法来隐藏其实现细节时。

##### 尝试运行完整的request/reply模块
现在我们准备尝试运行我们新的异步request/reply模块。 让我们在一个名为`replier.js`的文件中创建一个示例`replier`：

```javascript
const reply = require('./reply')(process);

reply((req, cb) => {
  setTimeout(() => {
    cb({sum: req.a + req.b});
  }, req.delay);
});
```

我们的`replier`只需计算两个接收到的数字之间的和，并在某个延迟（也在请求中指定）之后返回结果。这将允许我们验证响应的顺序也可能与我们发送请求的顺序不同，以确认我们的模块正在工作。

完成示例的最后一步是在名为`requestor.js`的文件中创建请求者，该文件还具有使用`child_process.fork()`启动`replier`的任务：

```javascript
const replier = require('child_process')
                .fork(`${__dirname}/replier.js`);
const request = require('./request')(replier);

request({a: 1, b: 2, delay: 500}, res => {
  console.log('1 + 2 = ', res.sum);
  // 这应该是我们收到的最后一个回复，所以我们关闭了channel
  replier.disconnect();
});

request({a: 6, b: 1, delay: 100}, res => {
  console.log('6 + 1 = ', res.sum);
});
```

请求者启动`replier`，然后将其引用传递给我们的请求模块。然后，我们运行一些示例请求，并验证它们与收到的响应之间的关联是否正确。

要试用这个示例，只需启动`requestor.js`模块; 输出应该类似于以下内容：

```
6 + 1 = 7
1 + 2 = 3
```

![](http://oczira72b.bkt.clouddn.com/18-3-7/11271387.jpg)

这证实了我们的模式完美地工作，并且`reply`与他们自己的请求正确地相关联，不管他们以什么顺序发送或接收。

### 返回地址
关联`ID`是在单向信道之上创建请求/回复通信的基本模式；然而，当我们的消息架构拥有多个通道或队列，或者可能有多个请求者时，这还不够。在这些情况下，除了关联`ID`之外，我们还需要知道返回地址，这是允许回复者将回复发送回请求的原始发件人的一条信息。

#### 在AMQP中实现返回地址模式
在`AMQP`中，返回地址是请求者正在侦听传入回复的队列。因为响应只能由一个请求者接收，所以队列是私有的并且不在不同的使用者之间共享是很重要的。从这些属性中，我们可以推断出我们将需要一个暂时队列，将其作用于请求者的连接，并且应答者必须与返回队列建立点对点通信，以便能够传递其响应。

以下为我们提供了这种情况的一个例子：

![](http://oczira72b.bkt.clouddn.com/18-3-7/34375191.jpg)

为了在`AMQP`上创建请求/应答模式，我们需要做的就是在消息属性中指定响应队列的名称；这样，回复者知道应答消息必须传送到哪里。 这个理论看起来非常简单，所以我们来看看如何在真正的应用程序中实现它。

##### 实现request
现在让我们在`AMQP`之上构建一个请求/回复抽象。我们将使用`RabbitMQ`作为代理，但任何兼容的`AMQP`代理都应该可以完成这项工作。让我们从请求开始（在`amqpRequest.js`模块中实现）；我们只会在这里展示相关的部分。

第一件事情是我们如何创建队列来保存响应；看以下代码：

```javascript
channel.assertQueue('', {exclusive: true});
```

当我们创建队列时，我们没有指定任何名字，这意味着我们会选择一个随机的名字；除此之外，队列是独占的，这意味着它被绑定到活动的`AMQP`连接，并且在连接关闭时它将被销毁。没有必要将队列绑定到交换机，因为我们不需要任何路由或分配到多个队列；这意味着消息必须直接传递到我们的响应队列中。

接下来，让我们看看我们如何产生一个新的请求：

```javascript
class AMQPRequest {
  //...
  request(queue, message, callback) {
    const id = uuid.v4();
    this.idToCallbackMap[id] = callback;
    this.channel.sendToQueue(queue, new Buffer(JSON.stringify(message)), {
      correlationId: id,
      replyTo: this.replyQueue
    });
  }
}
```

`request()`方法接受请求队列的名称和要发送的消息作为输入。正如我们在前一节中所了解的，我们需要生成一个关联`ID`并将其关联到回调函数。最后，我们发送消息，指定`correlationId`和`replyTo`属性作为元数据。

有趣的是，为了发送消息，我们使用`channel.sentToQueue() API`而不是`channel.publish()`；这是因为我们不希望使用交换机来实施任何发布/订阅分发，而是直接进入目标队列的更基本的点对点传递。

> 在`AMQP`中，我们可以指定一组要传递给消费者的属性（或元数据）以及主要消息。

我们的`amqpRequest`类的最后一个重要部分是我们监听传入响应的地方：

```javascript
_listenForResponses() {
  return this.channel.consume(this.replyQueue, msg => {
    const correlationId = msg.properties.correlationId;
    const handler = this.idToCallbackMap[correlationId];
    if (handler) {
      handler(JSON.parse(msg.content.toString()));
    }
  }, {
    noAck: true
  });
}
```

在前面的代码中，我们监听我们明确创建的用于接收响应的队列中的消息，然后为每个传入消息读取关联`ID`，并将它与等待答复的处理程序列表进行匹配。一旦我们有了处理程序，我们只需要通过传递`reply`消息来调用它。

##### 实现reply
