# Writing Modules
`Node.js`模块系统弥补了原生`JavaScript`缺乏把代码组织到不同独立单元的这一缺陷。模块系统最大的优点就是能够使用`require()`函数将模块链接在一起，这是一种简单而强大的方法。但是，对于许多新的`Node.js`的开发人员可能会对模块系统的使用产生疑问。实际上，最常见的问题之一是：将组件X的实例传递到模块Y的最佳方式是什么？

有时候，这种疑问可能导致我们滥用单例模式，因为希望找到一种更熟悉的方式来将我们的模块链接在一起。另一方面，我们可能滥用依赖注入模式，利用它来处理任何类型的依赖（甚至无状态）。如果说如何组织模块是`Node.js`中最具争议性和观点性的话题之一应该不足为奇了。主流的组织模块方式很多，但没有任意一个观点处于主导地位。但实际上，每种方法都有其优点和缺点。

在本章中，我们将分析组织模块的各种方法，并强调它们的优缺点，以便我们能够在简单性，可重用性和可扩展性之间平衡，合理地选择和混用这些模块组织方式。具体来说，我们将介绍一些模式，如下所示：

* 硬编码依赖
* 依赖注入
* 服务定位器
* 依赖注入容器

然后，我们将探讨一个与书写模块密切相关的问题，即如何组织`Node.js`插件模块。对于这个问题，大多数书写插件模块的方式都差不多，但是与用户自己编写的应用程序模块的组织就不太相同了，特别是当插件作为单独的`Node.js`包分发时，问题就十分明显了。 

我们将学习如何构建一个`Node.js`插件，并如何把这些插件集成到主应用程序中。

在本章最后，对于`Node.js`如何组织模块就不再是晦涩难懂的话题了。

## 模块和依赖
每个应用程序都是多个模块组织在一起的结果，如同盖楼一样，随着应用程序日益迭代复杂，我们组织模块的方式将导致应用程序的成功或失败。这不仅与应用程序的拓展性相关，还是我们构建大型系统的重点关注点。过于复杂紊乱的模块依赖是一种灾难，它增加了我们项目的组织难度，在这种情况下，代码的任何修改和拓展都将会使我们付出巨大的代价。

最糟糕的情况是，这些模块严重耦合，导致我们不重写整个应用程序就不更改代码的任何一部分。当然，不必害怕，我们并不用从写第一个模块开始就开始全面规划我们的模块。但只要我们遵循应有的模式，就不会出现这样的问题。

`Node.js`提供了一个很好的工具来连接和组织应用程序。那就是`CommonJS`模块系统。但是，使用模块系统并不能够保证我们我们一定能解决模块依赖的问题，如果使用不当，将会使得耦合变得更加严重。在本节中，我们将讨论书写`Node.js`模块的基本模式。

### Node.js最常见的依赖
在一个软件体系结构中，我们在设计其的过程中就应该考虑到可能影响其中任何一个组件依赖关系的实体、状态、数据格式。例如，一个组件可能使用另一个组件的提供的服务，也可能依赖系统特定的一个全局状态，或者实现一个特定的通信协议，以便与其他组件交换信息等等。依赖的概念十分广泛，有时会显得难以评估。

但是，在`Node.js`中，我们可以确定一个最常见也最容易识别的最基本的依赖模型。当然，当我们在讨论模块之间的依赖关系，我们应该首先明确：模块是我们组织和构建代码的基本机制。不依赖模块系统构建的大型应用程序是十分不合理的。如果使用正确的方式来组织应用程序的各个模块单元，它会带来很多好处。实际上，一个模块的属性可以概括如下：

* 一个模块应该具有可读性和可理解性，因为它应该专注于一件事
* 一个模块被表示为一个单独的文件，使得其更容易被识别
* 模块可以更容易地在不同的应用程序中复用

一个模块代表的是一个完全私有的命名空间，并通过`module.exports`来公开访问这个模块的接口。

但是，对于一个成功的模块设计，只是简单地将应用程序或库的功能区分为不同的模块是完全不够的。最常见的错误会出现在我们创建了一个过于复杂的模块，那么想要替换或更改这个模块会对整个应用的架构产生巨大的影响。这时就能够意识到把代码组织成模块的优势了。我们需要在模块设计中找到一个平衡点。

### 内聚与耦合
评判创建的模块平衡性两个最重要的特征就是内聚度和耦合度。这两个特征可以应用于软件体系结构中的任何类型的组件或子系统。因此在构建`Node.js`模块时也可以把这两个特征作为重要的参考价值。这两个属性定义如下：

* 内聚度：用于度量模块内部功能之间的相关性。例如，对于一个只做一件事的模块，其中的所有部件都只对这一件事起作用，那说明这个模块具有很高的内聚度。举个例子，那种包含把任何类型的对象存储到数据库的函数内聚度就较低，如`saveProduct()`、`saveInvoice()`、`saveUser()`等。

* 耦合度：评判模块对系统其他模块的依赖程度。例如，当一个模块直接读取或修改另一个模块的数据时，该模块与另一个模块紧密耦合。另外，通过全局或共享状态交互的两个模块紧密耦合。另一方面，仅通过参数传递进行通信的两个模块耦合度较低。

理想情况下，一个模块应该具有较高的内聚度和较低的耦合度，这样的模块更易于理解、重用和扩展。

### 有状态模块
在`JavaScript`中，一切都是对象。它没有纯粹的类或者接口的概念，因为其动态类型的机制，已经将接口或者策略和实现细节分开。这就是为什么我们在`Chapter 6-Design Patterns`看到在`JavaScript`中一些设计模式和传统的设计模式看起来如此不同并且简单的多的原因。

在JavaScript中，将接口与实现分离的例子很少。 然而，通过使用`Node.js`模块系统，我们引入了一个特定的模块，接口不会受到其它模块的影响。在正常情况下，这没有什么问题，但是如果我们使用`require()`来加载一个导出有状态实例的模块，比如数据库交互对象，HTTP服务器实例，乃至普通的任何对象这不是无状态的，我们实际上是在引用的模块都是一个又一个的单例，因此模块系统有着单例模式的优点和缺点，此外，也有一些不同的地方。

#### Node.js的单例模式
很多刚接触`Node.js`的人对于如何正确地实现单例模式感到困惑，通常情况下，应用程序的各个模块之间共享一个实例。`Node.js`中要想实现这一点特别简单；只需使用`module.exports`导出实例就足以获得与`Singleton`模式非常相似的效果。

例如，考虑下面这行代码：

```javascript
//'db.js' module
module.exports = new Database('my-app-db');
```

通过导出`Database`的一个实例，我们可以假定在当前包（这可以很容易地成为我们应用程序的整个代码）内，我们将只有一个`db`模块的实例。这是可能的，因为我们知道，`Node.js`将在第一次调用`require()`之后缓存模块，确保在随后的调用中不再执行它，而是返回缓存实例。例如，我们可以很容易地获得我们之前定义的`db`模块的一个共享实例，使用下面这行代码：

```javascript
const db = require('./db');
```

但是注意，该模块使用的是相对路径引入，因此其是符合单例模式的。我们在`Chapter2-Node.js Essential Patterns`中看到，每个包在其`node_modules`目录中都可能有自己的一组专用依赖项，这可能会导致同一个模块会有多个实例，例如，考虑将`db`模块封装到名为`mydb`的包中的情况。看以下代码`package.json`文件中的代码：

```json
{
  "name": "mydb",
  "main": "db.js"
}
```

现在考虑下面的依赖包的关系树：

```
app/
   `-- node_modules
       |-- packageA
       |  `-- node_modules
       |      `-- mydb
       `-- packageB
           `-- node_modules
               `-- mydb
```

`packageA`和`packageB`都依赖于`mydb`模块；反过来，其它的应用程序模块，可能同时依赖于`packageA`和`packageB`。 我们刚刚描述的场景将打破关于数据库实例唯一性的假设；实际上，`packageA`和`packageB`都将使用如下命令加载`db`实例：

```javascript
const db = require('mydb');
```

然而，`packageA`和`packageB`实际上会加载两个不同的单例，因为`mydb`模块将根据所需的包来解析到不同的目录。

在这一点上，我们可以很容易地说，除非我们使用真正的全局变量来存储一个模块实例，否则之前描述的单例模式在`Node.js`中不存在，如下所示：

```javascript
global.db = new Database('my-app-db');
```

这将保证该实例将是唯一的，并在整个应用程序中共享，仅仅是在一个模块中。但是，我们应该尽量避免这么做。在大多数情况下，我们并不需要一个纯粹的单例模式，无论如何，我们稍后会看到，还有其他模式可以用来在不同的包中共享一个实例。

> 在本书中，为了简单起见，我们将使用术语单例模式来描述由模块导出的有状态对象，即使这并不代表严格定义的单一实例。但是，我们可以肯定地说，它与原始的单例模式具有相同的含义：可以在不同的组件之间共享状态。

## 书写模块的模式
现在我们已经讨论了一些关于内聚和耦合的基本理论，我们已经准备好了一些更实际的概念。实际上，在这一节中，我们将介绍怎么书写模块。我们重点讲解如何利用有状态模块实例，毫无疑问，它是应用程序中最重要的一类依赖。

### 硬编码依赖
我们开始通过分析两个模块之间最常见的关系来看硬编码依赖。在`Node.js`中，当一个客户端模块使用`require()`加载另一个模块时就会建立模块的硬编码依赖关系。正如我们将在本节中看到的，这种建立模块依赖关系的方法简单而有效，但是我们必须更加关注有状态实例的硬编码依赖关系，否则在有状态实例模块会限制我们的模块复用。

#### 使用硬编码的依赖关系构建鉴权服务
我们从下图所示的结构开始分析：

![](http://oczira72b.bkt.clouddn.com/18-1-12/48304376.jpg)

上图显示了分层体系结构的典型示例；它描述了一个简单的鉴权服务的结构。`AuthController`接受来自客户端的输入，从请求中提取登录信息，并执行一些初步验证。之后`AuthService`检查客户端提供的凭证是否与存储在数据库中的信息匹配；这是通过使用`db`模块执行一些特定的查询来完成的，作为与数据库通信的一种手段。这三个组件连接在一起的方式将决定它们的可重用性，可测试性和可维护性的强度。

将这些组件连接在一起的最自然的方法是通过`AuthService`请求`db`模块，然后从`AuthController`请求`AuthService`。 这是我们正在讨论的硬编码依赖。

让我们通过实际实现刚刚描述的系统来演示这一点。那么我们来设计一个简单的鉴权服务器，它将有以下两个`HTTP API`：

* `POST '/ login'`：接收包含用户名和密码对进行身份验证的`JSON`对象。 成功时，它会返回一个`JSON Web Token（JWT）`，随后的请求中使用它来验证用户的身份。

> `JSON Web Token`是一种客户端和服务端身份验证的格式。但随着单页应用程序和跨源资源共享（CORS）技术的增长，基于`cookie`的身份验证的更为灵活的替代方案，其受欢迎程度正在不断提高。要了解更多关于`JSON Web Token`的信息，可以参考http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html上的规范

* `GET'/ checkToken'`：查看用户是否具有权限。

对于这个例子，我们将使用几种技术；其中一些对我们来说并不陌生。我们使用[express](https://npmjs.org/package/express)来实现`Web API`和[levelup](https://npmjs.org/package/levelup)来存储用户的数据。


##### db模块
我们先从底层开始构建应用程序；我们需要的第一件事就是公开一个`levelUp`数据库实例的模块。 我们来创建一个名为`lib/db.js`的新文件，其中包含以下内容：

```javascript
const level = require('level');
const sublevel = require('level-sublevel');
module.exports = sublevel(
  level('example-db', {
    valueEncoding: 'json'
  })
);
```

前面的模块只是创建一个到存储在`./example-db`目录中的`LevelDB`数据库的连接，然后使用[sublevel](https://npmjs.org/package/level-sublevel)来修饰实例，该插件添加了支持增删查改数据库（可以将其与`SQL`或`MongoDB`进行比较）。模块导出的对象是数据库对象本身，它是一个有状态的实例；因此，我们创建的是单例。

##### authService模块
现在我们有了`db`单例，我们可以使用它来实现`lib/authService.js`模块，它负责查询数据库，根据用户身份凭证查看用户是否具有权限。 代码如下（只显示相关部分）：

```javascript
"use strict";

const jwt = require('jwt-simple');
const bcrypt = require('bcrypt');

const db = require('./db');
const users = db.sublevel('users');

const tokenSecret = 'SHHH!';

exports.login = (username, password, callback) => {
  users.get(username, (err, user) => {
    if(err) return callback(err);
    
    bcrypt.compare(password, user.hash, (err, res) => {
      if(err) return callback(err);
      if(!res) return callback(new Error('Invalid password'));
      
      let token = jwt.encode({
        username: username,
        expire: Date.now() + (1000 * 60 * 60) //1 hour
      }, tokenSecret);
      
      callback(null, token);
    });
  });
};

exports.checkToken = (token, callback) => {
  let userData;
  try {
    //jwt.decode will throw if the token is invalid
    userData = jwt.decode(token, tokenSecret);
    if (userData.expire <= Date.now()) {
      throw new Error('Token expired');
    }
  } catch(err) {
    return process.nextTick(callback.bind(null, err));
  }
    
  users.get(userData.username, (err, user) => {
    if (err) return callback(err);
    callback(null, {username: userData.username});
  });
};

```

`authService`模块实现`login()`服务，该服务负责查询数据库，检查用户名和密码信息，`checkToken()`服务接受`token`作为参数并验证其有效性。

上面的代码是有状态模块的硬编码依赖关系的第一个示例。我们正在谈论`db`模块，我们只需要加载它。生成的`db`变量包含一个已经初始化的数据库对象，我们可以直接使用它来执行我们的查询。

在这一点上，我们可以看到，我们为`authService`模块创建的所有代码并不需要`db`模块的一个特定实例，任何实例都可以正常发挥作用。但是，`authService`模块硬编码依赖于`levelUp`数据库对象实例，这意味着我们将无法在不更改其模块本身代码的情况下将`authService`与另一个数据库实例结合使用。

##### authController模块
继续在应用程序的层次上，我们现在要看看`lib/authController.js`模块。这个模块负责处理`HTTP`请求，它本质上是`Express`路由的集合；该模块的代码如下：

```javascript
"use strict";

const authService = require('./authService');

exports.login = (req, res, next) => {
  authService.login(req.body.username, req.body.password,
    (err, result) => {
      if (err) {
        return res.status(401).send({
          ok: false,
          error: 'Invalid username/password'
        });
      }
      res.status(200).send({ok: true, token: result});
    }
  );
};

exports.checkToken = (req, res, next) => {
  authService.checkToken(req.query.token,
    (err, result) => {
      if (err) {
        return res.status(401).send({
          ok: false,
          error: 'Token is invalid or expired'  
        });
      }
      res.status(200).send({ok: 'true', user: result});
    }
  );
};
```

`authController`模块实现两个`Express`路由：`login()`用于执行登录操作并返回相应的`token`，`checkToken()`用于检查`token`的有效性。这两个路由委托他们的大部分逻辑到`authService`，所以他们唯一的工作是处理`HTTP`请求和响应。

我们也可以看到，在这种情况下，我们使用有状态模块`authService`来硬编码依赖项。是的，`authService`模块通过传递性是有状态的，因为它直接依赖于`db`模块。 有了这个，我们理解了硬编码的依赖关系如何贯穿整个应用程序的结构中：`authController`模块依赖于`authService`模块，而`authService`模块依赖于`db`模块；这意味着`authService`模块本身是间接链接到一个特定的数据库实例的。

##### app模块
最后，在应用程序的入口点，我们调用我们的`controller`。遵循约定，我们将把这个逻辑放在名为`app.js`的模块中，放在我们项目的根目录下，如下所示：

```javascript
"use strict";

const Express = require('express');
const bodyParser = require('body-parser');
const errorHandler = require('errorhandler');
const http = require('http');

const authController = require('./lib/authController');

let app = module.exports = new Express();
app.use(bodyParser.json());

app.post('/login', authController.login);
app.get('/checkToken', authController.checkToken);

app.use(errorHandler());
http.createServer(app).listen(3000, () => {
  console.log('Express server started');
});
```

我们可以看到，我们的应用程序模块是非常基础的。 它包含一个简单的`Express`服务器，它注册了一些中间件和`authController`导出的两条路由。 当然，对于我们来说最重要的代码是`authController`所导出的硬编码依赖实例。

##### 运行鉴权服务
在我们尝试我们刚刚实现的认证服务器之前，我们建议您使用代码示例中提供的`populate_db.js`脚本来填充数据库中的一些示例数据。 这样做之后，我们可以通过运行以下命令来启动服务器：

```bash
node app
```

然后我们可以尝试调用我们创建的两个`Web`服务; 我们可以使用`REST`客户端来执行此操作，或者使用旧的`curl`命令。 例如，要执行登录，我们可以运行以下命令：

```bash
curl -X POST -d '{"username": "alice", "password":"secret"}' http://localhost:3000/login -H "Content-Type: application/json"
```

前面的命令应该返回一个`token`，我们可以使用它来测试 `/checkLogin`的Web服务（只需输入以下命令并替换`<TOKEN HERE>`）：

```bash
curl -X GET -H "Accept: application/json" http://localhost:3000/checkToken?token=<TOKEN HERE>
```

前面的命令应该返回一个字符串，如下所示，这确认我们的服务器正在按预期工作：

```json
{"ok":"true","user":{"username":"alice"}}
```

#### 硬编码依赖的优点和缺点
我们刚刚实现的示例演示了`Node.js`中书写模块的传统方式以及利用模块系统的全部功能来管理应用程序各个组件之间的依赖关系。我们从模块中导出有状态的实例，让`Node.js`管理它们的生命周期，然后我们直接从应用程序的其他部分引入它们。这样管理起来非常直观，易于理解和调试，每个模块初始化和引入，都不会受到任何外部条件的干预。

然而，另一方面，对有状态实例的依赖性进行硬编码会限制将模块与其他实例关联的可能性，这使得在单元测试的过程中，其可重用性更低，测试难度更大。例如，将`authService`与其他数据库实例结合使用几乎是不可能的，因为它的依赖关系是用一个特定的实例进行硬编码的。同样，单独测试`authService`可能是一件困难的事情，因为我们不能轻易地模拟另一模块使用数据库。

最后，重要的是要看到使用硬编码依赖的大多数缺点都与有状态的实例相关联。这意味着如果我们使用`require()`来加载一个无状态模块，例如一个工厂，构造函数或者一组无状态函数，我们就不会遇到同样的问题。我们仍然会与特定的实现紧密耦合，但在`Node.js`中，这通常不会影响组件的可重用性，因为在模块内部创建的实例不会引入与特定状态的耦合。

### 依赖注入
依赖注入（DI）模式可能是软件设计中最容易被误解的概念之一。许多人将这个术语与框架和依赖注入容器相关联，例如`Spring`（用于`Java`和`C#`）或`Pimple`（用于`PHP`），但实际上它是一个很简单的概念。依赖注入模式背后的主要思想是由外部实体提供输入的组件的依赖关系。

这样的实体可以是客户端组件或全局容器，它集中了系统所有模块的关联。这种方法的主要优点是解耦，特别是对于取决于有状态实例的模块。使用DI，从外部接收每个依赖项，而不是硬编码到模块中。这意味着模块可以配置为其中的依赖关系，因此可以在不同的上下文中重用。

为了在实践中演示这种模式，我们现在要重构我们在前一节中构建的鉴权服务器，使用DI来连接它的模块。

#### 使用DI重构鉴权服务器
使用DI重构我们的模块是很简单的：我们不需要将依赖关系硬编码到有状态实例，而是创建一个工厂，它将一组依赖作为参数。

让我们立即开始这个重构; 让我们来看看如下的`lib/db.js`模块：

```javascript
"use strict";

const level = require('level');
const sublevel = require('level-sublevel');

module.exports = function(dbName) {
  return sublevel(
    level(dbName, {valueEncoding: 'json'})
  );
};
```

重构过程的第一步是将`db`模块转换为工厂模式。结果是我们现在可以使用它创建尽可能多的数据库实例，这意味着整个模块现在可以重用和无状态。

我们继续并实现新版本的`lib/authService.js`模块：

```javascript
"use strict";

const jwt = require('jwt-simple');
const bcrypt = require('bcrypt');

module.exports = (db, tokenSecret) => {
  const users = db.sublevel('users');
  const authService = {};
  
  authService.login = (username, password, callback) => {
    users.get(username, (err, user) => {
      if(err) return callback(err);
      
      bcrypt.compare(password, user.hash, (err, res) => {
        if(err) return callback(err);
        if(!res) return callback(new Error('Invalid password'));
        
        const token = jwt.encode({
          username: username,
          expire: Date.now() + (1000 * 60 * 60) //1 hour
        }, tokenSecret);
        
        callback(null, token);
      });
    });
  };

  authService.checkToken = (token, callback) => {
    let userData;
    try {
      //jwt.decode will throw if the token is invalid
      userData = jwt.decode(token, tokenSecret);
      if (userData.expire <= Date.now()) {
        throw new Error('Token expired');
      }
    } catch(err) {
      return process.nextTick(callback.bind(null, err));
    }
      
    users.get(userData.username, (err, user) => {
      if(err) return callback(err);
      callback(null, {username: userData.username});
    });
  };
  
  return authService;
};
```

此外，`authService`模块现在是无状态的; 它不再导出任何特定的实例，只是一个简单的工厂。 但最重要的细节是，我们将`db`依赖注入作为工厂函数的一个参数，删除以前的硬编码依赖。这个简单的更改使我们能够通过将它连接到任何数据库实例来创建一个新的`authService`模块。

我们可以用类似的方式重构`lib/authController.js`模块，如下所示：

```javascript
"use strict";

module.exports = (authService) => {
  const authController = {};
  
  authController.login = (req, res, next) => {
    authService.login(req.body.username, req.body.password,
      (err, result) => {
        if (err) {
          return res.status(401).send({
            ok: false,
            error: 'Invalid username/password'
          });
        }
        res.status(200).send({ok: true, token: result});
      }
    );
  };

  authController.checkToken = (req, res, next) => {
    authService.checkToken(req.query.token,
      (err, result) => {
        if (err) {
          return res.status(401).send({
            ok: false,
            error: 'Token is invalid or expired'  
          });
        }
        res.status(200).send({ok: 'true', user: result});
      }
    );
  };
  
  return authController;
};
```

`authController`模块根本没有任何硬编码依赖，甚至没有状态。唯一的依赖`authService`模块在调用时作为输入提供给工厂。

好吧，现在是时候看看所有这些模块是在哪里创建和连接在一起的。 答案在于`app.js`模块，它代表了我们应用程序中的最顶层。其代码如下：

```javascript
"use strict";

const Express = require('express');
const bodyParser = require('body-parser');
const errorHandler = require('errorhandler');
const http = require('http');

const app = module.exports = new Express();
app.use(bodyParser.json());

const dbFactory = require('./lib/db');
const authServiceFactory = require('./lib/authService');
const authControllerFactory = require('./lib/authController');

const db = dbFactory('example-db');
const authService = authServiceFactory(db, 'SHHH!');
const authController = authControllerFactory(authService);

app.post('/login', authController.login);
app.get('/checkToken', authController.checkToken);

app.use(errorHandler());
http.createServer(app).listen(3000, () => {
  console.log('Express server started');
});
```

前面的代码可以概括如下：

1. 我们加载`services`的工厂；在这一点上，其仍然是无状态的对象。
2. 我们通过引入它所需的依赖来实例化每个服务。这是模块创建和链接的阶段。
3. 最后，我们像往常一样在`Express`服务器上注册`authController`模块的路由。

鉴权服务器现在使用`DI`链接，提高了其复用性。

#### DI的不同类型
我们刚刚介绍的例子只演示了一种类型的`DI`（工厂注入），但是还有一些类型的`DI`更值得一提：

* 构造函数注入：在这种类型的`DI`中，依赖关系在创建时传递给构造函数；一个可能的例子可以是：

```javascript
const service = new Service(dependencyA, dependencyB);
```

* 属性注入：在这种类型的`DI`中，依赖关系在创建之后附加到对象上，如以下代码所示：

```javascript
const service = new Service();
service.dependencyA = anInstanceOfDependencyA;
```

属性注入意味着一个对象会被创建为不一致的状态，因为它没有连接到它的依赖关系，所以它是最不健壮的，但是当依赖关系之间存在循环时，它有时可能是有用的。例如，如果我们有两个组件A和B，它们都使用工厂或构造函数注入，并且都相互依赖，我们不能实例化它们中的任何一个，因为两者都需要另一个存在才能被创建。我们来看一个简单的例子，如下所示：

```javascript
function Afactory(b) {
  return {
    foo: function() {
      b.say();
    },
    what: function() {
      return 'Hello!';
    }
  }
}

function Bfactory(a) {
  return {
    a: a,
    say: function() {
      console.log('I say: ' + a.what);
    }
  }
}
```

前两个工厂之间的依赖关系死锁只能通过属性注入来解决，例如先创建一个不完整的`B`实例，然后才能创建`A`。最后，我们将`A`注入到`B`中，方法是设置相关属性 如下：

```javascript
const b = Bfactory(null);
const a = Afactory(b);
a.b = b;
```

> 在极少数情况下，依赖图中的循环是不容易避免的; 然而，重要的是要记住，这往往是一个糟糕的设计，应该尽可能避免。

#### DI的优点和缺点
在使用`DI`的鉴权服务器示例中，我们能够将我们的模块与特定的依赖项实例分离。结果是，我们现在可以用最少的代价复用每个模块，而且代码没有任何改变。测试使用`DI`模式的模块也大大简化；我们可以轻松地模拟模块的依赖关系，并且独立于系统其他部分的状态来测试我们的模块。

我们前面介绍的例子中要强调的另一个重要方面是，我们将依赖链接的地方从底层移到了顶层。

这个想法是，高级组件在本质上比低级组件更不易重复使用，这是因为我们在应用程序的层次越多，组件越具体。

基于这个假设，那么高级组件底层依赖关系的应用程序架构的顺序是可以颠倒的，这样底层组件只依赖于一个接口（在`JavaScript`中，它是只是我们期望的一个依赖的接口），而定义一个依赖的实现的所有权是给予更高级别的组件的。在我们的鉴权服务器中，实际上，所有的依赖关系都被实例化，并被连接到最上面的组件，即我们的应用程序模块（`app.js`），这也是不太可重用的，并且耦合度较高。

所以耦合度和复用性是相悖的。通常，如果编码时无法解决依赖关系，理解系统各个组件之间的关系就会变得更困难。另外，如果我们看一下我们在应用程序模块中实例化所有依赖的方式，我们可以看到我们必须遵循特定的顺序。我们实际上不得不手动构建整个应用程序的依赖关系图。当要链接的模块数量变多时，这可能变得难以管理。

解决这个问题的一个可行的解决方案是在多个组件之间拆分依赖，而不是集中在一个地方。这可以减少涉及管理依赖关系的复杂度，因为每个组件只负责其特定的依赖关系子图。当然，我们也可以选择仅在本地使用`DI`，只是在必要时使用`DI`，而不是在整个应用程序之上构建。

我们将在本章后面看到，另一种简化复杂体系结构中模块连接的可能解决方案是使用一个DI容器，一个专门负责实例化和连接应用程序所有依赖关系的组件。

使用`DI`肯定会增加我们模块的复杂性和冗长度，但正如我们前面所看到的，这样做有很多好的理由。取决于我们想要获得的简单性和可重用性之间的平衡，至于选择依赖注入还是选择硬编码依赖，则取决于我们。

> DI经常结合`Dependency Inversion principle（依赖倒置准则）` 和 `Inversion of Control（控制反转）`一并讨论; 然而，他们虽然相关，但却是不同的概念。

### 服务定位器
在前面的章节中，我们学习了`DI`如何通过获得可重用和解耦的模块连接依赖关系。与这一模式相类似的另一种模式是服务定位器。服务定位器核心原则是拥有一个中央注册中心，以便管理系统组件，并在模块需要加载依赖时作为中介。这个想法是要求服务定位器所连接的是依赖注入模块，而不是硬编码模块。

理解这一点很重要，通过使用服务定位器，我们引入了对它的依赖关系，它连接到模块的方式决定了它们的耦合程度，其可重用性较高。 在`Node.js`中，我们可以确定三种类型的服务定位器，区分它们的关键因素是它们连接到系统各个组件的方式：

* 硬编码依赖服务定位器
* 依赖注入服务定位器
* 全局注入服务定位器

硬编码依赖服务定位器耦合度较高，因为它由使用`require()`直接引入服务定位器的实例组成。在`Node.js`中，这可以被认为是一种反模式，因为它引入了一个紧密耦合的组件。在这种情况下，服务定位器在重用性方面显然没有提供任何价值，只是增加了另一层级的间接性和复杂性。因此应该抛弃硬编码依赖服务定位器这种模块引入方式。

依赖注入服务定位器通过`DI`引用组件。这可以被认为是一次注入一整套依赖的更方便的方法，而不是一个接一个地提供它们。而且我们将看到它的优势并不止于此。

全局注入服务定位器直接注入到全局。这与硬编码服务定位器具有相同的缺点，但由于它是全局的，因此它是一个真正的单例，因此可以很容易地用作包之间共享实例的模式。我们将在后面的章节中看到这一点，但现在我们可以肯定地说，全局注入服务定位器使用场景更少。

> `Node.js`模块系统已经实现了服务定位器模式的变体，其中`require()`代表服务定位器本身的全局实例。

一旦我们开始使用服务定位器模式，上述所说的将变得更加清晰。现在重构鉴权服务器来实践服务定位器。

#### 使用服务定位器重构鉴权服务
我们现在要使用服务定位器重构鉴权服务器。要做到这一点，第一步是实现服务定位器本身；我们将使用一个新的模块`lib/serviceLocator.js`：

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