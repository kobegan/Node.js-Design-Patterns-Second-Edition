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

### 基于Streams的异步控制流
通过我们已经介绍的例子，应该清楚的是，`Streams`不仅可以用来处理`I / O`，而且可以用作处理任何类型数据的优雅编程模式。 但优点并不止这些；还可以利用`Streams`来实现异步控制流，在本节将会看到。

#### 顺序执行
默认情况下，`Streams`将按顺序处理数据；例如，转换的`Streams`的`_transform()`函数在前一个数据块执行`callback()`之后才会进行下一块数据块的调用。这是`Streams`的一个重要属性，按正确顺序处理每个数据块至关重要，但是也可以利用这一属性将`Streams`实现优雅的传统控制流模式。

代码总是比太多的解释要好得多，所以让我们来演示一下如何使用流来按顺序执行异步任务的例子。让我们创建一个函数来连接一组接收到的文件作为输入，确保遵守提供的顺序。我们创建一个名为`concatFiles.js`的新模块，并从其依赖开始：

```javascript
const fromArray = require('from2-array');
const through = require('through2');
const fs = require('fs');
```

我们将使用`through2`来简化转换的`Streams`的创建，并使用`from2-array`从一个对象数组中创建可读的`Streams`。
接下来，我们可以定义`concatFiles()`函数：

```javascript
function concatFiles(destination, files, callback) {
  const destStream = fs.createWriteStream(destination);
  fromArray.obj(files)             //[1]
    .pipe(through.obj((file, enc, done) => {   //[2]
      const src = fs.createReadStream(file);
      src.pipe(destStream, {end: false});
      src.on('end', done); //[3]
    }))
    .on('finish', () => {         //[4]
      destStream.end();
      callback();
    });
}

module.exports = concatFiles;
```

前面的函数通过将`files`数组转换为`Streams`来实现对`files`数组的顺序迭代。 该函数所遵循的程序解释如下：

1. 首先，我们使用`from2-array`从`files`数组创建一个可读的`Streams`。
2. 接下来，我们使用`through`来创建一个转换的`Streams`来处理序列中的每个文件。对于每个文件，我们创建一个可读的`Streams`，并通过管道将其输入到表示输出文件的`destStream`中。 在源文件完成读取后，通过在`pipe()`方法的第二个参数中指定`{end：false}`，我们确保不关闭`destStream`。
3. 当源文件的所有内容都被传送到`destStream`时，我们调用`through.obj`公开的`done`函数来传递当前处理已经完成，在我们的情况下这是需要触发处理下一个文件。
4. 所有文件处理完后，`finish`事件被触发。我们最后可以结束`destStream`并调用`concatFiles()`的`callback()`函数，这个函数表示整个操作的完成。

我们现在可以尝试使用我们刚刚创建的小模块。让我们创建一个名为`concat.js`的新文件来完成一个示例：

```javascript
const concatFiles = require('./concatFiles');

concatFiles(process.argv[2], process.argv.slice(3), () => {
  console.log('Files concatenated successfully');
});
```

我们现在可以运行上述程序，将目标文件作为第一个命令行参数，接着是要连接的文件列表，例如：

```bash
node concat allTogether.txt file1.txt file2.txt
```

执行这一条命令，会创建一个名为`allTogether.txt`的新文件，其中按顺序保存`file1.txt`和`file2.txt`的内容。

使用`concatFiles()`函数，我们能够仅使用`Streams`实现异步操作的顺序执行。正如我们在`Chapter3 Asynchronous Control Flow Patters with Callbacks`中看到的那样，如果使用纯`JavaScript`实现，或者使用`async`等外部库，则需要使用或实现迭代器。我们现在提供了另外一个可以达到同样效果的方法，正如我们所看到的，它的实现方式非常优雅且可读性高。

> 模式：使用Streams或Streams的组合，可以轻松地按顺序遍历一组异步任务。

#### 无序并行执行
我们刚刚看到`Streams`按顺序处理每个数据块，但有时这可能并不能这么做，因为这样并没有充分利用`Node.js`的并发性。如果我们必须对每个数据块执行一个缓慢的异步操作，那么并行化执行这一组异步任务完全是有必要的。当然，只有在每个数据块之间没有关系的情况下才能应用这种模式，这些数据块可能经常发生在对象模式的`Streams`中，但是对于二进制模式的`Streams`很少使用无序的并行执行。

> 注意：当处理数据的顺序很重要时，不能使用无序并行执行的Streams。

为了并行化一个可转换的`Streams`的执行，我们可以运用`Chapter3 Asynchronous Control Flow Patters with Callbacks`所讲到的无序并行执行的相同模式，然后做出一些改变使它们适用于`Streams`。让我们看看这是如何更改的。

##### 实现一个无序并行的Streams
让我们用一个例子直接说明：我们创建一个叫做`parallelStream.js`的模块，然后自定义一个普通的可转换的`Streams`，然后给出一系列可转换流的方法：

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

我们来分析一下这个新的自定义的类。正如你所看到的一样，构造函数接受一个`userTransform()`函数作为参数，然后将其另存为一个实例变量；我们也调用父构造函数，并且我们默认启用对象模式。

接下来，来看`_transform()`方法，在这个方法中，我们执行`userTransform()`函数，然后增加当前正在运行的任务个数; 最后，我们通过调用`done()`来通知当前转换步骤已经完成。`_transform()`方法展示了如何并行处理另一项任务。我们不用等待`userTransform()`方法执行完毕再调用`done()`。 相反，我们立即执行`done()`方法。另一方面，我们提供了一个特殊的回调函数给`userTransform()`方法，这就是`this._onComplete()`方法；以便我们在`userTransform()`完成的时候收到通知。

在`Streams`终止之前，会调用`_flush()`方法，所以如果仍有任务正在运行，我们可以通过不立即调用`done()`回调函数来延迟`finish`事件的触发。相反，我们将其分配给`this.terminateCallback`变量。为了理解`Streams`如何正确终止，来看`_onComplete()`方法。

在每组异步任务最终完成时，`_onComplete()`方法会被调用。首先，它会检查是否有任务正在运行，如果没有，则调用`this.terminateCallback()`函数，这将导致`Streams`结束，触发`_flush()`方法的`finish`事件。

利用刚刚构建的`ParallelStream`类可以轻松地创建一个无序并行执行的可转换的`Streams`实例，但是有个注意：它不会保留项目接收的顺序。实际上，异步操作可以在任何时候都有可能完成并推送数据，而跟它们开始的时刻并没有必然的联系。因此我们知道，对于二进制模式的`Streams`并不适用，因为二进制的`Streams`对顺序要求较高。

##### 实现一个URL监控应用程序
现在，让我们使用`ParallelStream`模块实现一个具体的例子。让我们想象以下我们想要构建一个简单的服务来监控一个大`URL`列表的状态，让我们想象以下，所有的这些`URL`包含在一个单独的文件中，并且每一个`URL`占据一个空行。

`Streams`能够为这个场景提供一个高效且优雅的解决方案。特别是当我们使用我们刚刚写的`ParallelStream`类来无序地审核这些`URL`。

接下来，让我们创建一个简单的放在`checkUrls.js`模块的应用程序。

```javascript
const fs = require('fs');
const split = require('split');
const request = require('request');
const ParallelStream = require('./parallelStream');

fs.createReadStream(process.argv[2])         //[1]
  .pipe(split())                             //[2]
  .pipe(new ParallelStream((url, enc, done, push) => {     //[3]
    if(!url) return done();
    request.head(url, (err, response) => {
      push(url + ' is ' + (err ? 'down' : 'up') + '\n');
      done();
    });
  }))
  .pipe(fs.createWriteStream('results.txt'))   //[4]
  .on('finish', () => console.log('All urls were checked'))
;
```

正如我们所看到的，通过流，我们的代码看起来非常优雅，直观。 让我们看看它是如何工作的：

1. 首先，我们通过给定的文件参数创建一个可读的`Streams`，便于接下来读取文件。
2. 我们通过[split](https://npmjs.org/package/split)将输入的文件的`Streams`的内容输出一个可转换的`Streams`到管道中，并且分隔了数据块的每一行。
3. 然后，是时候使用我们的`ParallelStream`来检查`URL`了，我们发送一个`HEAD`请求然后等待请求的`response`。当请求返回时，我们把请求的结果`push`到`stream`中。
4. 最后，通过管道把结果保存到`results.txt`文件中。

```bash
node checkUrls urlList.txt
```

这里的文件`urlList.txt`包含一组`URL`，例如：

* `http://www.mariocasciaro.me/`
* `http://loige.co/`
* `http://thiswillbedownforsure.com/`

当应用执行完成后，我们可以看到一个文件`results.txt`被创建，里面包含有操作的结果，例如：

* `http://thiswillbedownforsure.com is down`
* `http://loige.co is up`
* `http://www.mariocasciaro.me is up`

输出的结果的顺序很有可能与输入文件中指定`URL`的顺序不同。这是`Streams`无序并行执行任务的明显特征。

> 出于好奇，我们可能想尝试用一个正常的through2流替换ParallelStream，并比较两者的行为和性能（你可能想这样做的一个练习）。我们将会看到，使用through2的方式会比较慢，因为每个URL都将按顺序进行检查，而且文件results.txt中结果的顺序也会被保留。

#### 无序限制并行执行
如果运行包含数千或数百万个URL的文件的`checkUrls`应用程序，我们肯定会遇到麻烦。我们的应用程序将同时创建不受控制的连接数量，并行发送大量数据，并可能破坏应用程序的稳定性和整个系统的可用性。我们已经知道，控制负载的无序限制并行执行是一个极好的解决方案。

让我们通过创建一个`limitedParallelStream.js`模块来看看它是如何工作的，这个模块是改编自上一节中创建的`parallelStream.js`模块。

让我们看看它的构造函数：

```javascript
class LimitedParallelStream extends stream.Transform {
  constructor(concurrency, userTransform) {
    super({objectMode: true});
    this.concurrency = concurrency;
    this.userTransform = userTransform;
    this.running = 0;
    this.terminateCallback = null;
    this.continueCallback = null;
  }
// ...
}
```


我们需要一个`concurrency`变量作为输入来限制并发量，这次我们要保存两个回调函数，`continueCallback`用于任何挂起的`_transform`方法，`terminateCallback`用于_flush方法的回调。
接下来看`_transform()`方法：

```javascript
_transform(chunk, enc, done) {
  this.running++;
  this.userTransform(chunk, enc,  this.push.bind(this), this._onComplete.bind(this));
  if(this.running < this.concurrency) {
    done();
  } else {
    this.continueCallback = done;
  }
}
```

这次在`_transform()`方法中，我们必须在调用`done()`之前检查是否达到了最大并行数量的限制，如果没有达到了限制，才能触发下一个项目的处理。如果我们已经达到最大并行数量的限制，我们可以简单地将`done()`回调保存到`continueCallback`变量中，以便在任务完成后立即调用它。

`_flush()`方法与`ParallelStream`类保持完全一样，所以我们直接转到实现`_onComplete()`方法：

```javascript
_onComplete(err) {
  this.running--;
  if(err) {
    return this.emit('error', err);
  }
  const tmpCallback = this.continueCallback;
  this.continueCallback = null;
  tmpCallback && tmpCallback();
  if(this.running === 0) {
    this.terminateCallback && this.terminateCallback();
  }
}
```

每当任务完成，我们调用任何已保存的`continueCallback()`将导致
`stream`解锁，触发下一个项目的处理。

这就是`limitedParallelStream`模块。 我们现在可以在`checkUrls`模块中使用它来代替`parallelStream`，并且将我们的任务的并发限制在我们设置的值上。

#### 顺序并行执行
我们以前创建的并行`Streams`可能会使得数据的顺序混乱，但是在某些情况下这是不可接受的。有时，实际上，有那种需要每个数据块都以接收到的相同顺序发出的业务场景。我们仍然可以并行运行`transform`函数。我们所要做的就是对每个任务发出的数据进行排序，使其遵循与接收数据相同的顺序。

这种技术涉及使用`buffer`，在每个正在运行的任务发出时重新排序块。为简洁起见，我们不打算提供这样一个`stream`的实现，因为这本书的范围是相当冗长的；我们要做的就是重用为了这个特定目的而构建的`npm`上的一个可用包，例如[through2-parallel](https://npmjs.org/package/through2-parallel)。

我们可以通过修改现有的`checkUrls`模块来快速检查一个有序的并行执行的行为。 假设我们希望我们的结果按照与输入文件中的`URL`相同的顺序编写。 我们可以使用通过`through2-parallel`来实现：

```javascript
const fs = require('fs');
const split = require('split');
const request = require('request');
const throughParallel = require('through2-parallel');

fs.createReadStream(process.argv[2])
  .pipe(split())
  .pipe(throughParallel.obj({concurrency: 2}, function (url, enc, done) {
    if(!url) return done();
    request.head(url, (err, response) => {
      this.push(url + ' is ' + (err ? 'down' : 'up') + '\n');
      done();
    });
  }))
  .pipe(fs.createWriteStream('results.txt'))
  .on('finish', () => console.log('All urls were checked'))
;
```

正如我们所看到的，`through2-parallel`的接口与`through2`的接口非常相似；唯一的不同是在`through2-parallel`还可以为我们提供的`transform`函数指定一个并发限制。如果我们尝试运行这个新版本的`checkUrls`，我们会看到`results.txt`文件列出结果的顺序与输入文件中
URLs的出现顺序是一样的。

通过这个，我们总结了使用`Streams`实现异步控制流的分析；接下来，我们研究管道模式。

### 管道模式
就像在现实生活中一样，`Node.js`的`Streams`也可以按照不同的模式进行管道连接。事实上，我们可以将两个不同的`Streams`合并成一个`Streams`，将一个`Streams`分成两个或更多的管道，或者根据条件重定向流。 在本节中，我们将探讨可应用于`Node.js`的`Streams`最重要的管道技术。

#### 组合的Streams
在本章中，我们强调`Streams`提供了一个简单的基础结构来模块化和重用我们的代码，但是却漏掉了一个重要的部分：如果我们想要模块化和重用整个流水线？如果我们想要合并多个`Streams`，使它们看起来像外部的`Streams`，那该怎么办？下图显示了这是什么意思：

![](http://oczira72b.bkt.clouddn.com/17-12-9/23434927.jpg)

从上图中，我们看到了如何组合几个流的了：

* 当我们写入组合的`Streams`的时候，实际上我们是写入组合的`Streams`的第一个单元，即`StreamA`。
* 当我们从组合的`Streams`中读取信息时，实际上我们从组合的`Streams`的最后一个单元中读取。

一个组合的`Streams`通常是一个多重的`Streams`，通过连接第一个单元的写入端和连接最后一个单元的读取端。

> 要从两个不同的Streams（一个可读的Streams和一个可写入的Streams）中创建一个多重的Streams，我们可以使用一个npm模块，例如[duplexer2](https://npmjs.org/package/duplexer2)。

但上述这么做并不完整。实际上，组合的`Streams`还应该做到捕获到管道中任意一段`Streams`单元产生的错误。我们已经说过，任何错误都不会自动传播到管道中。 所以，我们必须有适当的错误管理，我们将不得不显式附加一个错误监听器到每个`Streams`。但是，组合的`Streams`实际上是一个黑盒，这意味着我们无法访问管道中间的任何单元，所以对于管道中任意单元的异常捕获，组合的`Streams`也充当聚合器的角色。

总而言之，组合的`Streams`具有两个主要优点：

* 管道内部是一个黑盒，对使用者不可见。
* 简化了错误管理，因为我们不必为管道中的每个单元附加一个错误侦听器，而只需要给组合的`Streams`自身附加上就可以了。

组合的`Streams`是一个非常通用和普遍的做法，所以如果我们没有任何特殊的需要，我们可能只想重用现有的解决方案，如[multipipe](https://www.npmjs.org/package/multipipe)或[combine-stream](https://www.npmjs.org/package/combine-stream)。

#### 实现一个组合的Streams
为了说明一个简单的例子，我们来考虑下面两个组合的`Streams`的情况：

* 压缩和加密数据
* 解压和解密数据

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

例如，我们现在可以使用这些组合的数据流，如同黑盒，这些对我们均是不可见的，可以创建一个小型应用程序，通过压缩和加密来归档文件。 让我们在一个名为`archive.js`的新模块中做这件事： 

```javascript
const fs = require('fs');
const compressAndEncryptStream = require('./combinedStreams').compressAndEncrypt;
fs.createReadStream(process.argv[3])
  .pipe(compressAndEncryptStream(process.argv[2]))
  .pipe(fs.createWriteStream(process.argv[3] + ".gz.enc"));
```

我们可以通过从我们创建的流水线中构建一个组合的`Stream`来进一步改进前面的代码，但这次并不只是为了获得对外不可见的黑盒，而是为了进行异常捕获。 实际上，正如我们已经提到过的那样，写下如下的代码只会捕获最后一个`Stream`单元发出的错误：

```javascript
fs.createReadStream(process.argv[3])
  .pipe(compressAndEncryptStream(process.argv[2]))
  .pipe(fs.createWriteStream(process.argv[3] + ".gz.enc"))
  .on('error', function(err) {
    // 只会捕获最后一个单元的错误
    console.log(err);
  });
```

但是，通过把所有的`Streams`结合在一起，我们可以优雅地解决这个问题。重构后的`archive.js`如下： 

```javascript
const combine = require('multipipe');
   const fs = require('fs');
   const compressAndEncryptStream =
     require('./combinedStreams').compressAndEncrypt;
   combine(
     fs.createReadStream(process.argv[3])
     .pipe(compressAndEncryptStream(process.argv[2]))
     .pipe(fs.createWriteStream(process.argv[3] + ".gz.enc"))
   ).on('error', err => {
     // 使用组合的Stream可以捕获任意位置的错误
     console.log(err);
   });
```

正如我们所看到的，我们现在可以将一个错误侦听器直接附加到组合的`Streams`，它将接收任何内部流发出的任何`error`事件。
现在，要运行`archive`模块，只需在命令行参数中指定`password`和`file`参数，即压缩模块的参数：

```bash
node archive mypassword /path/to/a/file.text
```

通过这个例子，我们已经清楚地证明了组合的`Stream`是多么重要; 从一个方面来说，它允许我们创建流的可重用组合，从另一方面来说，它简化了管道的错误管理。

### 分开的`Streams`
我们可以通过将单个可读的`Stream`管道化为多个可写入的`Stream`来执行`Stream`的分支。当我们想要将相同的数据发送到不同的目的地时，这便体现其作用了，例如，两个不同的套接字或两个不同的文件。当我们想要对相同的数据执行不同的转换时，或者当我们想要根据一些标准拆分数据时，也可以使用它。如图所示：

![](http://oczira72b.bkt.clouddn.com/17-12-31/44841803.jpg)

在`Node.js`中分开的`Stream`是一件小事。举例说明。

#### 实现一个多重校验和的生成器
让我们创建一个输出给定文件的`sha1`和`md5`散列的小工具。我们来调用这个新模块`generateHashes.js`，看如下的代码：

```javascript
const fs = require('fs');
const crypto = require('crypto');
const sha1Stream = crypto.createHash('sha1');
sha1Stream.setEncoding('base64');
const md5Stream = crypto.createHash('md5');
md5Stream.setEncoding('base64');
```

目前为止没什么特别的 该模块的下一个部分实际上是我们将从文件创建一个可读的`Stream`，并将其分叉到两个不同的流，以获得另外两个文件，其中一个包含`sha1`散列，另一个包含`md5`校验和：

```javascript
const inputFile = process.argv[2];
const inputStream = fs.createReadStream(inputFile);
inputStream
  .pipe(sha1Stream)
  .pipe(fs.createWriteStream(inputFile + '.sha1'));
inputStream
  .pipe(md5Stream)
  .pipe(fs.createWriteStream(inputFile + '.md5'));
```

这很简单：`inputStream`变量通过管道一边输入到`sha1Stream`，另一边输入到`md5Stream`。但是要注意：

* 当`inputStream`结束时，`md5Stream`和`sha1Stream`会自动结束，除非当调用`pipe()`时指定了`end`选项为`false`。

* `Stream`的两个分支会接受相同的数据块，因此当对数据执行一些副作用的操作时我们必须非常谨慎，因为那样会影响另外一个分支。

* 黑盒外会产生背压，来自`inputStream`的数据流的流速会根据接收最慢的分支的流速作出调整。

### 合并的`Streams`
合并与分开相对，通过把一组可读的`Streams`合并到一个单独的可写的`Stream`里，如图所示：

![](http://oczira72b.bkt.clouddn.com/17-12-31/93682580.jpg)

将多个`Streams`合并为一个通常是一个简单的操作; 然而，我们必须注意我们处理`end`事件的方式，因为使用自动结束选项的管道系统会在一个源结束时立即结束目标流。 这通常会导致错误，因为其他还未结束的源将继续写入已终止的`Stream`。 解决此问题的方法是在将多个源传输到单个目标时使用选项`{end：false}`，并且只有在所有源完成读取后才在目标`Stream`上调用`end()`。

#### 用多个源文件压缩为一个压缩包
举一个简单的例子，我们来实现一个小程序，它根据两个不同目录的内容创建一个压缩包。 为此，我们将介绍两个新的`npm`模块：

* [tar](https://www.npmjs.com/package/tar)用来创建压缩包
* [fstream](https://www.npmjs.com/package/fstream)从文件系统文件创建对象streams的库

我们创建一个新模块`mergeTar.js`，如下开始初始化：

```javascript
var tar = require('tar');
var fstream = require('fstream');
var path = require('path');
var destination = path.resolve(process.argv[2]);
var sourceA = path.resolve(process.argv[3]);
var sourceB = path.resolve(process.argv[4]);
```

在前面的代码中，我们只加载全部依赖包和初始化包含目标文件和两个源目录(`sourceA`和`sourceB`)的变量。

接下来，我们创建`tar`的`Stream`并通过管道输出到一个可写入的`Stream`：

```javascript
const pack = tar.Pack();
pack.pipe(fstream.Writer(destination));
```

现在，我们开始初始化源`Stream`

```javascript
let endCount = 0;

function onEnd() {
  if (++endCount === 2) {
    pack.end();
  }
}

const sourceStreamA = fstream.Reader({
    type: "Directory",
    path: sourceA
  })
  .on('end', onEnd);

const sourceStreamB = fstream.Reader({
    type: "Directory",
    path: sourceB
  })
  .on('end', onEnd);
```

在前面的代码中，我们创建了从两个源目录（`sourceStreamA`和`sourceStreamB`）中读取的`Stream`那么对于每个源`Stream`，我们附加一个`end`事件订阅者，只有当这两个目录被完全读取时，才会触发`pack`的`end`事件。

最后，合并两个`Stream`：

```javascript
sourceStreamA.pipe(pack, {end: false});
sourceStreamB.pipe(pack, {end: false});
```

我们将两个源文件都压缩到`pack`这个`Stream`中，并通过设定`pipe()`的`option`参数为`{end：false}`配置终点`Stream`的自动触发`end`事件。

这样，我们已经完成了我们简单的`TAR`程序。我们可以通过提供目标文件作为第一个命令行参数，然后是两个源目录来尝试运行这个实用程序：

```javascript
node mergeTar dest.tar /path/to/sourceA /path/to/sourceB
```

在`npm`中我们可以找到一些可以简化`Stream`的合并的模块：

* [merge-stream](https://www.npmjs.com/package/merge-stream)
* [multistream-merge](https://www.npmjs.com/package/multistream-merge)

要注意，流入目标`Stream`的数据是随机混合的，这是一个在某些类型的对象流中可以接受的属性（正如我们在上一个例子中看到的那样），但是在处理二进制`Stream`时通常是一个不希望这样。

然而，我们可以通过一种模式按顺序合并`Stream`; 它包含一个接一个地合并源`Stream`，当前一个结束时，开始发送第二段数据块（就像连接所有源`Stream`的输出一样）。在`npm`上，我们可以找到一些也处理这种情况的软件包。其中之一是[multistream](https://npmjs.org/package/multistream)。

### 多路复用和多路分解
合并`Stream`模式有一个特殊的模式，我们并不是真的只想将多个`Stream`合并在一起，而是使用一个共享通道来传送一组数据`Stream`。与之前的不一样，因为源数据`Stream`在共享通道内保持逻辑分离，这使得一旦数据到达共享通道的另一端，我们就可以再次分离数据`Stream`。如图所示：

![](http://oczira72b.bkt.clouddn.com/18-1-1/35586096.jpg)

将多个`Stream`组合在单个`Stream`上传输的操作被称为多路复用，而相反的操作（即，从共享`Stream`接收数据重构原始的`Stream`）则被称为多路分用。执行这些操作的设备分别称为多路复用器和多路分解器（。 这是一个在计算机科学和电信领域广泛研究的话题，因为它是几乎任何类型的通信媒体，如电话，广播，电视，当然还有互联网本身的基础之一。 对于本书的范围，我们不会过多解释，因为这是一个很大的话题。

我们想在本节中演示的是，如何使用共享的`Node.js Streams`来传送多个逻辑上分离的`Stream`，然后在共享`Stream`的另一端再次分离，即实现一次多路复用和多路分解。

#### 创建一个远程logger日志记录
举例说明，我们希望有一个小程序来启动子进程，并将其标准输出和标准错误都重定向到远程服务器，服务器接受它们然后保存为两个单独的文件。因此，在这种情况下，共享介质是`TCP`连接，而要复用的两个通道是子进程的`stdout`和`stderr`。 我们将利用分组交换的技术，这种技术与`IP`，`TCP`或`UDP`等协议所使用的技术相同，包括将数据封装在数据包中，允许我们指定各种源信息，这对多路复用，路由，控制 流程，检查损坏的数据都十分有帮助。

如图所示，这个例子的协议大概是这样，数据被封装成具有以下结构的数据包：

![](http://oczira72b.bkt.clouddn.com/18-1-1/44372113.jpg)

##### 在客户端实现多路复用
先说客户端，创建一个名为`client.js`的模块，这是我们这个应用程序的一部分，它负责启动一个子进程并实现`Stream`多路复用。

开始定义模块，首先加载依赖：

```javascript
const child_process = require('child_process');
const net = require('net');
```

然后开始实现多路复用的函数：

```javascript
function multiplexChannels(sources, destination) {
  let totalChannels = sources.length;

  for(let i = 0; i < sources.length; i++) {
    sources[i]
      .on('readable', function() { // [1]
        let chunk;
        while ((chunk = this.read()) !== null) {
          const outBuff = new Buffer(1 + 4 + chunk.length); // [2]
          outBuff.writeUInt8(i, 0);
          outBuff.writeUInt32BE(chunk.length, 1);
          chunk.copy(outBuff, 5);
          console.log('Sending packet to channel: ' + i);
          destination.write(outBuff); // [3]
        }
      })
      .on('end', () => { //[4]
        if (--totalChannels === 0) {
          destination.end();
        }
      });
  }
}
```

`multiplexChannels()`函数接受要复用的源`Stream`作为输入
和复用接口作为参数，然后执行以下步骤：

1. 对于每个源`Stream`，它会注册一个`readable`事件侦听器，我们使用`non-flowing`模式从流中读取数据。

2. 每读取一个数据块，我们将其封装到一个首部中，首部的顺序为：`channel ID`为1字节（`UInt8`），数据包大小为4字节（`UInt32BE`），然后为实际数据。

3. 数据包准备好后，我们将其写入目标`Stream`。

4. 我们为`end`事件注册一个监听器，以便当所有源`Stream`结束时，`end`事件触发，通知目标`Stream`触发`end`事件。

> 注意，我们的协议最多能够复用多达256个不同的源流，因为我们只有1个字节来标识`channel`。

```javascript
const socket = net.connect(3000, () => { // [1]
  const child = child_process.fork( // [2]
    process.argv[2],
    process.argv.slice(3), {
      silent: true
    }
  );
  multiplexChannels([child.stdout, child.stderr], socket); // [3]
});
```

在最后，我们执行以下操作：

1. 我们创建一个新的`TCP`客户端连接到地址`localhost:3000`。
2. 我们通过使用第一个命令行参数作为路径来启动子进程，同时我们提供剩余的`process.argv`数组作为子进程的参数。我们指定选项`{silent：true}`，以便子进程不会继承父级的`stdout`和`stderr`。
3. 我们使用`mutiplexChannels()`函数将`stdout`和`stderr`多路复用到`socket`里。

##### 在服务端实现多路分解
现在来看服务端，创建`server.js`模块，在这里我们将来自远程连接的`Stream`多路分解，并将它们传送到两个不同的文件中。

首先创建一个名为`demultiplexChannel()`的函数：

```javascript
function demultiplexChannel(source, destinations) {
  let currentChannel = null;
  let currentLength = null;
  source
    .on('readable', () => { //[1]
      let chunk;
      if(currentChannel === null) {          //[2]
        chunk = source.read(1);
        currentChannel = chunk && chunk.readUInt8(0);
      }
    
      if(currentLength === null) {          //[3]
        chunk = source.read(4);
        currentLength = chunk && chunk.readUInt32BE(0);
        if(currentLength === null) {
          return;
        }
      }
    
      chunk = source.read(currentLength);        //[4]
      if(chunk === null) {
        return;
      }
    
      console.log('Received packet from: ' + currentChannel);
    
      destinations[currentChannel].write(chunk);      //[5]
      currentChannel = null;
      currentLength = null;
    })
    .on('end', () => {            //[6]
      destinations.forEach(destination => destination.end());
      console.log('Source channel closed');
    })
  ;
}
```

上面的代码可能看起来很复杂，仔细阅读并非如此；由于`Node.js`可读的`Stream`的拉动特性，我们可以很容易地实现我们的小协议的多路分解，如下所示：
1. 我们开始使用`non-flowing`模式从流中读取数据。
2. 首先，如果我们还没有读取`channel ID`，我们尝试从流中读取1个字节，然后将其转换为数字。
3. 下一步是读取首部的长度。我们需要读取4个字节，所以有可能在内部`Buffer`还没有足够的数据，这将导致`this.read()`调用返回`null`。在这种情况下，我们只是中断解析，然后重试下一个`readable`事件。
4. 当我们最终还可以读取数据大小时，我们知道从内部`Buffer`中拉出多少数据，所以我们尝试读取所有数据。
5. 当我们读取所有的数据时，我们可以把它写到正确的目标通道，一定要记得重置`currentChannel`和`currentLength`变量（这些变量将被用来解析下一个数据包）。
6. 最后，当源`channel`结束时，一定不要忘记调用目标`Stream`的`end()`方法。

既然我们可以多路分解源`Stream`，进行如下调用：

```javascript
net.createServer(socket => {
  const stdoutStream = fs.createWriteStream('stdout.log');
  const stderrStream = fs.createWriteStream('stderr.log');
  demultiplexChannel(socket, [stdoutStream, stderrStream]);
})
  .listen(3000, () => console.log('Server started'))
;
```

在上面的代码中，我们首先在`3000`端口上启动一个`TCP`服务器，然后对于我们接收到的每个连接，我们将创建两个可写入的`Stream`，指向两个不同的文件，一个用于标准输出，另一个用于标准错误; 这些是我们的目标`channel`。 最后，我们使用`demultiplexChannel()`将套接字流解复用为`stdoutStream`和`stderrStream`。

##### 运行多路复用和多路分解应用程序
现在，我们准备尝试运行我们的新的多路复用/多路分解应用程序，但首先让我们创建一个小的`Node.js`程序来产生一些示例输出; 我们把它叫做`generateData.js`：

```javascript
console.log("out1");
console.log("out2");
console.error("err1");
console.log("out3");
console.error("err2");
```

首先，让我们开始运行服务端：

```bash
node server
```

然后运行客户端，需要提供作为子进程的文件参数：

```bash
node client generateData.js
```

![](http://oczira72b.bkt.clouddn.com/18-1-1/50338550.jpg)

客户端几乎立马运行，但是进程结束时，`generateData`应用程序的标准输入和标准输出经过一个`TCP`连接，然后在服务器端，被多路分解成两个文件。

> 注意，当我们使用`child_process.fork()`时，我们的客户端能够启动别的`Node.js`模块。

#### 对象Streams的多路复用和多路分解
我们刚刚展示的例子演示了如何复用和解复用二进制/文本`Stream`，但值得一提的是，相同的规则也适用于对象`Stream`。 最大的区别是，使用对象，我们已经有了使用原子消息（对象）传输数据的方法，所以多路复用就像设置一个属性`channel ID`到每个对象一样简单，而多路分解只需要读`·channel ID`属性，并将每个对象路由到正确的目标`Stream`。

还有一种模式是取一个对象上的几个属性并分发到多个目的`Stream`的模式 通过这种模式，我们可以实现复杂的流程，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-1-1/38782926.jpg)

如上图所示，取一个对象`Stream`表示`animals`，然后根据动物类型：`reptiles`，`amphibians`和`mammals`，然后分发到正确的目标`Stream`中。

## 总结
在本章中，我们已经对`Node.js Streams`及其使用案例进行了阐述，但同时也应该为编程范式打开一扇大门，几乎具有无限的可能性。我们了解了为什么`Stream`被`Node.js`社区赞誉，并且我们掌握了它们的基本功能，使我们能够利用它做更多有趣的事情。我们分析了一些先进的模式，并开始了解如何将不同配置的`Streams`连接在一起，掌握这些特性，从而使流如此多才多艺，功能强大。

如果我们遇到不能用一个`Stream`来实现的功能，我们可以通过将其他`Streams`连接在一起来实现，这是`Node.js`的一个很好的特性；`Streams`在处理二进制数据，字符串和对象都十分有用，并具有鲜明的特点。

在下一章中，我们将重点介绍传统的面向对象的设计模式。尽管`JavaScript`在某种程度上是面向对象的语言，但在`Node.js`中，函数式或混合方法通常是首选。在阅读下一章便揭晓答案。