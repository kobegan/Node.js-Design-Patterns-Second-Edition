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
装饰者模式是一种结构模式，由动态增加现有对象的行为组成。 这与经典继承不同，因为行为不会添加到同一类的所有对象中，而只会添加到明确装饰的实例中。

从实现的角度来看，它与代理模式非常相似，但不是增强或修改对象的现有接口的行为，而是使用新功能增强它，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-1-28/5433037.jpg)

在上图中，`Decorator`对象通过添加`methodC()`操作来扩展`Component`对象。

通常将现有的方法委托给装饰对象，而无需进一步处理。 当然，如果需要，我们可以轻松地组合代理模式，以便对现有方法的调用也可以被拦截和操纵。

### 实现装饰者模式的技巧
虽然代理模式和装饰者模式在概念上是两种不同的模式，不同的模式，他们实际上共享相同的实施策略。让我们来回顾一下。

#### 对象组合
使用组合，被装饰的组件通常被包裹在继承它的新对象周围。在这种情况下，装饰器只需要定义新的方法，而将现有的方法委托给原始组件：

```javascript
function decorate(component) {
  const proto = Object.getPrototypeOf(component);

  function Decorator(component) {
    this.component = component;
  }
  Decorator.prototype = Object.create(proto);
  // 新方法
  Decorator.prototype.greetings = function() {
    return 'Hi!';
  };
  // 委托方法
  Decorator.prototype.hello = function() {
    return this.component.hello.apply(this.component, arguments);
  };
  return new Decorator(component);
}
```

#### 对象增强
装饰者模式也可以通过简单地将新方法直接附加到被装饰对象来实现，如下所示：

```javascript
function decorate(component) {
  // 新方法
  component.greetings = () => {
    return component;
  };
}
```

对于使用对象组合和对象增强在代理模式的缺陷也同样适用于装饰者模式。现在让我们通过一个实例来练习装饰者模式！

### 装饰一个`LevelUP`数据库
在我们开始编码下一个例子之前，先说一下我们现在要使用的模块`LevelUP`。

#### 介绍`LevelUP`和`LevelDB`
[LevelUP](https://npmjs.org/package/levelup)是`Google`的`LevelDB`上的一个`Node.js`包装器，它是最初为了在`Chrome`浏览器中实现`IndexedDB`而创建的键/值存储库，但它远不止于此。由于其极简主义和可扩展性，`LevelDB`被`Dominic Tarr`定义为“Node.js的数据库”。像`Node.js`一样，`LevelDB`提供了非常高效的性能，只有最基本的一组功能，允许开发人员在其上构建任何类型的数据库。
`Node.js`社区（在这种情况下是`Rod Vagg`）并没有错过通过创建`LevelUP`将这个数据库的强大功能带入`Node.js`的机会。作为`LevelDB`的包装，它演变成支持从内存存储到其他`NoSQL`数据库（如`Riak`和`Redis`）到Web存储引擎（如`IndexedDB`和`localStorage`）的几种后端，使我们可以在服务器和客户端上使用相同的`API`，开放了一些非常有趣的场景。

现在，`LevelUP`已经形成了一个完整的生态系统，由插件和模块组成，扩展了微型核心，实现复制，二级索引，实时更新，查询引擎等功能。而且，完整的数据库是建立在`LevelUP`之上的，包括`CouchDB`的克隆（例如[PouchDB](https://npmjs.org/package/pouchdb)和[CouchUP](https://npmjs.org/package/couchup)），甚至包括图数据库，[levelgraph](https://npmjs.org/package/levelgraph)，它可以在`Node.js`和浏览器上工作！

> 了解更多关于LevelUP生态系统的信息： https://github.com/rvagg/node-levelup/wiki/Modules 

#### 实现一个`LevelUP`插件
在下一个示例中，我们将展示如何使用装饰者模式为`LevelUP`创建一个简单的插件，特别是使用对象增强技术，这是最简单但仍然最实用且最有效的方法来装饰对象能力。

> 为了方便，我们将使用[level](http://npmjs.org/package/level)，它捆绑了`levelup`和名为`leveldown`的默认适配器，后者使用`LevelDB`作为后端。

我们想要构建的是一个`LevelUP`的插件，它允许我们在每次将具有特定模式的对象保存到数据库时接收通知。 例如，如果我们订阅`{a: 1}`这种类型的对象，我们希望在保存诸如`{a: 1, b: 3}`或`{a: 1, c: 'x'}`的对象时收到通知进入数据库。

我们开始通过创建一个名为`levelSubscribe.js`的新模块来构建我们的小插件。 然后我们将插入下面的代码：

```javascript
module.exports = function levelSubscribe(db) {
  db.subscribe = (pattern, listener) => {       //[1]
    db.on('put', (key, val) => {         //[2]
      const match = Object.keys(pattern).every(
        k => (pattern[k] === val[k])     //[3]
      );
      
      if(match) {
        listener(key, val);            //[4]
      }
    });
  };
  return db;
};
```

这就是我们的插件，它非常简单。让我们简单看看在前面的代码中会发生什么：

1. 用一个名为`subscribe()`的新方法来装饰`db`对象。并且使用对象增强的方式直接将方法直接附加到提供的`db`实例。
2. 监听对数据库进行的任何`put`操作。
3. 执行了一个非常简单的模式匹配算法，它验证了所提供的模式中的所有属性。
4. 一旦匹配成功，通知监听者。

现在让我们来创建一些代码 - 在一个名为`levelSubscribeTest.js`的新文件中 - 试用我们的新插件：

```javascript
const level = require('level'); // [1]
const levelSubscribe = require('./levelSubscribe'); // [2]

let db = level(__dirname + '/db', {valueEncoding: 'json'});

db = levelSubscribe(db);
db.subscribe(
  {doctype: 'tweet', language: 'en'}, // [3]
  (k, val) => console.log(val)
);

db.put('1', {doctype: 'tweet', text: 'Hi', language: 'en'}); //[4]
db.put('2', {doctype: 'company', name: 'ACME Co.'});
```

这就是我们在前面的代码中所做的：

1. 首先，我们初始化我们的`LevelUP`数据库，选择存储文件的目录以及这些值的默认编码。
2. 然后，我们附上我们的插件，它装饰原始的`db`对象。
3. 此时，我们准备使用由我们的插件提供的新特性，即`subscribe()`方法，在那里我们订阅包含`doctype: "tweet"`和`language: "en"`属性的对象。
4. 最后，我们使用`put`来保存数据库中的一些值。 第一个调用将触发与订阅相关的回调，我们将看到存储在控制台中的对象。这是因为在这种情况下，对象匹配订阅。相反，第二个调用将不会生成任何输出，因为存储的对象将不符合订阅条件。

这个例子展示了装饰者模式在其最简单实现中的实际应用：对象 增强。它看起来像一个普通的模式，但是如果使用得当，它无疑是很强大的。

> 为了简单起见，我们的插件只能与`put`操作结合使用，但是实际上对于[batch](https://github.com/rvagg/node-levelup#batch)操作也可以使用装饰者模式来进行拓展。

### 实际应用场景
有关更多使用装饰器的更多示例，我们可能要阅读一些更多的`LevelUP`插件的代码：

* [level-inverted-index](https://github.com/dominictarr/level-inverted-index)：这是一个插件，它将倒排索引添加到`LevelUP`数据库中，允许我们在存储在数据库中的值上执行简单的文本搜索。
* [level-plus](https://github.com/eugeneware/levelplus)：这是一个将原子更新添加到`LevelUP`数据库的插件。

## 适配器模式（`Adapter`）
适配器模式允许我们使用不同的接口访问对象的功能。顾名思义，它适配一个对象，以便它可以被不同接口调用。

下图阐述了适配器模式情况：

![](http://oczira72b.bkt.clouddn.com/18-1-28/85641871.jpg)

上图显示了`Adapter`对象的本质是`Adaptee`对象的包装，暴露了一个不同的接口。该图还突出显示了`Adapter`对象的操作也可以是`Adaptee`对象上一个或多个方法调用的组合。从实现的角度来看，最常见的技术是组合，其中`Adapter`的方法为`Adaptee`的方法提供了桥梁。这个模式非常简单，让我们举例说明。

### 通过文件系统`API`使用`LevelUP`
现在我们将围绕`LevelUP API`构建一个适配器，将其转换为与核心`fs`模块兼容的接口。 特别是，我们将确保每次调用`readFile()`和`writeFile()`都将转化为对`db.get()`和`db.put()`的调用。 这样我们就可以使用一个`LevelUP`数据库作为简单文件系统操作的存储后端。

首先创建一个名为`fsAdapter.js`的新模块。 我们将首先加载依赖关系并导出我们要用来构建适配器`createFsAdapter()`工厂函数：

```javascript
const path = require('path');
module.exports = function createFsAdapter(db) {
  const fs = {};
  // ...
}
```

接下来，我们会在工厂函数内实现`readFile()`函数，确保它的接口
与`fs`模块某一个原有函数是兼容的：

```javascript
fs.readFile = function(filename, options, callback) {
  if (typeof options === 'function') {
    callback = options;
    options = {};
  } else if (typeof options === 'string') {
    options = {
      encoding: options
    };
  }
  db.get(path.resolve(filename), { //[1]
      valueEncoding: options.encoding
    },
    function(err, value) {
      if (err) {
        if (err.type === 'NotFoundError') { //[2]
          err = new Error('ENOENT, open \'' + filename + '\'');
          err.code = 'ENOENT';
          err.errno = 34;
          err.path = filename;
        }
        return callback && callback(err);
      }
      callback && callback(null, value); //[3]
    }
  );
};
```

在前面的代码中，我们不得不做一些额外的工作，以确保我们的新函数的行为尽可能接近原始的`fs.readFile()`函数。

该函数所执行的步骤如下所述：

1. 为了从`db`类提取一个文件，我们使用`filename`作为`key`调用`db.get()`，使用它的完整的路径（使用`path.resolve()`）。我们设置`valueEncoding`的值，等于作为输入参数的任意`encoding`选项。
2. 如果`key`在数据库没有找到，我们创建一个带有`ENOENT`作为错误码的`error`，错误码`fs`模块用来表示一个不存在的文件。其余的`error`转发给`callback`。
3. 如果`key-value`对从数据库中成功提取，我们会使用`callback`将`value`返回给调用者。

我们可以看到，我们创建的函数相当粗糙， 它并不可能成为`fs.readFile()`函数的完美替代品，但是在最常见的情况下它肯定会完成它的工作。

为了完成我们的`fs`模块适配器插件，现在让我们看看如何实现`writeFile()`函数：

```javascript
fs.writeFile = (filename, contents, options, callback) => {
  if (typeof options === 'function') {
    callback = options;
    options = {};
  } else if (typeof options === 'string') {
    options = {
      encoding: options
    };
  }
  db.put(path.resolve(filename), contents, {
    valueEncoding: options.encoding
  }, callback);
}
```

另外，在这种情况下，我们没有进行完美的包装和适配，因为我们忽略了一些选项，比如对于文件权限（`options.mode`）来说，我们会按照原样传递从数据库收到的任何错误。

最后，我们只需要返回`fs`对象并使用下面这行代码关闭工厂函数：

```javascript
return fs;
```

`fsAdapter.js`完整代码：

```javascript
const path = require('path');

module.exports = function createFsAdapter(db) {
  const fs = {};

  fs.readFile = (filename, options, callback) => {
    if (typeof options === 'function') {
      callback = options;
      options = {};
    } else if(typeof options === 'string') {
      options = {encoding: options};
    }

    db.get(path.resolve(filename), {         //[1]
        valueEncoding: options.encoding
      },
      (err, value) => {
        if(err) {
          if(err.type === 'NotFoundError') {       //[2]
            err = new Error(`ENOENT, open "${filename}"`);
            err.code = 'ENOENT';
            err.errno = 34;
            err.path = filename;
          }
          return callback && callback(err);
        }
        callback && callback(null, value);       //[3]
      }
    );
  };

  fs.writeFile = (filename, contents, options, callback) => {
    if(typeof options === 'function') {
      callback = options;
      options = {};
    } else if(typeof options === 'string') {
      options = {encoding: options};
    }

    db.put(path.resolve(filename), contents, {
      valueEncoding: options.encoding
    }, callback);
  };

  return fs;
};
```

我们的新适配器插件已经准备就绪；如果我们现在编写一个测试模块，我们可以尝试使用它：

```javascript
const fs = require('fs');

fs.writeFile('file.txt', 'Hello!', () => {
  fs.readFile('file.txt', {encoding: 'utf8'}, (err, res) => {
    console.log(res);
  });
});

// 试图读取不存在的文件
fs.readFile('missing.txt', {encoding: 'utf8'}, (err, res) => {
  console.log(err);
});
```

![](http://oczira72b.bkt.clouddn.com/18-1-28/57959386.jpg)

上面的代码使用原始的`fs API`在文件系统上执行一些读写操作，并应该在控制台上打印如下内容：

```
{ [Error: ENOENT, open 'missing.txt'] errno: 34, code: 'ENOENT', path: 'missing.txt' }
Hello!
```

现在，我们可以尝试用我们的适配器替换`fs`模块，如下所示：

```javascript
const levelup = require('level');
const fsAdapter = require('./fsAdapter');
const db = levelup('./fsDB', {
  valueEncoding: 'binary'
});
const fs = fsAdapter(db);
```

再次运行我们的程序应该产生相同的输出，除了我们指定的文件没有任何部分是使用文件系统读取或写入的。相反，使用我们的适配器执行的任何操作都将转换为在`LevelUP`数据库上执行的操作。

我们刚创建的适配器可能看起来很傻，因为我们不明确使用数据库代替真正的文件系统的目的是什么。但是，我们应该记住，`LevelUP`本身具有适配器，可以使数据库也在浏览器中运行；其中一个适配器是[level.js](https://npmjs.org/package/level-js)。现在我们的适配器应该是完美的。我们可以考虑使用它来与依赖于`fs`模块的浏览器代码共享！例如，我们在`Chapter 3-Asynchronous Control Flow Patterns with Callbacks`中创建的`Web爬虫应用程序`使用`fs API`来存储在其操作期间下载的网页；我们的适配器将允许它在浏览器中运行只需稍作修改！我们很快就会意识到，在涉及到与浏览器共享代码时，适配器模式也是一个非常重要的模式，我们将在`Chapter8-Universal JavaScript for Web Applications`中详细介绍。

### 实际应用场景
适配器模式有很多实际应用场景的例子，在这里列出一些最值得注意的例子进行分析：

* 我们已经知道`LevelUP`能够在浏览器中使用不同的存储后端运行，从默认的`LevelDB`到`IndexedDB`。 这是通过创建复制内部私有的`LevelUP API`的各种适配器实现的。可以到以下链接看看是如何实现的： https://github.com/rvagg/node-levelup/wiki/Modules#storage-back-ends 。
* `jugglingdb`是一个多数据库的`ORM`，当然，使用多个适配器使其与不同的数据库兼容。可以到以下链接看看是如何实现的： https://github.com/1602/jugglingdb/tree/master/lib/adapters 。
* 对我们创建的例子的完美补充是[level-filesystem](https://www.npmjs.org/package/level-filesystem)，它是在`LevelUP`之上的`fs API`的正确实现。

## 策略模式（`Strategy`）
策略模式通过将可变部分提取为单独的，可交换的对象`Strategy`来使对象`Context`支持其逻辑中的变化。`Context`实现通用逻辑，而策略实现了可变部分，允许上下文根据不同因素（如输入值，系统配置或用户偏好）调整其行为。这些策略通常是解决方案的一部分，他们都实现了相同的接口，这是`Context`对象所期望的接口。下图显示了我们刚刚描述的情况：

![](http://oczira72b.bkt.clouddn.com/18-1-28/45787526.jpg)

上图显示了`Context`对象如何将不同的策略插入到其结构中，就好像它们是一个机器的可替换部分一样。想象一下汽车，其轮胎可以视为适应不同路况的策略。我们可以安装冬季轮胎在雪路上行驶，这要归功于他们的螺栓，而我们可以决定为高速公路行驶的高性能轮胎做长途旅行。一方面，我们不想把整个车改变，另一方面，我们不想要一辆八轮车，这样就可以在任何一条路上行驶。

我们很快就明白这种模式有多强大，不仅有助于分离算法中的关注点，而且还使其具有更好的灵活性并适应同一问题的不同变化。

策略模式在支持算法变化需要复杂的条件逻辑（大量的`if...else`或`switch`语句）或混合同一族不同算法的所有情况下特别有用。设想一个名为`Order`的对象，表示一个电子商务网站的在线订单。该对象有一个名为`pay()`的方法，就像它说的那样，完成订单并将资金从用户转移到商城用户。

为了支持不同的支付系统，我们有几个选项，如下所示：

* 在`pay()`方法中使用`if...else`语句来完成基于操作的操作。
* 在选择的付款选项上将支付的逻辑委托给实现用户选择的特定支付网关逻辑的策略对象。

在第一种解决方案中，我们的订单对象不能支持其他支付方式，除非其代码被修改。而且，当支付选项的数量增加时，这可能变得相当复杂。相反，使用策略模式使得`Order`对象支持几乎无限数量的支付方法，并且保持其范围仅限于管理用户的细节，购买的项目和相对价格，同时将完成支付的工作委派给另一个对象。

现在让我们用一个简单实际的例子来展示这个模式。

### 多格式配置对象
让我们考虑一个名为`Config`的对象，该对象包含应用程序使用的一组配置参数，例如数据库URL，服务器的侦听端口等。`Config`对象应该能够提供一个简单的接口来访问这些参数，而且还可以使用持久性存储（如文件）导入和导出配置。我们希望能够支持不同的格式来存储配置，例如`JSON`，`INI`或`YAML`。

通过应用我们了解的策略模式，我们可以立即识别`Config`对象的变量部分，这是允许我们序列化和反序列化配置的功能。

让我们创建一个名为`config.js`的新模块，让我们定义配置管理器的通用部分：

```javascript
const fs = require('fs');
const objectPath = require('object-path');
class Config {
  constructor(strategy) {
    this.data = {};
    this.strategy = strategy;
  }
  get(path) {
    return objectPath.get(this.data, path);
  }
  // ...
}
```

在前面的代码中，我们将配置数据封装到一个实例变量（`this.data`）中，然后我们提供了`set()`和`get()`方法，允许我们使用`object-path`访问配置属性（例如，`property.subProperty`），通过利用[object-path](https://npmjs.org/package/object-path)。在构造函数中，我们也采取了一种策略作为输入，它代表解析和序列化数据的算法。

现在让我们看看我们将如何使用策略，开始编写`Config`类的剩余部分：

```javascript
const fs = require('fs');
const objectPath = require('object-path');

class Config {
  constructor(strategy) {
    this.data = {};
    this.strategy = strategy;
  }

  get(path) {
    return objectPath.get(this.data, path);
  }

  set(path, value) {
    return objectPath.set(this.data, path, value);
  }

  read(file) {
    console.log(`Deserializing from ${file}`);
    this.data = this.strategy.deserialize(fs.readFileSync(file, 'utf-8'));
  }

  save(file) {
    console.log(`Serializing to ${file}`);
    fs.writeFileSync(file, this.strategy.serialize(this.data));
  }
}

module.exports = Config;
```

在前面的代码中，当从文件中读取配置时，我们将反序列化任务委托给策略；那么当我们想把配置保存到文件中时，我们使用策略来序列化配置。这个简单的设计允许`Config`对象在加载和保存数据时支持不同的文件格式。

为了演示这一点，我们在一个名为`strategies.js`的文件中创建一些策略。 让我们从解析和序列化`JSON`数据的策略开始：

```javascript
module.exports.json = {
  deserialize: data => JSON.parse(data),
  serialize: data => JSON.stringify(data, null, '  ')
}
```

没有什么复杂的！我们的策略简单地实现了接口，以便它可以被`Config`对象使用。

同样，我们要创建的下一个策略允许我们支持`INI`文件格式：

```javascript
const ini = require('ini'); // https://npmjs.org/package/ini
module.exports.ini = {
  deserialize: data => ini.parse(data),
  serialize: data => ini.stringify(data)
}
```

现在，为了向您展示如何结合在一起，我们创建一个名为`configTest.js`的文件，让我们尝试使用不同的格式文件加载和保存示例配置：

```javascript
const Config = require('./config');
const strategies = require('./strategies');
const jsonConfig = new Config(strategies.json);
jsonConfig.read('samples/conf.json');
jsonConfig.set('book.nodejs', 'design patterns');
jsonConfig.save('samples/conf_mod.json');
const iniConfig = new Config(strategies.ini);
iniConfig.read('samples/conf.ini');
iniConfig.set('book.nodejs', 'design patterns');
iniConfig.save('samples/conf_mod.ini');
```

我们的测试模块揭示了策略模式的属性。我们只定义了一个`Config`类，它实现了我们的配置管理器的公共部分，同时改变了用于序列化和反序列化的策略，允许我们创建支持不同文件格式的不同`Config`实例。

前面的例子只显示了使用策略模式实现多格式配置对象的方法之一。其他有效的方法可能如下：

* 创建两个不同的策略系列：一个用于反序列化，另一个用于序列化。这将允许从格式读取并保存到另一个格式。
* 根据所提供文件的扩展名，动态选择策略；`Config`对象可以保持一个`map extension`->`strategy`，并用它来为给定的扩展名选择正确的算法。

正如我们所看到的，有几种选择使用策略的选择，正确的选择取决于我们的要求，以及我们希望获得的特性/简单性的折衷。

而且，模式本身的实现可能会有很大的不同，例如，以其最简单的形式，`context`和`strategy`都可以是简单的函数：

```javascript
function context(strategy) {...}
```

尽管前面的情况看起来可能微不足道，但在`JavaScript`等编程语言中，函数是一等公民，并且可以用作完全成熟的对象。

在所有这些变化之间，不变的是模式背后的思想；模式的实现可以稍微改变，但驱动模式实现的核心概念永远是一样的。

### 实际应用场景
[Passport.js](http://passportjs.org)是`Node.js`的认证框架，它允许在`Web`服务器上支持不同的认证方案。通过`Passport`，我们可以轻松使用`Facebook`登录或使用`Twitter`登录功能到我们的`Web`应用程序。`Passport`使用策略模式将认证过程中所需的公共逻辑与可以更改的部分（即实际的认证步骤）分开。例如，我们可能想要使用`OAuth`来获取访问令牌来访问`Facebook`或`Twitter`个人资料，或者只需使用本地数据库来验证用户名/密码。对于`Passport`，这些都是完成身份验证过程的不同策略，正如我们所能想象的，这使得这个库可以支持几乎无限的身份验证服务。客户以看看 http://passportjs.org/guide/providers 上支持的不同身份验证，以了解策略模式可以执行的操作。

## 状态模式（`State`）
状态模式是策略模式的变体，策略根据`Context`的状态而变化。 我们在前面的章节已经看到，如何根据用户的偏好，配置参数和提供的输入等不同的变量来选择一个策略，一旦这个选择完成，策略在`Context`剩余的寿命期间保持不变。

相反，在状态模式中，策略（在这种情况下也称为状态）是动态的，可以在`Context`的生命周期中改变，从而允许其行为根据其内部状态进行调整，如下图所示：

![](http://oczira72b.bkt.clouddn.com/18-1-28/91813397.jpg)

想象一下，我们有一个酒店预订系统和一个`Reservation`对象来模拟房间预订。

这是一个经典的情况，我们必须根据其状态来调整对象的行为。考虑以下一系列事件：

1. 当订单初始创建时，用户可以使用`confirm()`方法确认订单；当然，他们不能使用`cancel()`方法取消预约，因为订单还没有被确认。但是，如果他们在购买之前改变主意，他们可以使用`delete()`方法删除它。
2. 一旦确认订单，再次使用`confirm()`方法没有任何意义；不过，现在应该可以取消预约，但不能再删除，因为要保留对应记录。
3. 在预约日期前一天，不应取消订单。因为这太迟了。

现在想象一下，我们必须实现我们在一个单一的对象中描述的预订系统；我们已经可以画出所有的`if...else`或者`switch`语句逻辑图，这些语句是我们必须写的，以便根据预留的状态来启用/禁用每个动作。

在这种情况下，状态模式是完美的：将会有三种策略，全部实现描述的三个方法（`confirm()`，`cancel()`和`delete()`），每个只执行一个行为，一个策略对应于一种状态。通过使用状态模式，`Reservation`对象从一个行为切换到另一个行为应该是非常容易的。这只需要在每个状态变化上激活一个不同的策略。

状态转换可以由`Context`对象，客户端代码或`State`对象本身启动和控制。通常由`State`对象本身控制，因为这在灵活性和解耦方面效果较好，因为`Context`对象不必知道所有可能的状态以及如何在它们之间转换。

### 实现一个基本的`fail-safe socket`
现在我们来看一个具体的例子，以便我们能够运用我们所了解到的状态模式。让我们建立一个客户端`TCP`套接字，当与服务器的连接丢失时不会丢失客户端请求；相反，我们希望将服务器处于脱机状态的时间内发送的所有数据进行排队，然后在连接重新建立后立即尝试发送。我们希望在一个简单的监控系统中利用这个套接字，在这个系统中，一组机器每隔一段时间发送一些关于资源利用率的统计信息；如果收集这些资源的服务器关闭，则我们的套接字将继续在本地排队数据，直到服务器重新联机为止。

首先创建一个名为`failsafeSocket.js`的模块来表示我们的`context`对象：

```javascript
const OfflineState = require('./offlineState');
const OnlineState = require('./onlineState');

class FailsafeSocket {
  constructor (options) { // [1]
    this.options = options;
    this.queue = [];
    this.currentState = null;
    this.socket = null;
    this.states = {
      offline: new OfflineState(this),
      online: new OnlineState(this)
    };
    this.changeState('offline');
  }

  changeState (state) { // [2]
    console.log('Activating state: ' + state);
    this.currentState = this.states[state];
    this.currentState.activate();
  }

  send(data) { // [3]
    this.currentState.send(data);
  }
}

module.exports = options => {
  return new FailsafeSocket(options);
};
```

`FailsafeSocket`类由三个主要元素组成：

1. 构造函数初始化各种数据结构，包括将包含在套接字脱机时发送的任何数据的队列。此外，它还创建了一组两个状态，一个用于在脱机状态下实现套接字的行为，另一个用于在套接字处于联机状态时的状态。
2. `changeState()`方法负责从一个状态转换到另一个状态。 它只是更新`currentState`实例变量，并调用目标状态的`activate()`。
3. `send()`方法是套接字的功能，这是我们希望基于离线/在线状态具有不同行为的地方。我们可以看到，这是通过将操作委托给当前活动状态来完成的。

现在让我们来看看这两个状态是什么样子的，从`offlineState.js`模块开始：

```javascript
const jot = require('json-over-tcp'); // [1]

module.exports = class OfflineState {

  constructor (failsafeSocket) {
    this.failsafeSocket = failsafeSocket;
  }

  send(data) { // [2]
    this.failsafeSocket.queue.push(data);
  }

  activate() { // [3]
    const retry = () => {
      setTimeout(() => this.activate(), 500);
    };

    this.failsafeSocket.socket = jot.connect(
      this.failsafeSocket.options,
      () => {
        this.failsafeSocket.socket.removeListener('error', retry);
        this.failsafeSocket.changeState('online');
      }
    );
    this.failsafeSocket.socket.once('error', retry);
  }
};
```

我们创建的模块负责在脱机状态下管理套接字的行为：

1. 我们将使用一个名为[json-over-tcp](https://npmjs.org/package/json-over-tcp)的库来代替使用原始的`TCP`套接字，这将使我们能够轻松地在一个`TCP`连接中发送`JSON`对象。
2. `send()`方法只负责排队它接收到的任何数据。我们假设我们是离线的，这就是我们需要做的。
3. `activate()`方法尝试使用`json-over-tcp`与服务器建立连接。如果操作失败，则在`500`毫秒后再次尝试。它会继续尝试，直到建立有效的连接，在这种情况下，`failafeSocket`的状态将转换为联机状态。

接下来，让我们实现`onlineState.js`模块，然后让我们实现`onlineState`策略，如下所示：

```javascript
module.exports = class OnlineState {
  constructor(failsafeSocket) {
    this.failsafeSocket = failsafeSocket;
  }

  send(data) { // [1]
    this.failsafeSocket.socket.write(data);
  };

  activate() { // [2]
    this.failsafeSocket.queue.forEach(data => {
      this.failsafeSocket.socket.write(data);
    });
    this.failsafeSocket.queue = [];

    this.failsafeSocket.socket.once('error', () => {
      this.failsafeSocket.changeState('offline');
    });
  }
};
```

`OnlineState`策略非常简单，解释如下：

1. `send()`方法直接将数据写入套接字，因为我们假设`TCP`已连接。
2. `activate()`方法刷新套接字处于脱机状态时排队的所有数据，并且还开始监听任何`error`事件；我们将把这个作为套接字下线的前兆。发生这种情况时，我们转换到`offline`状态。

这就是`failsafeSocket`；现在我们准备构建一个示例客户端和一个服务器来尝试。把服务器代码放在一个名为`server.js`的模块中：

```javascript
const jot = require('json-over-tcp');
const server = jot.createServer({
  port: 5000
});
server.on('connection', socket => {
  socket.on('data', data => {
    console.log('Client data', data);
  });
});

server.listen({
  port: 5000
}, () => console.log('Started'));
```

> 注意：原书的代码有错，现在的`jot.createServer()`接受的参数是一个对象，这里把书上的`5000`改为`{ post: 5000 }`。

然后看客户端代码`client.js`：

```javascript
const createFailsafeSocket = require('./failsafeSocket');
const failsafeSocket = createFailsafeSocket({
  port: 5000
});
setInterval(() => {
  // 每隔1000毫秒发送当前内存使用状态
  failsafeSocket.send(process.memoryUsage());
}, 1000);
```

![](http://oczira72b.bkt.clouddn.com/18-1-28/25208527.jpg)

我们的服务器只是打印它接收到的任何`JSON`对象消息给控制台，而我们的客户端利用一个`FailsafeSocket`对象每秒发送一次内存利用率的测量值。

尝试构建的小型系统，我们应该运行客户端和服务器，然后通过停止然后重新启动服务器来测试`failafeSocket`的功能。 我们应该看到，客户端的状态在线和离线之间发生了变化，服务器离线时收集的任何请求都会排队，然后在服务器重新联机后重新发送。

这个例子应该清楚地说明状态模式如何能够帮助增加一个组件的模块化和可读性，这个组件必须根据状态来调整它的行为。

> 我们在本节中构建的`FailsafeSocket`类仅用于演示状态模式，并不希望成为处理`TCP`套接字内连接问题的完整且`100％`可靠的解决方案。例如，我们不验证写入套接字流，而让所有数据都被服务器接收到，这将需要更多与我们想描述的模式无关的代码。

## 模板模式（`Template`）
我们将要分析的下一个模式叫做模板模式，它与策略模式有许多共同点。模板由定义一个抽象的伪类组成，它代表了算法的框架，其中一些步骤是未定义的。然后子类可以通过实现缺少的步骤填充算法中的空白，称为模板方法。这种模式的目的是使定义一个类的家族成为可能，这些类都是类似算法的变体。下面的`UML`图显示了我们刚刚描述的结构：

![](http://oczira72b.bkt.clouddn.com/18-1-28/19628193.jpg)

上图中显示的三个具体类扩展了`Template`并为`templateMethod()`提供了一个实现，使用`C++`术语来说，该实现方法是抽象或者说是虚函数；在`JavaScript`中，这意味着该方法是未定义的或被分配给一个总是抛出异常的函数，这表明该方法必须被实现。模板模式可以被认为比我们目前所看到的其他模式更加符合面向对象思想，因为继承是其实现的核心部分。

模板模式和策略模式的目的非常相似，但两者的主要区别在于它们的结构和实现。两者都允许我们改变算法的某些部分，同时重用公共部分；然而，尽管策略模式允许我们在运行时动态地执行它，但使用模板模式完成算法是在具体类被定义的时候确定的。在这些假设下，模板模式可能更适合那些我们想要创建一个算法的预先打包的变体的情况。与往常一样，一种模式与另一种模式的选择取决于开发者，他们必须考虑每个用例的各种利弊。

### 使用模板模式的配置管理器
为了更好地了解模板模式和状态模式之间的区别，现在让我们重新实现我们在关于策略模式的章节中定义的`Config`对象，但是这次使用模板模式。就像以前版本的`Config`对象一样，我们希望能够使用不同的文件格式来加载和保存一组配置属性。

首先定义模板类，我们将其称为`ConfigTemplate`：

```javascript
const fs = require('fs');
const objectPath = require('object-path');
class ConfigTemplate {
  read(file) {
    console.log(`Deserializing from ${file}`);
    this.data = this._deserialize(fs.readFileSync(file, 'utf-8'));
  }
  save(file) {
    console.log(`Serializing to ${file}`);
    fs.writeFileSync(file, this._serialize(this.data));
  }
  get(path) {
    return objectPath.get(this.data, path);
  }
  set(path, value) {
    return objectPath.set(this.data, path, value);
  }
  _serialize() {
    throw new Error('_serialize() must be implemented');
  }
  _deserialize() {
    throw new Error('_deserialize() must be implemented');
  }
}
module.exports = ConfigTemplate;
```

新的`ConfigTemplate`类定义了两个模板方法：`_deserialize()`和`_serialize()`，它们是执行加载和保存配置所需的。 名称开头的下划线表示它们仅供内部使用，这是一种标记受保护方法的简单方法。由于在`JavaScript`中我们不能将方法声明为抽象方法，我们简单地将它们定义为存根，如果它们被调用（即，如果它们没有被具体子类覆盖）则抛出异常。

现在让我们使用我们的模板创建一个具体的类，例如，允许我们使用`JSON`格式加载和保存配置：

```javascript
const util = require('util');
const ConfigTemplate = require('./configTemplate');
class JsonConfig extends ConfigTemplate {
  _deserialize(data) {
    return JSON.parse(data);
  };
  _serialize(data) {
    return JSON.stringify(data, null, ' ');
  }
}
module.exports = JsonConfig;
```

`JsonConfig`类从我们的模板，`ConfigTemplate`类和
为`_deserialize()`和`_serialize()`方法提供了一个具体的实现。`JsonConfig`类现在可以作为独立的配置对象使用，而不使用
需要指定一个序列化和反序列化的策略，因为它是在类本身中实现的：

```javascript
const JsonConfig = require('./jsonConfig');

const jsonConfig = new JsonConfig();
jsonConfig.read('samples/conf.json');
jsonConfig.set('nodejs', 'design patterns');
jsonConfig.save('samples/conf_mod.json');
```

![](http://oczira72b.bkt.clouddn.com/18-1-28/17661306.jpg)

通过使用模板模式，我们可以通过重复使用从父模板类继承的逻辑和接口，仅提供一些抽象方法的实现，从而使我们能够获得一个全新的完全配置管理器。

### 实际应用场景
这种模式不应该听起来对我们来说是全新的。我们已经在`Chapter 5-Coding with Streams`时遇到过它，当我们扩展不同的`Streams`类来实现我们的自定义流。在这种情况下，模板方法是`_write()`，`_read()`，`_transform()`或`_flush()`方法，具体取决于我们想要实现的流类。要创建一个新的自定义流，我们需要从一个特定的抽象流类继承，为模板方法提供一个实现。

## 中间件模式（`Middleware`）
`Node.js`中最有特色的模式之一绝对是中间件模式。不幸的是，对于没有经验的人来说，这也是最令人困惑的事情之一，特别是来自企业架构的开发人员。疑惑的原因可能与中间件这个术语的含义有关，中间件在企业架构术语中表示各种软件套件，这些软件套件有助于抽象`OS API`，`网络通信`，`内存管理`等较底层的操作，允许开发人员只关注应用程序的商业案例。在这种情况下，中间件回顾了诸如`CORBA`，`Enterprise Service Bus`，`Spring`，`JBoss`等主题，但是在更通用的意义上，它也可以定义任何类型的软件层，它们在低级服务和应用程序字面上是中间的软件）。

### `Express`的中间件