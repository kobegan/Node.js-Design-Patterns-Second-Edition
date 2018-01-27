# Design Patterns
设计模式是重复出现的问题的可重用解决方案；该术语的定义非常广泛，可以涵盖应用程序的多个领域。然而，这个术语通常与著名的面向对象模式相关联，又被称作可复用的面向对象基础方法。我们经常会将这些特定的模式集合称为传统设计模式或`GoF`设计模式。

在`JavaScript`中应用面向对象的设计模式并不像传统的面向对象的语言那样线性和形式化。我们知道，`JavaScript`是范式化的，面向对象的，基于原型的，并且是动态类型语言；它将函数视为一等公民，并允许函数式的编程风格。这些特性使得`JavaScript`成为一种非常通用的语言，它为开发人员提供了巨大的力量，但同时也造成其编程风格与传统语言不同。人们总结`JavaScript`的编程范式，最后总结出`JavaScript`生态系统的模式。有很多方法可以使用`JavaScript`实现相同的结果。对于`JavaScript`的问题，解决一个问题的模式是多样化的。这种现象的一个明显的例子就是`JavaScript`生态系统中有丰富的框架和类库；可能没有其他语言见过这么多，尤其是现在`Node.js`已经给`JavaScript`带来了惊人的新的可能性，并创造了许多新的场景。

在这种背景下，传统的设计模式也受到`JavaScript`本质的影响。实现它们的方式有很多，所以它们传统的，强烈的面向对象的实现意味着它们不再是模式。在某些情况下，它们甚至是不需要的，因为我们知道，`JavaScript`没有真正的类或抽象接口。不变的是每个模式的基本原理，解决的问题以及解决方案核心的概念。

本章探讨的设计模式如下：

* 工厂模式（`Factory`）
* 揭示构造模式（`Revealing constructor`）
* 代理模式（`Proxy`）
* 装饰者模式（`Decorator`）
* 适配器模式（`Adapter`）
* 策略模式（`Strategy`）
* 状态模式（`State`）
* 模板模式（`Template`）
* 中间件模式（`Middleware`）
* 命令模式（`Command`）

> 本章假定读者对`JavaScript`中继承的工作原理有一些概念。 另外请注意，在本章中，我们经常使用一般的和更直观的图来描述一个模式来代替标准的`UML`，因为许多模式可以有一个不仅基于类而且基于对象甚至函数的实现。

## 工厂模式（`Factory`）
我们从`Node.js`中最简单，最常见的设计模式工厂模式开始。

### 用于创建对象的通用接口
我们已经强调了这样的事实：在`JavaScript`中，因为函数的简单性，易用性和可拓展性，函数实例通常比纯粹的面向对象设计更受欢迎。创建新的对象实例时尤其如此。 实际上，调用一个工厂，而不是直接使用`new`运算符或`Object.create()`从一个原型创建一个新的对象，在很多方面是非常方便和灵活的。

首先，工厂允许我们将对象创建与实现分离开来；从本质上讲，一个工厂包装了一个新实例的创建，给了我们更多的灵活性和控制。 在工厂内部，我们可以使用闭包，使用原型和`new`运算符，使用`Object.create()`创建新实例，甚至根据特定条件返回不同的实例。对于对象的使用者而言，其完全不知道这个实例是怎么进行创建的。事实是，通过使用`new`，我们将我们的代码绑定到创建对象的一种特定方式，而在`JavaScript`中，可以更灵活且自由地创建对象。作为下面这个简单的例子，我们来考虑通过工厂模式创建一个`Image`对象：

```javascript
function createImage(name) {
  return new Image(name);
}
const image = createImage('photo.jpeg');
```

`createImage()`工厂可能看起来完全没有必要。为什么不直接使用`new`运算符来实例化`Image`类？像下面这行代码：

```javascript
const image = new Image(name);
```

正如我们已经提到的，使用`new`将我们的代码绑定到一个特定类型的对象；对于前面的例子，绑定到`Image`类型的对象。工厂模式创建对象更为灵活；想象一下，如果我们想要重构`Image`类，把它分成更小的类，使得其支持各种图像格式。如果我们将工厂作为创建新图像的唯一方法，我们可以像如下拓展代码，而不会破坏任何现有的代码：

```javascript
function createImage(name) {
  if (name.match(/\.jpeg$/)) {
    return new JpegImage(name);
  } else if (name.match(/\.gif$/)) {
    return new GifImage(name);
  } else if (name.match(/\.png$/)) {
    return new PngImage(name);
  } else {
    throw new Exception('Unsupported format');
  }
}
```

工厂还允许我们不暴露它创建的对象的构造函数，并防止它们被扩展或修改。 在`Node.js`中，这可以通过仅导出工厂来实现，同时保持每个构造函数都是私有的。

### 强制封装机制
由于闭包，工厂也可以用来实现封装。

正如我们所知，在`JavaScript`中，我们没有权限修饰符（例如，我们不能声明私有变量），所以强制封装的唯一方法是通过函数作用域和闭包。 工厂可以用来实现封装，直接声明私有变量；以下面的代码为例：

```javascript
function createPerson(name) {
  const privateProperties = {};
  const person = {
    setName: name => {
      if (!name) throw new Error('A person must have a name');
      privateProperties.name = name;
    },
    getName: () => {
      return privateProperties.name;
    }
  };
  person.setName(name);
  return person;
}
```

在前面的代码中，我们利用闭包来创建两个对象：一个表示工厂返回的公共接口的`person`对象，一个从外部不可访问的`privateProperties`，只能通过`person`提供的接口来操作目的。例如，在前面的代码中，要确保`person`的`name`永远不为空；如`name`只是`person`对象的属性，则不可能做到强制封装。

工厂只是我们创建私有成员变量的技术之一，事实上，也有很多其它的方法定义私有成员变量：

* 在构造函数中定义私有变量
* 使用约定，用下划线`_`或美元符号`$`（但这在技术上不会阻止从外部访问成员）的属性名称前缀
* 使用`ES2015 WeakMaps`

### 构建一个简单的profiler
现在，我们来看一个使用工厂模式的完整示例。让我们构建一个简单的`profiler`，看一个具有以下属性的对象：

* `start()`方法，触发一个会话开始
* `end()`方法，终止会话并记录它的执行时间，打印到控制台

我们首先创建一个名为`profiler.js`的文件，它将包含以下内容：

```javascript
class Profiler {
  constructor(label) {
    this.label = label;
    this.lastTime = null;
  }
  start() {
    this.lastTime = process.hrtime();
  }
  end() {
    const diff = process.hrtime(this.lastTime);
    console.log(
      `Timer "${this.label}" took ${diff[0]} seconds and ${diff[1]}
           nanoseconds.`
    );
  }
}
```

前面的类没有什么特别之处。我们只需使用默认的定时器来保存当`start()`被调用时的时间，然后计算到执行`end()`时的所经过的时间，并将结果打印到控制台。

现在，如果我们要在真实世界的应用程序中使用这样一个`profiler`来计算不同程序的执行时间，我们可以很容易想象我们将会在标准输出中产生大量的日志记录，特别是在生产环境中。我们可能想要做的是将分析信息重定向到另一个源（例如数据库），或者，如果应用程序正在生产环境下运行，则将`profiler`完全禁用。很明显，如果我们直接使用`new`运算符实例化一个`Profiler`对象，那么我们需要在客户端代码或`Profiler`对象本身中添加一些额外的逻辑，以便在不同的逻辑之间切换。我们可以使用工模式厂来抽象创建`Profiler`对象，这样，根据应用程序是以生产模式还是开发模式运行，我们可以返回完全正常工作的`Profiler`对象，或者具有相同接口的模拟对象，但方法是空函数。让我们在`profiler.js`模块中执行此操作，而不是导出`Profiler`构造函数，而只导出一个函数，即我们的工厂。以下是其代码：

```javascript
module.exports = function(label) {
  if (process.env.NODE_ENV === 'development') {
    return new Profiler(label); // [1]
  } else if (process.env.NODE_ENV === 'production') {
    return { // [2]
      start: function() {},
      end: function() {}
    }
  } else {
    throw new Error('Must set NODE_ENV');
  }
};
```

我们创建的工厂从其中抽象了`Profiler`对象的创建过程：

* 如果应用程序正在开发模式下运行，我们会完全返回一个新的具有完整功能的`Profiler`对象。
* 如果应用程序正在生产模式下运行，则返回一个模拟对象，它的`start()`和`stop()`方法是空函数。

值得一提的是，由于`JavaScript`的动态输入，我们能够在一种情况下返回一个使用`new`运算符实例化的对象，而在另一种情况下返回一个简单的对象字面值。工厂模式可以很好地实现这一点，我们可以在工厂函数中以任何方式创建对象，可以执行额外的初始化步骤或者根据特定的条件返回不同类型的对象，而这些细节对于对象的使用者来说都是透明的。我们可以很容易地理解这种简单模式的强大。

现在我们可以使用我们的`profiler`，来看以下代码：

```javascript
const profiler = require('./profiler');

function getRandomArray(len) {
  const p = profiler('Generating a ' + len + ' items long array');
  p.start();
  const arr = [];
  for (let i = 0; i < len; i++) {
    arr.push(Math.random());
  }
  p.end();
}
getRandomArray(1e6);
console.log('Done');
```

变量`p`包含我们的`profiler`对象实例，但是我们不知道它是
如何创建的，和在这个代码点它是如何实现的。如果我们将上面的代
码包含在`profilerTest.js`中，我们可以很容易地测验证这些假设。测试启用代码分析功能的程序，运行以下命令：

```bash
export NODE_ENV=development; node profilerTest
```

前面的命令启用开发环境的`profiler`然后打印分析信息到控制台。
如果我们想要看看生产环境下的`profiler`，我们可以运行下面的命令：

```bash
export NODE_ENV=production; node profilerTest
```

我们刚才展示的示例只是工厂模式的简单应用程序，但它清楚地显示了将对象的创建与实现分离的优点。

### 可组合的工厂函数
现在我们对如何在`Node.js`中实现工厂函数有了一个很好的想法，我们准备引入一个最近在`JavaScript`社区中引起了关注的高级模式。我们正在谈论可组合的工厂函数，它代表了一种特定类型的工厂函数，可以“组合”在一起构建新的更强大的工厂函数。它们允​​许我们构建继承关系较为复杂的对象十分有用。

我们可以用一个简单而有效的例子来阐明这个概念。假设我们要构建一个游戏，其中屏幕上的角色可以有许多不同的行为：可以在屏幕上移动;他们可以砍杀和射击。是的，要成为一个角色，他们应该有一些基本的属性，如生命值，屏幕上的位置和角色类型。

我们要定义几种类型的角色，每一种特定的行为：

* `Character`：具有生命值，位置和名字的基础角色
* `Mover`：可移动的角色
* `Slasher`：可砍杀他人的角色
* `Shooter`：能够射击的角色（只要有子弹就可以成为`Shooter`！）

理想情况下，我们可以定义新的角色类型，结合现有角色的不同行为。 我们希望有绝对的自由，例如，我们希望在现有的基础上定义这些新的类型：

* `Runner`：可移动的角色
* `Samurai`：可移动和砍杀他人的角色
* `Sniper`：不能移动但能射击的角色
* `Gunslinger`：可以移动和射击的角色
* `Western Samurai`：可移动、砍杀他人和射击的角色

正如你所看到的，我们希望完全自由地结合每个基本类型的特征，所以现在应该很明显的是我们不能用类和继承来简单地模拟这个问题。

相反，我们将使用可组合的工厂函数，特别是我们可以使用[stamp](https://www.npmjs.com/package/stampit)模块。

这个模块提供了一个直观的接口来定义工厂函数，可以组合起来构建新的工厂函数。基本上，它允许我们定义工厂函数，通过使用方便流畅的接口来描述它们，这些工厂函数将生成具有一组特定属性和方法的对象。

让我们看看如何通过`stamp`定义我们的游戏的基本角色。我们将从基础的角色开始：

```javascript
const stampit = require('stampit');
const character = stampit().
props({
  name: 'anonymous',
  lifePoints: 100,
  x: 0,
  y: 0
});
```

在前面的代码片段中，我们定义了角色的工厂函数，它可以用来创建基本角色的新实例。每个角色将具有以下属性：name，lifePoints，x和y，默认值分别为`'anonymous'`，`100`，`0`和`0`。使用`stampit`的`props`方法可以定义这些属性。 要使用这个工厂函数，我们可以这样做：

```javascript
const c = character();
c.name = 'John';
c.lifePoints = 10;
console.log(c); // { name: 'John', lifePoints: 10, x:0, y:0 }
```

现在，让我们来定义`mover`工厂函数：

```javascript
const mover = stampit()
  .methods({
    move(xIncr, yIncr) {
      this.x += xIncr;
      this.y += yIncr;
      console.log(`${this.name} moved to [${this.x}, ${this.y}]`);
    }
  });
```

在这种情况下，我们使用`stampit`的`methods`函数来声明这个工厂函数产生的对象中所有可用的方法。 对于我们的`Mover`定义，我们有一个`move`函数可以增加实例的`x`和`y`的位置。 请注意，我们可以从方法内使用关键字`this`来访问实例属性。

现在我们已经理解了基本的概念，我们可以很容易地添加`slasher`和`shooter`类型的工厂函数定义：

```javascript
const slasher = stampit()
  .methods({
    slash(direction) {
      console.log(`${this.name} slashed to the ${direction}`);
    }
  });
const shooter = stampit()
  .props({
    bullets: 6
  })
  .methods({
    shoot(direction) {
      if (this.bullets > 0) {
        --this.bullets;
        console.log(`${this.name} shoot to the ${direction}`);
      }
    }
  });
```

注意到我们如何使用`props`和`methods`来定义我们的`shooter`工厂函数。

现在我们已经定义了所有的基本类型，我们准备将它们组合起来创建新的更为复杂的工厂函数。

```javascript
const runner = stampit.compose(character, mover);
const samurai = stampit.compose(character, mover, slasher);
const sniper = stampit.compose(character, shooter);
const gunslinger = stampit.compose(character, mover, shooter);
const westernSamurai = stampit.compose(gunslinger, samurai);
```

`stampit.compose()`方法定义了一个新的组合的工厂函数，它的作用是根据组合工厂函数的方法和属性生成一个对象。 正如你所看到的那样，这是一个强大的机制，使我们能够自由地创建和组合工厂函数。

接下来我们实例化一个新的`westernSamurai`。

```javascript
const gojiro = westernSamurai();
gojiro.name = 'Gojiro Kiryu';
gojiro.move(1, 0);
gojiro.slash('left');
gojiro.shoot('right');
```

这将产生以下输出：

```
Yojimbo moved to [1, 0]
Yojimbo slashed to the left
Yojimbo shoot to the right
```

### 实际应用场景
正如我们所说的，工厂模式在`Node.js`中非常流行，许多软件包只提供用于创建新实例的工厂；常见一些例子如下：

* [Dnode](https://www.npmjs.com/package/dnode)：`Node.js`的远程程序调用（`RPC`）库。如果我们查看它的源代码，我们会看到它的逻辑实际上是实现成一个名为`D`的类；然而，实例并没有暴露给外界，因为唯一的接口是工厂，这使我们能够使用它创建类的新实例。你可以看看它的源代码。

* [Restify](https://npmjs.org/package/restify)：这是一个构建`REST API`的框架，它允许我们使用`restify.createServer()`工厂函数创建一个服务器的新实例，该工厂在内部创建一个新的实例`Server`类（不导出）。 你可以看看它的源代码。

其他模块公开了一个类和一个工厂，但将工厂作为创建新实例的主要方法或最方便的方法；一些例子如下：

* [http-proxy](https://npmjs.org/package/http-proxy)：这是一个可编程`HTTP`的代理库，用`httpProxy.createProxyServer(options)`创建新的实例。
* `Node.js`核心模块之`HTTP`：这是新实例主要使用`http.createServer()`创建的地方，但这实际上是`new http.Server()`的简写方式。
* [bunyan](https://npmjs.org/package/bunyan)：这是一个广泛使用的日志记录库；在其`README`文件中，要求这个仓库的`contributors`需要使用工厂函数`bunyan.createLogger()`作为创建新实例的主要方法，即使这相当于运行`new bunyan()`。

其他一些模块也提供了一个工厂函数来封装其组件实例的创建。常见的例子是`through2`和`from2`（我们在`Chapter 5-Coding with Streams`看到过它），它允许我们使用工厂方法简化新`Streams`的创建，从而显式地使用继承和`new`运算符。

还有一些使用`stamp`规范和组合工厂模式的模块，可以看看[react-stampit](https://www.npmjs.com/package/react-stampit)，它在前端使用组合工厂模式，使您可以轻松地组合组件功能，[remitter](https://www.npmjs.com/package/remitter)，一个基于`Redis`的`pub / sub`模块。

## 揭示构造函数模式（`Revealing constructor`）
揭示构造函数模式是一个相对较新的模式，在`Node.js`社区和`JavaScript`中越来越受到重视，特别是因为它在一些核心库（如`Promise`）中使用。

我们已经在`Chapter4-Asynchronous Control Flow Patterns with ES2015 and Beyond`中隐含地看到了这种模式，但是我们再回过头来分析一下`Promise`构造函数，以更详细地描述它：

```javascript
const promise = new Promise(function(resolve, reject) {
  // ...
});
```

正如你所看到的，`Promise`接受一个函数作为构造函数的参数，这被称为执行函数。这个函数是由`Promise`构造函数的内部实现调用的，它提供给构造函数，用于处理`pending`状态的`promise`的内部状态。换句话说，它确定了一个方式来调用`resolve`和`reject`函数，`promise`遵循这个机制，调用`resolve`和`reject`来改变对象的内部状态。

这样做的好处是只有构造函数的参数函数才有权`resolve`和`reject`，一旦构造了`Promise`对象，就可以安全地传递；没有其他代码将能够调用`resolve`或`ject`，来改变`Promise`的内部状态。
这就是为什么这个模式被`Domenic Denicola`的一篇博客文章命名为揭示构造函数模式的原因。

### 一个只读的`event emitter`
在这一段中，我们将使用揭示构造函数模式来构建一个只读的`event emitter`，这是一种特殊类型的`event emitter`，在这个`event emitter`内部方法，不允许调用`emit`方法，只有传递给构造函数的函数参数才能够调用`emit`方法。

让我们将`Roee`类的代码写入名为`roee.js`的文件中：

```javascript
const EventEmitter = require('events');
module.exports = class Roee extends EventEmitter {
  constructor(executor) {
    super();
    const emit = this.emit.bind(this);
    this.emit = undefined;
    executor(emit);
  }
};
```

在这个简单的类中，我们扩展了核心模块`EventEmitter`类，其接受一个`executor`函数作为构造函数的唯一参数。

在构造函数内部，我们调用`super`函数来确保通过调用其父构造函数来正确地初始化`event emitter`，然后保存`emit`函数的备份，并通过为其分配`undefined`来删除它。

最后，我们通过传递`emit`方法备份作为参数来调用`executor`函数。

这里要了解的重要一点是，在`undefined`被分配给`emit`方法之后，我们不能再从代码的其他部分调用它了。 我们的`emit`的备份版本被定义为一个局部变量，只会被转发给执行器函数。这个机制使我们能够仅在`executor`函数内使用`emit`。

现在让我们使用这个新类来创建一个简单的`ticker`，一个每秒发出一个`tick`并记录所有`tick`发出的数量的类。

这将是我们新的`ticker.js`模块的内容：

```javascript
const Roee = require('./roee');
const ticker = new Roee((emit) => {
  let tickCount = 0;
  setInterval(() => emit('tick', tickCount++), 1000);
});
module.exports = ticker;
```

正如你在这里看到的，代码量并不大。 我们实例化一个新的`Roee`，并在`executor`函数内传递`emit`作为参数。正是因为我们的`executor`函数接收`emit`作为参数，所以我们可以使用它每秒发出一个新的`tick`事件。

现在我们举例说明如何使用这个模块：

```javascript
const ticker = require('./ticker');
ticker.on('tick', (tickCount) => console.log(tickCount, 'TICK'));
// ticker.emit('something', {}); <-- This will fail
```

我们使用与任何其他基于`event emitter`的对象相同的`ticker`对象，我们可以用`on`方法附加任意数量的监听器，但是在这种情况下，如果我们尝试使用`emit`方法，那么我们的代码将抛出异常`TypeError: ticker.emit is not a function`。

> 即使这个例子在展示如何使用揭示构造函数模式，但值得一提的是这个事件发生器的只读功能并不是完美的，并且仍然有可能以几种方式绕过它。例如，我们仍然可以通过直接使用原型上的`emit`在我们的`ticker`实例上发出事件，如下所示：

```javascript
require('events').prototype.emit.call(ticker, 'someEvent', {});
```

### 实际应用场景
即使这种模式非常有趣和智能，但实际上，除了`Promise`构造函数以外，很难找到常见的应用实例。

值得一提的是，现在`Streams`议案中有一个新的规范，可以尝试使用揭示构造函数模式替代现今的模板模式，以便能够描述各种`Streams`对象的行为：可以看 https://streams.spec.whatwg.org/ 

另外需要指出的是，在之前`Chapter 5-Coding with Streams`当我们实现了`ParallelStream`类的时候。这个类作为构造函数参数接受`userTransform`函数作为参数（`executor`）。

即使在这种情况下，`executor`函数在构建时不被调用，但在`Streams`的内部`_transform()`方法中，揭示构造函数模式的一般概念仍然有效。实际上，这种方法允许我们在创建一个新的`ParallelStream`实例时，将`Streams`的一些内部方法（例如`push`函数）暴露给`executor`函数，使得我们在调用构造函数创建`ParallelStream`实例时执行与内部方法相关的一些操作。

## 代理模式（`Proxy`）
代理是一个控制访问另一个被称为主体对象的对象。代理对象和主体对象有一套相同的接口，这使得在使用代理的过程中，对于使用者而言是透明的。这种模式称为代理模式。代理拦截了所有要在主体对象上进行的操作，并增强或补充主体对象的行为。如图所示：

![](http://oczira72b.bkt.clouddn.com/18-1-25/4709175.jpg)

上图给我们展示了代理对象和主体对象具有相同的接口，以及这对客户端来说是如何完全透明的，客户端可以互换地使用其中一个。代理将每个操作转发给主体，通过额外的预处理或后处理来增强其行为。

> 要注意的是，我们并不是在讨论对于不同类需要实现不同的代理。代理模式要求代理对象需要保持各自主体的状态。

代理在几种情况下是有用的；例如，考虑以下几点情况：

* 数据验证：在代理向主体转发数据前验证其数据输入的合法性。
* 安全性：代理验证客户端是否有权限，仅仅当有权限时才会向主体对象发送相关请求。
* 缓存：代理对象保存内部缓存，仅仅当缓存未命中时才向主体对象发送相关请求。
* 懒加载：如果主体对象的创建需要消耗大量资源，代理可以推迟创建主体对象的时机，仅仅当需要主体对象时才创建主体对象。
* 日志：代理拦截方法和对应的参数调用，并在他们执行前后实现日志打印。
* 远程对象：代理可以接收远程对象，并使得其呈现为本地对象。

当然，代理模式还有更多的应用，但以上这些应该能让我们了解其主要用途。

### 实现代理的技术
当代理一个对象时，我们可以拦截所有的方法，或者只拦截其中的一些，而把其余的直接委托给主体对象。有几种方法可以实现这一点。让我们来分析其中的一些方法。

#### 对象组合
对象组合是一种将对象与另一个对象组合起来的技术，便于扩展或使用其中一个对象功能。对于代理模式而言，创建具有与主体对象相同接口的新对象，并且对该主体的引用以实例变量或闭包变量的形式存储在代理内部。

主体对象可以在创建时从客户端注入，也可以由代理自己创建。

以下是使用伪类和工厂模式创建代理对象的一个例子：

```javascript
function createProxy(subject) {
  const proto = Object.getPrototypeOf(subject);

  function Proxy(subject) {
    this.subject = subject;
  }
  Proxy.prototype = Object.create(proto);
  //proxied method
  Proxy.prototype.hello = function() {
    return this.subject.hello() + ' world!';
  };
  //delegated method
  Proxy.prototype.goodbye = function() {
    return this.subject.goodbye
      .apply(this.subject, arguments);
  };
  return new Proxy(subject);
}
module.exports = createProxy;
```

为了使用对象组合实现代理，我们必须拦截我们需要的方法（比如`hello()`），对于我们不需要的方法，则委托给主体对象调用（例如`goodbye()`方法）。

前面的代码也显示了主体对象有一个原型的特定情况，我们希望维护正确的原型链，以便执行代理`instanceof Subject`将返回`true`；我们使用继承来实现这一点。

这只是一个额外的步骤，当我们想要保持原型链时，才需要这个步骤，这对于改进代理的兼容性有用的。

但是，由于`JavaScript`具有动态类型，大多数情况下我们可以避免使用继承，并使用更直接的方法。例如，前面的代码中提供的代理的另一种实现，可能只使用对象字面量和工厂模式：

```javascript
function createProxy(subject) {
  return {
    // 代理方法
    hello: () => (subject.hello() + ' world!'),
    // 委托方法
    goodbye: () => (subject.goodbye.apply(subject, arguments))
  };
}
```

> 如果我们想创建一个委托其大部分方法的代理，那么使用称为[delegates](https://npmjs.org/package/delegates)自动生成这些代理会很方便。

#### 对象增强
对象增强是实现代理模式最佳的方式，通过用在代理对象上实现替换方法来直接修改对象；看下面的例子：

```javascript
function createProxy(subject) {
  const helloOrig = subject.hello;
  subject.hello = () => (helloOrig.call(this) + ' world!');
  return subject;
}
```

当我们需要实现的代理只有一个或几个方法的时候，这个技术绝对是最方便的，但是它有一个缺点，就是直接修改主体对象。

#### 不同技术的比较
对象组合被认为是创建代理的最安全的方式，因为它可以在不改变主体对象的原始行为的情况下创建代理。它唯一的缺点是我们必须手动委托所有的方法，即使我们只想代理其中的一个方法。如果需要的话，我们可能还必须委托访问主体对象的属性。

> 对象原型能够通过使用`Object.defineProperty()`被委托，可以查看[Object.defineProperty()的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

对于对象增强而言，可能我们并不是总想要修改主体对象，但是它没有出现在委派方法中出现的种种不便。为此，对象增强是 用`JavaScript`实现代理最实用的方式，如果修改主体对象不会导致大问题，对象增强是首选的技术。

然而，至少有一种情况下，对象组合几乎是必需的；这就是我们想要控制主体对象的初始化时，例如，只在需要它的时候才创建(懒加载)。

> 值得指出的是，通过使用工厂函数（在我们的例子中是`createProxy()`），我们可以将代码从用于生成代理。

### 创建一个可写入的日志流
为了实现一个代理模式的例子，现在我们创建一个可写入`Streams`的例子，通过拦截对`write()`函数的全部调用，然后对于每次调用记录一条信息。我们会使用对象组合来实现我们的代理，创建一个`loggingWritable.js`文件来实现：

```javascript
const fs = require('fs');

function createLoggingWritable(writableOrig) {
  const proto = Object.getPrototypeOf(writableOrig);

  function LoggingWritable(writableOrig) {
    this.writableOrig = writableOrig;
  }

  LoggingWritable.prototype = Object.create(proto);

  LoggingWritable.prototype.write = function(chunk, encoding, callback) {
    if(!callback && typeof encoding === 'function') {
      callback = encoding;
      encoding = undefined;
    }
    console.log('Writing ', chunk);
    return this.writableOrig.write(chunk, encoding, function() {
      console.log('Finished writing ', chunk);
      callback && callback();
    });
  };

  LoggingWritable.prototype.on = function() {
    return this.writableOrig.on
      .apply(this.writableOrig, arguments);
  };

  LoggingWritable.prototype.end = function() {
    return this.writableOrig.end
      .apply(this.writableOrig, arguments);
  };

  return new LoggingWritable(writableOrig);
}
```

在前面的代码中，我们创建了一个返回代理对象的代理版本的工厂函数，工厂函数需要传递主体对象作为参数。我们覆盖了`write()`方法，每次调用`write()`时都会将消息记录到标准输出，并且每次异步操作完成时都会记录消息。这也是创建异步函数的代理的一个很好的例子，因为我们知道代理回调函数也是必要的。这是在诸如`Node.js`的平台中要考虑的重要细节。 其余的方法`on()`和`end()`只是委托给原来的可写入的`Streams`（为了让代码更加精简，我们没有考虑可写入接口的其他方法）。
现在我们可以在`loggingWritable.js`模块中添加几行代码来测试我们刚创建的代理：

```javascript
const writable = fs.createWriteStream('test.txt');
const writableProxy = createLoggingWritable(writable);

writableProxy.write('First chunk');
writableProxy.write('Second chunk');
writable.write('This is not logged');
writableProxy.end();

```

因为使用对象组合，这个代理不会改变`Streams`或者它的外部行为的原有接口，如果我们运行前面的代码，我们现在回看到每个数据块写入`Streams`的过程透明地写到控制台中。

### 代理生态-函数钩子和AOP
在众多的设计模式中，代理模式在`Node.js`以及生态系统中都是相当流行的模式。实际上，我们可以找到几个允许我们简化代理创建的库，大部分时间利用对象增强作为实现方法。在社区中，这个模式也可以称为函数挂钩，或者有时称为面向方面的编程（`AOP`），它实际上是代理的一个常见应用领域。在`AOP`中，这些库通常允许开发人员为特定方法（或一组方法）设置执行前或执行后钩子，这些方法允许我们分别在执行建议的方法之前和之后执行自定义代码。

有时，代理也被称为中间件，因为在中间件模式中（我们将在本章后面会看到），它们允许我们对函数的输入/输出进行预处理和后处理。有时，他们也允许使用类似中间件的管道为同一方法注册多个钩子。

在`npm`上有几个库允许我们用很少的努力实现函数钩子。其中有[hooks](https://npmjs.org/package/hooks)，[hooker](https://npmjs.org/package/hooker)，和[meld](https://npmjs.org/package/meld)。

### ES2015的Proxy
`ES2015`规范引入了一个名为`Proxy`的全局对象，它可以从开始在`Node.js v6.0`中使用。

`Proxy API`包含一个`Proxy`构造函数，它接受一个`target`和一个`handler`作为参数：

```javascript
const proxy = new Proxy(target, handler);
```

这里，`target`表示应用代理的对象（我们的规范定义的主体对象），而`handler`是定义代理行为的特殊对象。

`handler`对象包含一系列具有预定义名称的可选方法，这些方法称为陷阱方法（例如，`apply`，`get`，`set`和`has`），这些方法在代理实例上执行相应的操作时会自动调用。

为了更好地理解这个`API`的工作原理，我们来看一个例子：

```javascript
const scientist = {
  name: 'nikola',
  surname: 'tesla'
};
const uppercaseScientist = new Proxy(scientist, {
  get: (target, property) => target[property].toUpperCase()
});
console.log(uppercaseScientist.name, uppercaseScientist.surname);
// NIKOLA TESLA
```

在这个例子中，我们使用`Proxy API`来拦截对目标对象`scientist`属性的所有访问，并将属性的原始值转换为大写字符串。

如果你仔细看看这个例子，你可能会注意到这个`API`的一些特别的东西：它允许我们拦截对目标对象的通用属性的访问。这是可能的，因为`API`不仅仅是一个简单的包装来促进代理对象的创建，就像我们在本章前面部分所定义的那样；相反，它是深入集成到`JavaScript`语言本身的一个特性，它使开发人员能够拦截和定制可以在对象上执行的许多操作。这个特性开创了一些新的有趣的场景，这些场景在元编程，运算符重载和对象虚拟化之前是不容易实现的。

我们来看另一个例子来阐述这个概念：

```javascript
const evenNumbers = new Proxy([], {
  get: (target, index) => index * 2,
  has: (target, number) => number % 2 === 0
});
console.log(2 in evenNumbers); // true
console.log(5 in evenNumbers); // false
console.log(evenNumbers[7]); // 14
```

在这个例子中，我们正在创建一个包含所有偶数的虚拟数组。它可以作为常规数组使用，这意味着我们可以使用常规数组语法访问数组中的每一项（例如，`evenNumbers[7]`），或者使用`in`运算符检查数组中是否存在元素（例如，偶数中有`2`个）。该数组被认为是虚拟的，因为我们从不在其中存储数据。
看一下这个实现，这个代理使用一个空的数组作为目标，然后在处理程序中定义陷阱`get`和`has`：

get陷阱拦截对数组元素的访问，返回给定索引的双倍，而是拦截`in`运算符的用法，并检查给定的数字是否是偶数。

`Proxy API`支持一些其他有趣的陷阱，如`set`，`delete`，和`construct`，并允许我们创建代理，可以根据需要撤销，禁用所有的陷阱和恢复`target`对象的原始行为。

分析所有这些功能超出了本章的范围。这里重要的是理解`Proxy API`提供了一个强大的基础，以便在需要时利用代理模式。

> 如果您想了解更多关于`Proxy API`的知识并发现其所有功能和陷阱方法，请参阅`Mozilla`的本文中的更多内容： https://developer.mozilla.org/it/docs/Web/JavaScript/Reference/Global_Objects/Proxy 。 另一个很好的来源是来自`Google`的详细文章： https://developers.google.com/web/updates/2016/02/es2015-proxies 。

### 实际应用场景
[Mongoose](http://mongoosejs.com)是`MongoDB`的一个流行的对象文档映射（`ODM`）库。 在内部，它使用[hooks](https://npmjs.org/package/hooks)为`init`，`validate`，`save`和`remove`函数提供预处理和后处理的钩子函数。有关官方文档，请参阅[Mongoose的官方文档](http://mongoosejs.com/docs/middleware.html)。

## 装饰者模式（`Decorator`）