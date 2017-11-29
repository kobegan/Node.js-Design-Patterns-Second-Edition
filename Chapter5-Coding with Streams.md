# Coding with Streams
`Streams`是`Node.js`最重要的组件和模式之一。 社区中有一句格言“Stream all the things（Steam就是所有的）”，仅此一点就足以描述流在`Node.js`中的地位。 `Dominic Tarr`作为`Node.js`社区的最大贡献者，它将流定义为`Node.js`最好，也是最难以理解的概念。

使`Node.js`的`Streams`如此吸引人还有其它原因; 此外，`Streams`不仅与性能或效率等技术特性有关，更重要的是它们的优雅性以及它们与`Node.js`的设计理念完美契合的方式。

在本章中，将会学到以下内容：
* `Streams`对于`Node.js`的重要性。
* 如何创建并使用`Streams`。
* `Streams`作为编程范式，不只是对于`I/O`而言，在多种应用场景下它的应用和强大的功能。
* 管道模式和在不同的配置中连接`Streams`。

## 发现Streams的重要性
在基于事件的平台（如`Node.js`）中，处理`I / O`的最有效的方法是实时处理，一旦有输入的信息，立马进行处理，一旦有需要输出的结果，也立马输出反馈。

在本节中，我们将首先介绍`Node.js`的`Streams`和它的优点。 请记住，这只是一个概述，因为本章后面将会详细介绍如何使用和组合`Streams`。

### Streams和Buffer的比较
我们在本书中几乎所有看到过的异步API都是使用的`Buffer`模式。 对于输入操作，`Buffer`模式会将来自资源的所有数据收集到`Buffer`区中; 一旦读取完整个资源，就会把结果传递给回调函数。 下图显示了这个范例的一个真实的例子：

![](http://oczira72b.bkt.clouddn.com/17-11-22/20243023.jpg)

从上图我们可以看到，在**t1**时刻，一些数据从资源接收并保存到缓冲区。 在**t2**时刻，最后一段数据被接收到另一个数据块，完成读取操作，这时，把整个缓冲区的内容发送给消费者。

另一方面，`Streams`允许你在数据到达时立即处理数据。 如下图所示：

![](http://oczira72b.bkt.clouddn.com/17-11-22/5372675.jpg)

这一张图显示了`Streams`如何从资源接收每个新的数据块，并立即提供给消费者，消费者现在不必等待缓冲区中收集所有数据再处理每个数据块。

但是这两种方法有什么区别呢？ 我们可以将它们概括为两点：

* 空间效率
* 时间效率

此外，`Node.js`的`Streams`具有另一个重要的优点：**可组合性（composability）**。 现在让我们看看这些属性对我们设计和编写应用程序的方式会产生什么影响。

### 空间效率
首先，`Streams`允许我们做一些看起来不可能的事情，通过缓冲数据并一次性处理。 例如，考虑一下我们必须读取一个非常大的文件，比如说数百`MB`甚至千`MB`。 显然，等待完全读取文件时返回大`Buffer`的`API`不是一个好主意。 想象一下，如果并发读取一些大文件， 我们的应用程序很容易耗尽内存。 除此之外，`V8`中的`Buffer`不能大于`0x3FFFFFFF`字节（小于`1GB`）。 所以，在耗尽物理内存之前，我们可能会碰壁。

### 使用Buffered的API进行压缩文件
举一个具体的例子，让我们考虑一个简单的命令行接口（`CLI`）的应用程序，它使用`Gzip`格式压缩文件。 使用`Buffered`的`API`，这样的应用程序在`Node.js`中大概这么编写（为简洁起见，省略了异常处理）：

```javascript
const fs = require('fs');
const zlib = require('zlib');
const file = process.argv[2];
fs.readFile(file, (err, buffer) => {
  zlib.gzip(buffer, (err, buffer) => {
    fs.writeFile(file + '.gz', buffer, err => {
      console.log('File successfully compressed');
    });
  });
});
```

现在，我们可以尝试将前面的代码放在一个叫做`gzip.js`的文件中，然后执行下面的命令：

```bash
node gzip <path to file>
```

如果我们选择一个足够大的文件，比如说大于`1GB`的文件，我们会收到一个错误信息，说明我们要读取的文件大于最大允许的缓冲区大小，如下所示：

```
RangeError: File size is greater than possible Buffer:0x3FFFFFFF
```

![](http://oczira72b.bkt.clouddn.com/17-11-22/34259823.jpg)

> 上面的例子中，没找到一个大文件，但确实对于大文件的读取速率慢了许多。

正如我们所预料到的那样，使用`Buffer`来进行大文件的读取显然是错误的。

### 使用Streams进行压缩文件
我们必须修复我们的`Gzip`应用程序，并使其处理大文件的最简单方法是使用`Streams`的`API`。 让我们看看如何实现这一点。 让我们用下面的代码替换刚创建的模块的内容：

```javascript
const fs = require('fs');
const zlib = require('zlib');
const file = process.argv[2];
fs.createReadStream(file)
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream(file + '.gz'))
  .on('finish', () => console.log('File successfully compressed'));
```

“是吗？”你可能会问。是的；正如我们所说的，由于`Streams`的接口和可组合性，因此我们还能写出这样的更加简洁，优雅和精炼的代码。 我们稍后会详细地看到这一点，但是现在需要认识到的重要一点是，程序可以顺畅地运行在任何大小的文件上，理想情况是内存利用率不变。 尝试一下（但考虑压缩一个大文件可能需要一段时间）。

### 时间效率
现在让我们考虑一个压缩文件并将其上传到远程`HTTP`服务器的应用程序的例子，该远程`HTTP`服务器进而将其解压缩并保存到文件系统中。如果我们的客户端是使用`Buffered`的`API`实现的，那么只有当整个文件被读取和压缩时，上传才会开始。 另一方面，只有在接收到所有数据的情况下，解压缩才会在服务器上启动。 实现相同结果的更好的解决方案涉及使用`Streams`。 在客户端机器上，`Streams`只要从文件系统中读取就可以压缩和发送数据块，而在服务器上，只要从远程对端接收到数据块，就可以解压每个数据块。 我们通过构建前面提到的应用程序来展示这一点，从服务器端开始。

我们创建一个叫做`gzipReceive.js`的模块，代码如下：

```javascript
const http = require('http');
const fs = require('fs');
const zlib = require('zlib');

const server = http.createServer((req, res) => {
  const filename = req.headers.filename;
  console.log('File request received: ' + filename);
  req
    .pipe(zlib.createGunzip())
    .pipe(fs.createWriteStream(filename))
    .on('finish', () => {
      res.writeHead(201, {
        'Content-Type': 'text/plain'
      });
      res.end('That\'s it\n');
      console.log(`File saved: ${filename}`);
    });
});

server.listen(3000, () => console.log('Listening'));
```

服务器从网络接收数据块，将其解压缩，并在接收到数据块后立即保存，这要归功于`Node.js`的`Streams`。

我们的应用程序的客户端将进入一个名为`gzipSend.js`的模块，如下所示：

在前面的代码中，我们再次使用`Streams`从文件中读取数据，然后在从文件系统中读取的同时压缩并发送每个数据块。

现在，运行这个应用程序，我们首先使用以下命令启动服务器：

```bash
node gzipReceive
```

然后，我们可以通过指定要发送的文件和服务器的地址（例如`localhost`）来启动客户端：

```bash
node gzipSend <path to file> localhost
```

![](http://oczira72b.bkt.clouddn.com/17-11-22/86343554.jpg)

如果我们选择一个足够大的文件，我们将更容易地看到数据如何从客户端流向服务器，但为什么这种模式下，我们使用`Streams`，比使用`Buffered`的`API`更有效率？ 下图应该给我们一个提示：

![](http://oczira72b.bkt.clouddn.com/17-11-22/17710786.jpg)

一个文件被处理的过程，它经过以下阶段：

1. 客户端从文件系统中读取
2. 客户端压缩数据
3. 客户端将数据发送到服务器
4. 服务端接收数据
5. 服务端解压数据
6. 服务端将数据写入磁盘 

为了完成处理，我们必须按照流水线顺序那样经过每个阶段，直到最后。在上图中，我们可以看到，使用`Buffered`的`API`，这个过程完全是顺序的。为了压缩数据，我们首先必须等待整个文件被读取完毕，然后，发送数据，我们必须等待整个文件被读取和压缩，依此类推。当我们使用`Streams`时，只要我们收到第一个数据块，流水线就会被启动，而不需要等待整个文件的读取。但更令人惊讶的是，当下一块数据可用时，不需要等待上一组任务完成；相反，另一条装配线是并行启动的。因为我们执行的每个任务都是异步的，这样显得很完美，所以可以通过`Node.js`来并行执行`Streams`的相关操作；唯一的限制就是每个阶段都必须保证数据块的到达顺序。

从前面的图可以看出，使用`Streams`的结果是整个过程花费的时间更少，因为我们不用等待所有数据被全部读取完毕和处理。

### 组合性
到目前为止，我们已经看到的代码已经告诉我们如何使用`pipe()`方法来组装`Streams`的数据块，`Streams`允许我们连接不同的处理单元，每个处理单元负责单一的职责（这是符合`Node.js`风格的）。这是可能的，因为`Streams`具有统一的接口，并且就`API`而言，不同`Streams`也可以很好的进行交互。唯一的先决条件是管道的下一个`Streams`必须支持上一个`Streams`生成的数据类型，可以是二进制，文本甚至是对象，我们将在后面的章节中看到。

为了证明`Streams`组合性的优势，我们可以尝试在我们先前构建的`gzipReceive / gzipSend`应用程序中添加加密功能。
为此，我们只需要通过向流水线添加另一个`Streams`来更新客户端。 确切地说，由`crypto.createChipher()`返回的流。 由此产生的代码应如下所示：

```javascript
const fs = require('fs');
const zlib = require('zlib');
const crypto = require('crypto');
const http = require('http');
const path = require('path');

const file = process.argv[2];
const server = process.argv[3];

const options = {
  hostname: server,
  port: 3000,
  path: '/',
  method: 'PUT',
  headers: {
    filename: path.basename(file),
    'Content-Type': 'application/octet-stream',
    'Content-Encoding': 'gzip'
  }
};

const req = http.request(options, res => {
  console.log('Server response: ' + res.statusCode);
});

fs.createReadStream(file)
  .pipe(zlib.createGzip())
  .pipe(crypto.createCipher('aes192', 'a_shared_secret'))
  .pipe(req)
  .on('finish', () => {
    console.log('File successfully sent');
  });
```

使用相同的方式，我们更新服务端的代码，使得它可以在数据块进行解压之前先解密：

```javascript
const http = require('http');
const fs = require('fs');
const zlib = require('zlib');
const crypto = require('crypto');

const server = http.createServer((req, res) => {
  const filename = req.headers.filename;
  console.log('File request received: ' + filename);
  req
    .pipe(crypto.createDecipher('aes192', 'a_shared_secret'))
    .pipe(zlib.createGunzip())
    .pipe(fs.createWriteStream(filename))
    .on('finish', () => {
      res.writeHead(201, {
        'Content-Type': 'text/plain'
      });
      res.end('That\'s it\n');
      console.log(`File saved: ${filename}`);
    });
});

server.listen(3000, () => console.log('Listening'));
```

> [crypto](https://nodejs.org/dist/latest-v9.x/docs/api/crypto.html#crypto_crypto)是Node.js的核心模块之一，提供了一系列加密算法。

只需几行代码，我们就在应用程序中添加了一个加密层。 我们只需要简单地通过把已经存在的`Streams`模块和加密层组合到一起，就可以。类似的，我们可以添加和合并其他`Streams`，如同在玩乐高积木一样。

显然，这种方法的主要优点是可重用性，但正如我们从目前为止所介绍的代码中可以看到的那样，`Streams`也可以实现更清晰，更模块化，更加简洁的代码。 出于这些原因，流通常不仅仅用于处理纯粹的`I / O`，而且它还是简化和模块化代码的手段。

## 开始使用Streams
在前面的章节中，我们了解了为什么`Streams`如此强大，而且它在`Node.js`中无处不在，甚至在`Node.js`的核心模块中也有其身影。 例如，我们已经看到，`fs`模块具有用于从文件读取的`createReadStream()`和用于写入文件的`createWriteStream()`，`HTTP`请求和响应对象本质上是`Streams`，并且`zlib`模块允许我们使用`Streams`式`API`压缩和解压缩数据块。

现在我们知道为什么`Streams`是如此重要，让我们退后一步，开始更详细地探索它。

### Streams的结构
`Node.js`中的每个`Streams`都是`Streams`核心模块中可用的四个基本抽象类之一的实现：

* `stream.Readable`
* `stream.Writable`
* `stream.Duplex`
* `stream.Transform`

每个`stream`类也是`EventEmitter`的一个实例。实际上，`Streams`可以产生几种类型的事件，比如`end`事件会在一个可读的`Streams`完成读取，或者错误读取，或其过程中产生异常时触发。

> 请注意，为简洁起见，在本章介绍的例子中，我们经常会忽略适当的错误处理。但是，在生产环境下中，总是建议为所有Stream注册错误事件侦听器。

`Streams`之所以如此灵活的原因之一是它不仅能够处理二进制数据，而且几乎可以处理任何`JavaScript`值。实际上，`Streams`可以支持两种操作模式：

* 二进制模式：以数据块形式（例如`buffers`或`strings`）流式传输数据
* 对象模式：将流数据视为一系列离散对象（这使得我们几乎可以使用任何`JavaScript`值）

这两种操作模式使我们不仅可以使用`I / O`流，而且还可以作为一种工具，以函数式的风格优雅地组合处理单元，我们将在本章后面看到。

> 在本章中，我们将主要使用在Node.js 0.11中引入的Node.js流接口，也称为版本3。 有关与旧接口差异的更多详细信息，请参阅StrongLoop在https://strongloop.com/strongblog/whats-new-io-js-beta-streams3/中的优秀博客文章。

### 可读的Streams
一个可读的`Streams`表示一个数据源，在`Node.js`中，它使用`stream`模块中的`Readableabstract`类实现。

#### 从Streams中读取信息
从可读`Streams`接收数据有两种方式：`non-flowing`模式和`flowing`模式。 我们来更详细地分析这些模式。

##### non-flowing模式（不流动模式）
从可读的`Streams`中读取数据的默认模式是为其附加一个可读事件侦听器，用于指示要读取的新数据的可用性。然后，在一个循环中，我们读取所有的数据，直到内部`buffer`被清空。这可以使用`read()`方法完成，该方法同步从内部缓冲区中读取数据，并返回表示数据块的`Buffer`或`String`对象。`read()`方法以如下使用模式：

```javascript
readable.read([size]);
```

使用这种方法，数据随时可以直接从`Streams`中按需提取。

为了说明这是如何工作的，我们创建一个名为`readStdin.js`的新模块，它实现了一个简单的程序，它从标准输入（一个可读流）中读取数据，并将所有数据回送到标准输出：

```javascript
process.stdin
  .on('readable', () => {
    let chunk;
    console.log('New data available');
    while ((chunk = process.stdin.read()) !== null) {
      console.log(
        `Chunk read: (${chunk.length}) "${chunk.toString()}"`
      );
    }
  })
  .on('end', () => process.stdout.write('End of stream'));
```

`read()`方法是一个同步操作，它从可读`Streams`的内部`Buffers`区中提取数据块。如果`Streams`在二进制模式下工作，返回的数据块默认为一个`Buffer`对象。

> 在以二进制模式工作的可读的Stream中，我们可以通过在Stream上调用setEncoding(encoding)来读取字符串而不是Buffer对象，并提供有效的编码格式（例如utf8）。

数据是从可读的侦听器中读取的，只要有新的数据，就会调用这个侦听器。当内部缓冲区中没有更多数据可用时，`read()`方法返回`null`；在这种情况下，我们不得不等待另一个可读的事件被触发，告诉我们可以再次读取或者等待表示`Streams`读取过程结束的`end`事件触发。当一个流以二进制模式工作时，我们也可以通过向`read()`方法传递一个`size`参数来指定我们想要读取的数据大小。这在实现网络协议或解析特定数据格式时特别有用。

现在，我们准备运行`readStdin`模块并进行实验。让我们在控制台中键入一些字符，然后按`Enter`键查看回显到标准输出中的数据。要终止流并因此生成一个正常的结束事件，我们需要插入一个`EOF`（文件结束）字符（在`Windows`上使用`Ctrl + Z`或在`Linux`上使用`Ctrl + D`）。

我们也可以尝试将我们的程序与其他程序连接起来;这可以使用管道运算符（`|`），它将程序的标准输出重定向到另一个程序的标准输入。例如，我们可以运行如下命令：

```bash
cat <path to a file> | node readStdin
```

这是流式范例是一个通用接口的一个很好的例子，它使得我们的程序能够进行通信，而不管它们是用什么语言写的。

##### flowing模式（流动模式）
从`Streams`中读取的另一种方法是将侦听器附加到`data`事件；这会将`Streams`切换为`flowing`模式，其中数据不是使用`read()`函数来提取的，而是一旦有数据到达`data`监听器就被推送到监听器内。例如，我们之前创建的`readStdin`应用程序将使用流动模式：

```javascript
process.stdin
  .on('data', chunk => {
    console.log('New data available');
    console.log(
      `Chunk read: (${chunk.length}) "${chunk.toString()}"`
    );
  })
  .on('end', () => process.stdout.write('End of stream'));
```

`flowing`模式是旧版`Streams`接口（也称为`Streams1`）的继承，其灵活性较低，`API`较少。随着`Streams2`接口的引入，`flowing`模式不是默认的工作模式，要启用它，需要将侦听器附加到`data`事件或显式调用`resume()`方法。 要暂时中断`Streams`触发`data`事件，我们可以调用`pause()`方法，导致任何传入数据缓存在内部`buffer`中。

> 调用pause()不会导致Streams切换回non-flowing模式。

#### 实现可读的Streams
现在我们知道如何从`Streams`中读取数据，下一步是学习如何实现一个新的`Readable`数据流。为此，有必要通过继承`stream.Readable`的原型来创建一个新的类。 具体流必须提供`_read()`方法的实现：

```javascript
readable._read(size)
```

`Readable`类的内部将调用`_read()`方法，而该方法又将启动
使用`push()`填充内部缓冲区：

> 请注意，read()是Stream消费者调用的方法，而_read()是一个由Stream子类实现的方法，不能直接调用。下划线通常表示该方法为私有方法，不应该直接调用。

为了演示如何实现新的可读`Streams`，我们可以尝试实现一个生成随机字符串的`Streams`。 我们来创建一个名为`randomStream.js`的新模块，它将包含我们的字符串的`generator`的代码：

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

在文件顶部，我们将加载我们的依赖关系。除了我们正在加载一个[chance的npm模块](https://npmjs.org/package/chance)之外，没有什么特别之处，它是一个用于生成各种随机值的库，从数字到字符串到整个句子都能生成随机值。

下一步是创建一个名为`RandomStream`的新类，并指定`stream.Readable`作为其父类。 在前面的代码中，我们调用父类的构造函数来初始化其内部状态，并将收到的`options`参数作为输入。通过`options`对象传递的可能参数包括以下内容：

* 用于将`Buffers`转换为`Strings`的`encoding`参数（默认值为`null`）
* 是否启用对象模式（`objectMode`默认为`false`）
* 存储在内部`buffer`区中的数据的上限，一旦超过这个上限，则暂停从`data source`读取（`highWaterMark`默认为`16KB`）

好的，现在让我们来解释一下我们重写的`stream.Readable`类的`_read()`方法：

* 该方法使用`chance`生成随机字符串。
* 它将字符串`push`内部`buffer`。 请注意，由于我们`push`的是`String`，此外我们还指定了编码为`utf8`（如果数据块只是一个二进制`Buffer`，则不需要）。
* 以`5%`的概率随机中断`stream`的随机字符串产生，通过`push` `null`到内部`Buffer`来表示`EOF`，即`stream`的结束。

我们还可以看到在`_read()`函数的输入中给出的`size`参数被忽略了，因为它是一个建议的参数。 我们可以简单地把所有可用的数据都`push`到内部的`buffer`中，但是如果在同一个调用中有多个推送，那么我们应该检查`push()`是否返回`false`，因为这意味着内部`buffer`已经达到了`highWaterMark`限制，我们应该停止添加更多的数据。

以上就是`RandomStream`模块，我们现在准备好使用它。我们来创建一个名为`generateRandom.js`的新模块，在这个模块中我们实例化一个新的`RandomStream`对象并从中提取一些数据：

```javascript
const RandomStream = require('./randomStream');
const randomStream = new RandomStream();

randomStream.on('readable', () => {
  let chunk;
  while ((chunk = randomStream.read()) !== null) {
    console.log(`Chunk received: ${chunk.toString()}`);
  }
});
```

现在，一切都准备好了，我们尝试新的自定义的`stream`。 像往常一样简单地执行`generateRandom`模块，观察随机的字符串在屏幕上流动。

### 可写的Streams
一个可写的`stream`表示一个数据终点，在`Node.js`中，它使用`stream`模块中的`Writable`抽象类来实现。

#### 写入一个stream
把一些数据放在可写入的`stream`中是一件简单的事情， 我们所要做的就是使用`write()`方法，它具有以下格式：

```javascript
writable.write(chunk, [encoding], [callback])
```

`encoding`参数是可选的，其在`chunk`是`String`类型时指定（默认为`utf8`，如果`chunk`是`Buffer`，则忽略）；当数据块被刷新到底层资源中时，`callback`就会被调用，`callback`参数也是可选的。

为了表示没有更多的数据将被写入`stream`中，我们必须使用`end()`方法：

```javascript
writable.end([chunk], [encoding], [callback])
```

我们可以通过`end()`方法提供最后一块数据。在这种情况下，`callbak`函数相当于为`finish`事件注册一个监听器，当数据块全部被写入`stream`中时，会触发该事件。

现在，让我们通过创建一个输出随机字符串序列的小型`HTTP`服务器来演示这是如何工作的：

```javascript
const Chance = require('chance');
const chance = new Chance();

require('http').createServer((req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/plain'
  }); //[1]
  while (chance.bool({
      likelihood: 95
    })) { //[2]
    res.write(chance.string() + '\n'); //[3]
  }
  res.end('\nThe end...\n'); //[4]
  res.on('finish', () => console.log('All data was sent')); //[5]
}).listen(8080, () => console.log('Listening on http://localhost:8080'));
```

我们创建了一个`HTTP服务器`，并把数据写入`res`对象，`res`对象是`http.ServerResponse`的一个实例，也是一个可写入的`stream`。下面来解释上述代码发生了什么：

1. 我们首先写`HTTP response`的头部。请注意，`writeHead()`不是`Writable`接口的一部分，实际上，这个方法是`http.ServerResponse`类公开的辅助方法。
2. 我们开始一个`5％`的概率终止的循环（进入循环体的概率为`chance.bool()`产生，其为`95％`）。
3. 在循环内部，我们写入一个随机字符串到`stream`。
4. 一旦我们不在循环中，我们调用`stream`的`end()`，表示没有更多
数据块将被写入。另外，我们在结束之前提供一个最终的字符串写入流中。
5. 最后，我们注册一个`finish`事件的监听器，当所有的数据块都被刷新到底层`socket`中时，这个事件将被触发。

我们可以调用这个小模块称为`entropyServer.js`，然后执行它。要测试这个服务器，我们可以在地址`http：// localhost:8080`打开一个浏览器，或者从终端使用`curl`命令，如下所示：

```bash
curl localhost:8080
```

此时，服务器应该开始向您选择的`HTTP客户端`发送随机字符串（请注意，某些浏览器可能会缓冲数据，并且流式传输行为可能不明显）。

#### Back-pressure（反压）
类似于在真实管道系统中流动的液体，`Node.js`的`stream`也可能遭受瓶颈，数据写入速度可能快于`stream`的消耗。 解决这个问题的机制包括缓冲输入数据；然而，如果数据`stream`没有给生产者任何反馈，我们可能会产生越来越多的数据被累积到内部缓冲区的情况，导致内存泄露的发生。

为了防止这种情况的发生，当内部`buffer`超过`highWaterMark`限制时，`writable.write()`将返回`false`。 可写入的`stream`具有`highWaterMark`属性，这是`write()`方法开始返回`false`的内部`Buffer`区大小的限制，一旦`Buffer`区的大小超过这个限制，表示应用程序应该停止写入。 当缓冲器被清空时，会触发一个叫做`drain`的事件，通知再次开始写入是安全的。 这种机制被称为`back-pressure`。

> 本节介绍的机制同样适用于可读的stream。事实上，在可读stream中也存在back-pressure，并且在_read()内调用的push()方法返回false时触发。 但是，这对于stream实现者来说是一个特定的问题，所以我们将不经常处理它。

我们可以通过修改之前创建的`entropyServer`模块来演示可写入的`stream`的`back-pressure`：

```javascript
const Chance = require('chance');
const chance = new Chance();

require('http').createServer((req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/plain'
  });

  function generateMore() { //[1]
    while (chance.bool({
        likelihood: 95
      })) {
      const shouldContinue = res.write(
        chance.string({
          length: (16 * 1024) - 1
        }) //[2]
      );
      if (!shouldContinue) { //[3]
        console.log('Backpressure');
        return res.once('drain', generateMore);
      }
    }
    res.end('\nThe end...\n', () => console.log('All data was sent'));
  }
  generateMore();
}).listen(8080, () => console.log('Listening on http://localhost:8080'));
```

前面代码中最重要的步骤可以概括如下：

1. 我们将主逻辑封装在一个名为`generateMore()`的函数中。
2. 为了增加获得一些`back-pressure`的机会，我们将数据块的大小增加到`16KB-1Byte`，这非常接近默认的`highWaterMark`限制。
3. 在写入一大块数据之后，我们检查`res.write()`的返回值。 如果它返回`false`，这意味着内部`buffer`已满，我们应该停止发送更多的数据。在这种情况下，我们从函数中退出，然后新注册一个写入事件的发布者，当`drain`事件触发时调用`generateMore`。

如果我们现在尝试再次运行服务器，然后使用`curl`生成客户端请求，则很可能会有一些`back-pressure`，因为服务器以非常高的速度生成数据，速度甚至会比底层`socket`更快。

#### 实现可写入的Streams
我们可以通过继承`stream.Writable`类来实现一个新的可写入的流，并为`_write()`方法提供一个实现。实现一个我们自定义的可写入的`Streams`类。

让我们构建一个可写入的`stream`，它接收对象的格式如下：

```
{
  path: <path to a file>
  content: <string or buffer>
}
```

这个类的作用是这样的：对于每一个对象，我们的`stream`必须将`content`部分保存到在给定路径中创建的文件中。 我们可以立即看到，我们`stream`的输入是对象，而不是`Strings`或`Buffers`，这意味着我们的`stream`必须以对象模式工作。

调用模块`toFileStream.js`：

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

作为第一步，我们加载所有我们所需要的依赖包。注意，我们需要模块`mkdirp`，正如你应该从前几章中所知道的，它应该使用`npm`安装。

我们创建了一个新类，它从`stream.Writable`扩展而来。

我们不得不调用父构造函数来初始化其内部状态；我们还提供了一个`option`对象作为参数，用于指定流在对象模式下工作（`objectMode：true`）。`stream.Writable`接受的其他选项如下：

* `highWaterMark`（默认值是`16KB`）：控制`back-pressure`的上限。
* `decodeStrings`（默认为`true`）：在字符串传递给`_write()`方法之前，将字符串自动解码为二进制`buffer`区。 在对象模式下这个参数被忽略。

最后，我们为`_write()`方法提供了一个实现。正如你所看到的，这个方法接受一个数据块，一个编码方式（只有在二进制模式下，`stream`选项`decodeStrings`设置为`false`时才有意义）。

另外，该方法接受一个回调函数，该函数在操作完成时需要调用；而不必要传递操作的结果，但是如果需要的话，我们仍然可以传递一个`error`对象，这将导致`stream`触发`error`事件。

现在，为了尝试我们刚刚构建的`stream`，我们可以创建一个名为`writeToFile.js`的新模块，并对该流执行一些写操作：

```javascript
const ToFileStream = require('./toFileStream.js');
const tfs = new ToFileStream();

tfs.write({path: "file1.txt", content: "Hello"});
tfs.write({path: "file2.txt", content: "Node.js"});
tfs.write({path: "file3.txt", content: "Streams"});
tfs.end(() => console.log("All files created"));
```

有了这个，我们创建并使用了我们的第一个自定义的可写入流。 像往常一样运行新模块来检查其输出；你会看到执行后会创建三个新文件。

### 双重的Streams
双重的`stream`既是可读的，也可写的。 当我们想描述一个既是数据源又是数据终点的实体时（例如`socket`），这就显得十分有用了。 双工流继承`stream.Readable`和`stream.Writable`的方法，所以它对我们来说并不新鲜。这意味着我们可以`read()`或`write()`数据，或者可以监听`readable`和`drain`事件。

要创建一个自定义的双重`stream`，我们必须为`_read()`和`_write()`提供一个实现。传递给`Duplex()`构造函数的`options`对象在内部被转发给`Readable`和`Writable`的构造函数。`options`参数的内容与前面讨论的相同，`options`增加了一个名为`allowHalfOpen`值（默认为`true`），如果设置为`false`，则会导致只要`stream`的一方（`Readable`和`Writable`）结束，`stream`就结束了。

> 为了使双重的stream在一方以对象模式工作，而在另一方以二进制模式工作，我们需要在流构造器中手动设置以下属性：

```javascript
this._writableState.objectMode
this._readableState.objectMode
```

### 转换的Streams
转换的`Streams`是专门设计用于处理数据转换的一种特殊类型的双重`Streams`。

在一个简单的双重`Streams`中，从`stream`中读取的数据和写入到其中的数据之间没有直接的关系（至少`stream`是不可知的）。 想想一个`TCP socket`，它只是向远程节点发送数据和从远程节点接收数据。`TCP socket`自身没有意识到输入和输出之间有任何关系。

下图说明了双重`Streams`中的数据流：

![](http://oczira72b.bkt.clouddn.com/17-11-28/19665538.jpg)

另一方面，转换的`Streams`对从可写入端接收到的每个数据块应用某种转换，然后在其可读端使转换的数据可用。

下图显示了数据如何在转换的`Streams`中流动：

![](http://oczira72b.bkt.clouddn.com/17-11-28/57545819.jpg)

从外面看，转换的`Streams`的接口与双重`Streams`的接口完全相同。但是，当我们想要构建一个新的双重`Streams`时，我们必须提供`_read()`和`_write()`方法，而为了实现一个新的变换流，我们必须填写另一对方法：`_transform()`和`_flush()`）。

我们来演示如何用一个例子来创建一个新的转换的`Streams`。

#### 实现转换的Streams
我们来实现一个转换的`Streams`，它将替换给定所有出现的字符串。 要做到这一点，我们必须创建一个名为`replaceStream.js`的新模块。 让我们直接看怎么实现它：

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

与往常一样，我们将从其依赖项开始构建模块。这次我们没有使用第三方模块。

然后我们创建了一个从`stream.Transform`基类继承的新类。该类的构造函数接受两个参数：`searchString`和`replaceString`。 正如你所想象的那样，它们允许我们定义要匹配的文本以及用作替换的字符串。 我们还初始化一个将由`_transform()`方法使用的`tailPiece`内部变量。

现在，我们来分析一下`_transform()`方法，它是我们新类的核心。`_transform()`方法与可写入的`stream`的`_write()`方法具有几乎相同的格式，但不是将数据写入底层资源，而是使用`this.push()`将其推入内部`buffer`，这与我们会在可读流的`_read()`方法中执行。这显示了转换的`Streams`的双方如何实际连接。

`ReplaceStream`的`_transform()`方法实现了我们这个新类的核心。正常情况下，搜索和替换`buffer`区中的字符串是一件容易的事情；但是，当数据流式传输时，情况则完全不同，可能的匹配可能分布在多个数据块中。代码后面的程序可以解释如下：

1. 我们的算法使用`searchString`函数作为分隔符来分割块。
2. 然后，它取出分隔后生成的数组的最后一项`lastPiece`，并提取其最后一个字符`searchString.length - 1`。结果被保存到`tailPiece`变量中，它将会被作为下一个数据块的前缀。
3. 最后，所有从`split()`得到的片段用`replaceString`作为分隔符连接在一起，并推入内部`buffer`区。

当`stream`结束时，我们可能仍然有最后一个`tailPiece`变量没有被压入内部缓冲区。这正是`_flush()`方法的用途；它在`stream`结束之前被调用，并且这是我们最终有机会完成流或者在完全结束流之前推送任何剩余数据的地方。

`_flush()`方法只需要一个回调函数作为参数，当所有的操作完成后，我们必须确保调用这个回调函数。完成了这个，我们已经完成了我们的`ReplaceStream`类。

现在，是时候尝试新的`stream`。我们可以创建另一个名为`replaceStreamTest.js`的模块来写入一些数据，然后读取转换的结果：

```javascript
const ReplaceStream = require('./replaceStream');

const rs = new ReplaceStream('World', 'Node.js');
rs.on('data', chunk => console.log(chunk.toString()));

rs.write('Hello W');
rs.write('orld!');
rs.end();
```

为了使得这个例子更复杂一些，我们把搜索词分布在两个不同的数据块上；然后，使用`flowing`模式，我们从同一个`stream`中读取数据，记录每个已转换的块。运行前面的程序应该产生以下输出：

```
Hel
lo Node.js
!
```

> 有一个值得提及是，第五种类型的stream：stream.PassThrough。 与我们介绍的其他流类不同，PassThrough不是抽象的，可以直接实例化，而不需要实现任何方法。实际上，这是一个可转换的stream，它可以输出每个数据块，而不需要进行任何转换。

### 使用管道连接Streams
`Unix`管道的概念是由`Douglas Mcllroy`发明的；这使程序的输出能够连接到下一个的输入。看看下面的命令：

```bash
echo Hello World! | sed s/World/Node.js/g
```

在前面的命令中，`echo`会将`Hello World!`写入标准输出，然后被重定向到`sed`命令的标准输入（因为有管道操作符 `|`）。 然后`sed`用`Node.js`替换任何`World`，并将结果打印到它的标准输出（这次是控制台）。

以类似的方式，可以使用可读的`Streams`的`pipe()`方法将`Node.js`的`Streams`连接在一起，它具有以下接口：

```javascript
readable.pipe(writable, [options])
```

非常直观地，`pipe()`方法将从可读的`Streams`中发出的数据抽取到所提供的可写入的`Streams`中。 另外，当可读的`Streams`发出`end`事件（除非我们指定`{end：false}`作为`options`）时，可写入的`Streams`将自动结束。 `pipe()`方法返回作为参数传递的可写入的`Streams`，如果这样的`stream`也是可读的（例如双重或可转换的`Streams`），则允许我们创建链式调用。

将两个`Streams`连接到一起时，则允许数据自动流向可写入的`Streams`，所以不需要调用`read()`或`write()`方法；但最重要的是不需要控制`back-pressure`，因为它会自动处理。

举个简单的例子（将会有大量的例子），我们可以创建一个名为`replace.js`的新模块，它接受来自标准输入的文本流，应用替换转换，然后将数据返回到标准输出：

```javascript
const ReplaceStream = require('./replaceStream');
process.stdin
  .pipe(new ReplaceStream(process.argv[2], process.argv[3]))
  .pipe(process.stdout);
```

上述程序将来自标准输入的数据传送到`ReplaceStream`，然后返回到标准输出。 现在，为了实践这个小应用程序，我们可以利用`Unix`管道将一些数据重定向到它的标准输入，如下所示：

```bash
echo Hello World! | node replace World Node.js
```

运行上述程序，会输出如下结果：

```
Hello Node.js
```

这个简单的例子演示了`Streams`（特别是文本`Streams`）是一个通用接口，管道几乎是构成和连接所有这些接口的通用方式。

> `error`事件不会通过管道自动传播。举个例子，看如下代码片段：

```javascript
stream1
  .pipe(stream2)
  .on('error', function() {});
```

> 在前面的链式调用中，我们将只捕获来自`stream2`的错误，这是由于我们给其添加了`erorr`事件侦听器。这意味着，如果我们想捕获从`stream1`生成的任何错误，我们必须直接附加另一个错误侦听器。 稍后我们将看到一种可以实现共同错误捕获的另一种模式（合并`Streams`）。 此外，我们应该注意到，如果目标`Streams`（读取的`Streams`）发出错误，它将会对源`Streams`通知一个`error`，之后导致管道的中断。

#### Streams如何通过管道
到目前为止，我们创建自定义`Streams`的方式并不完全遵循`Node`定义的模式；实际上，从`stream`基类继承是违反`small surface area`的，并需要一些示例代码。 这并不意味着`Streams`设计得不好，实际上，我们不应该忘记，因为`Streams`是`Node.js`核心的一部分，所以它们必须尽可能地灵活，广泛拓展`Streams`以致于用户级模块能够将它们充分运用。

然而，大多数情况下，我们并不需要原型继承可以给予的所有权力和可扩展性，但通常我们想要的仅仅是定义新`Streams`的一种快速开发的模式。`Node.js`社区当然也为此创建了一个解决方案。 一个完美的例子是[through2](https://npmjs.org/package/through2)，一个使得我们可以简单地创建转换的`Streams`的小型库。 通过`through2`，我们可以通过调用一个简单的函数来创建一个新的可转换的`Streams`：

```javascript
const transform = through2([options], [_transform], [_flush]);
```

类似的，[from2](https://npmjs.org/package/from2)也允许我们像下面这样创建一个可读的`Streams`：

```javascript
const readable = from2([options], _read);
```

接下来，我们将在本章其余部分展示它们的用法，那时，我们会清楚使用这些小型库的好处。

> [through](https://npmjs.org/package/through)和[from](https://www.npmjs.com/package/from)是基于`Stream1`规范的顶层库。

#### 基于Streams的异步控制流
