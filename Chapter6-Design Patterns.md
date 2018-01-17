# Design Patterns
设计模式是重复出现的问题的可重用解决方案；该术语的定义非常广泛，可以涵盖应用程序的多个领域。然而，这个术语通常与著名的面向对象模式相关联，又被称作可复用的面向对象基础方法。我们经常会将这些特定的模式集合称为传统设计模式或`GoF`设计模式。

在`JavaScript`中应用面向对象的设计模式并不像传统的面向对象的语言那样线性和形式化。我们知道，`JavaScript`是范式化的，面向对象的，基于原型的，并且是动态类型语言；它将函数视为一等公民，并允许函数式的编程风格。这些特性使得`JavaScript`成为一种非常通用的语言，它为开发人员提供了巨大的力量，但同时也造成其编程风格与传统语言不同。人们总结`JavaScript`的编程范式，最后总结出`JavaScript`生态系统的模式。有很多方法可以使用`JavaScript`实现相同的结果。对于`JavaScript`的问题，解决一个问题的模式是多样化的。这种现象的一个明显的例子就是`JavaScript`生态系统中有丰富的框架和类库；可能没有其他语言见过这么多，尤其是现在`Node.js`已经给`JavaScript`带来了惊人的新的可能性，并创造了许多新的场景。

在这种背景下，传统的设计模式也受到`JavaScript`本质的影响。实现它们的方式有很多，所以它们传统的，强烈的面向对象的实现意味着它们不再是模式。在某些情况下，它们甚至是不需要的，因为我们知道，`JavaScript`没有真正的类或抽象接口。不变的是每个模式的基本原理，解决的问题以及解决方案核心的概念。

本章探讨的设计模式如下：

* 工厂模式（`Factory`）
* 揭示模块模式（`Revealing constructor`）
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

