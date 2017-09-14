---
layout: docs
title:  "Nest"
categories: nest
toc: true
permalink: /wiki/nest
---


[![nest](/assets/img/nest.png)](http://nestjs.com/) [![github](/assets/img/github.svg)](https://github.com/kamilmysliwiec/nest)

Nest 是一个强大的 Node.js Web 框架，可以帮助你轻松地构建高效，可扩展的应用程序。它采用现代 JavaScript，基于 TypeScript 构建，并结合了 OOP（面向对象编程）和 FP （函数式编程）的最佳概念。

它不仅是又一个框架。你不必等待一个大型的社区，因为 Nest 建立在著名仓库 Express 和 socket.io 之上。这意味着，你可以快速开始使用框架，而不必担心第三方插件的缺失。

https://segmentfault.com/a/1190000009573441




## 组件(Components)

几乎所有的东西都是组件，`Service`，`Repository`，`Provider`等等。并且他们可以通过构造函数注入控制器或另一组件。

实际上，`UsersService`应该从持久层调用适当的方法，例如，`UsersRepository`组件。我们没有任何类型的数据库，所以我们再次使用假数据。
```js
import { Component } from '@nestjs/common';
import { HttpException } from '@nestjs/core';

@Component()
export class UsersService {
    private users = [
        { id: 1, name: "John Doe" },
        { id: 2, name: "Alice Caeiro" },
        { id: 3, name: "Who Knows" },
    ];
    getAllUsers() {
        return Promise.resolve(this.users);
    }
    getUser(id: number) {
        const user = this.users.find((user) => user.id === id);
        if (!user) {
            throw new HttpException("User not found", 404);
        }
        return Promise.resolve(user);
    }
    addUser(user) {
        this.users.push(user);
        return Promise.resolve();
    }
}
```
Nest 组件是一个简单的类，使用`@Component()`注释。

在 `getUser()` 方法中可以看到，我们使用了 `HttpException`。它是 Nest 内置的异常，拥有两个参数，错误消息和状态代码。创建域异常是一个很好的做法，它应该扩展 `HttpException` （更多见错误处理章节）。

我们的服务准备好了。让我们在 `UsersController` 中使用它。
```js
@Controller('users')
export class UsersController {
    constructor(private usersService: UsersService) {}

    @Get()
    getAllUsers(@Response req) {
        this.usersService.getAllUsers()
            .then((users) => res.status(HttpStatus.OK).json(users));
    }

    @Get('/:id')
    getUser(@Response() res, @Param('id') id) {
        this.usersService.getUser(+id)
            .then((user) => res.status(HttpStatus.OK).json(user));
    }

    @Post()
    addUser(@Response() res, @Body('user') user) {
        this.usersService.addUser(req.body.user)
            .then((msg) => res.status(HttpStatus.CREATED).json(msg));
    }
}
```
如图所示，`UsersService`将被注入到构造函数中。

如果你不是 TypeScript 爱好者，并且使用纯 JavaScript，则必须按以下方式执行：
```js
import { Dependencies, Controller, Get, Post, Response, Param, Body, HttpStatus } from '@nestjs/common';

@Controller('users')
@Dependencies(UsersService)
export class UsersController {
    constructor(usersService) {
        this.usersService = usersService;
    }

    @Get()
    getAllUsers(@Response() res) {
        this.usersService.getAllUsers()
            .then((users) => res.status(HttpStatus.OK).json(users));
    }

    @Get('/:id')
    getUser(@Response() res, @Param('id') id) {
        this.usersService.getUser(+id)
            .then((user) => res.status(HttpStatus.OK).json(user));
    }

    @Post()
    addUser(@Response() res, @Body('user') user) {
        this.usersService.addUser(user)
            .then((msg) => res.status(HttpStatus.CREATED).json(msg));
    }
}
````
因为 Nest 不知道有关 `UsersService` 的任何内容。此组件不是 `ApplicationModule` 的一部分，我们必须在那里添加：
```js
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
    controllers: [ UsersController ],
    components: [ UsersService ],
})
export class ApplicationModule {}
```
现在我们的应用程序将执行，但仍然有一个路由不能正常工作，就是 `addUser` 。为什么？因为我们正在尝试解析请求体（`req.body.user`），而没有使用 `express` 的 `body-parser` 中间件。正如你知道的，可以通过 express 实例作为 `NestFactory.create()` 方法的第二个参数。
让我们安装插件：
```bash
npm install --save body-parser
```
然后在我们的 express 实例中设置它。
```js
import express from 'express';
import * as bodyParser from 'body-parser';
import { NestFactory } from '@nestjs/common';
import { ApplicationModule } from './modules/app.module';

const instance = express();
instance.use(bodyParser.json());

const app = NestFactory.create(ApplicationModule, instance);
app.listen(3000, () => console.log('Application is listening on port 3000'));
```

### Async / await
Nest 与 ES7 的 `async` / `await` 功能兼容。因此我们可以快速重写我们的 `UsersController` ：
```js
@Controller('users')
export class UsersController {
    constructor(private usersService: UsersService) {}

    @Get()
    async getAllUsers(@Response() res) {
        const users = await this.usersService.getAllUsers();
        res.status(HttpStatus.OK).json(users);
    }

    @Get('/:id')
    async getUser(@Response() res, @Param('id') id) {
        const user = await this.usersService.getUser(+id);
        res.status(HttpStatus.OK).json(user);
    }

    @Post()
    async addUser(@Response() res, @Body('user') user) {
        const msg = await this.usersService.getUser(user);
        res.status(HttpStatus.CREATED).json(msg);
    }
}
```
看起来更好么？在这里你可以阅读更多关于 `async` / `await。`

## 模块(Modules)

模块是一个带有 `@Module({})` 装饰器的类。该装饰器提供元数据，该框架用于组织应用程序结构。
  现在，这是我们的 `ApplicationModule`：
```js
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
    controllers: [ UsersController ],
    components: [ UsersService ],
})
export class ApplicationModule {}
```
默认情况下，模块封装每个依赖关系。这意味着不可能在模块之外使用其组件或控制器。

每个模块也可以导入到另一个模块。实际上，你应该将 Nest 模块看做是 模块树。

我们将 `UsersController` 和 `UsersService` 移动到 `UsersModule`。只需创建新文件，例如，`users.module.ts` 包含以下内容：
```js
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
    controllers: [ UsersController ],
    components: [ UsersService ],
})
export class UsersModule {}
```
然后将 `UsersModule` 导入到 `ApplicationModule` （我们的主应用程序）：
```js
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';

@Module({
    modules: [ UsersModule ]
})
export class ApplicationModule {}
```
### 依赖注入

模块可以轻松地注入组件，看起来如下：
```js
@Module({
    controllers: [ UsersController ],
    components: [ UsersService, ChatGateway ],
})
export class UsersModule implements NestModule {
    constructor(private usersService: UsersService) {}
}
```
此外，组件还可以注入模块：
```js
export class UsersController {
    constructor(private module: UsersModule) {}
}
```

可以看到出，使用 Nest 可以将代码自然地拆分成可分离和可重用的模块。

## 中间件(Middlewares)

中间件是一个在路由处理程序前被调用的函数。中间件函数可以访问请求和响应对象，因此可以修改它们。它们也可以向一个屏障，如果中间件函数不调用 `next()`，请求将永远不会被路由处理程序处理。
让我们来构建一个虚拟授权中间件。我们将使用 `X-Access-Token` HTTP 头来提供用户名（这是个奇怪的想法，但是不重要）。
```js
import { UsersService } from './users.service';
import { HttpException } from '@nestjs/core';
import { Middleware, NestMiddleware } from '@nestjs/common';

@Middleware()
export class AuthMiddleware implements NestMiddleware {
    constructor(private usersService: UsersService) {}

    resolve() {
        return (req, res, next) => {
            const userName = req.headers["x-access-token"];
            const users = this.usersService.getUsers();

            const user = users.find((user) => user.name === userName);
            if (!user) {
                throw new HttpException('User not found.', 401);
            }
            req.user = user;
            next();
        }
    }
}
```
一些事实如下：

- 你应该使用 `@Middleware()` 注解来告诉 Nest，这个类是一个中间件，
- 你可以使用 `NestMiddleware` 界面，这强制你使用 `resolve()` 方法，
- 中间件（与组件相同）可以通过其构造函数的注入依赖项（依赖关系必须是模块的一部分），
- 中间件必须有 `resolve()` 方法，它必须返回另一个函数（高阶函数）。为什么？因为有很多第三方插件准备使用 express 中间件，你可以简单地在 Nest 中使用，还要感谢这个方案。

好了，我们已经准备好了中间件，但是我们并没有在任何地方使用它。我们来设置它：
```js
import { Module, MiddlewaresConsumer } from '@nestjs/common';

@Module({
    controllers: [ UsersController ],
    components: [ UsersService ],
    exports: [ UsersService ],
})
export class UsersModule {
    configure(consumer: MiddlewaresConsumer) {
        consumer.apply(AuthMiddleware).forRoutes(UsersController);
    }
}
```
如上所示，模块可以有其他方法，`configure()`。此方法接收 `MiddlewaresConsumer`  对象作为参数，它可以帮助我们配置中间件。

这个对象有 `apply()` 方法，它接收到无数的中间件（这个方法使用扩展运算符，所以可以传递多个由逗号分隔的类）。 `apply()` 方法返回对象，它有两种方法：

- `forRoutes()`：我们使用这种方法通过逗号分隔控制器或对象（具有 `path` 和 `method` 属性），不限个数，
- `with()`：我们使用这种方法将自定义参数传递给 `resolve()` 中间件方法。

### 它如何工作？
当你在 `forRoutes` 方法中传递 `UsersController` 时，Nest 将为控制器中的每个路由设置中间件：
```
GET: users
GET: users/:id
POST: users
```
但是也可以直接定义应该使用哪个路径的中间件，就像这样：
```js
consumer.apply(AuthMiddleware).forRoutes({
    path: '*', method: RequestMethod.ALL
});
```
### 链(Chaining)
你可以简单的调用 `apply()` 链。
```
consumer.apply(AuthMiddleware, PassportMidleware)
    .forRoutes(UsersController, OrdersController, ClientController);
    .apply(...)
    .forRoutes(...);
```
### 顺序

中间件按照与数组相同的顺序调用。在子模块中配置的中间件将在父模块配置之后被调用。

## 网关（Gateways）实时应用程序

Nest 中有特殊组件称为网关。网关帮助我们创建实时的网络应用程序。他们是一些封装的 `socket.io` 功能调整到框架架构。

> 网关是一个组件，因此它可以通过构造函数注入依赖关系。网关也可以注入到另一个组件。

```js
import { WebSocketGateway } from '@nestjs/websockets';

@WebSocketGateway()
export class UsersGateway {}
```
默认情况下，服务器在 80 端口上运行，并使用默认的命名空间。我们可以轻松地改变这些设置：
```js
@WebSocketGateway({ port: 81, namespace: 'users' })
```
当然，只有当 UsersGateway 在 components 模块组件数组中时，服务器才会运行，所以我们必须把它放在那里：
```js
@Module({
    controllers: [ UsersController ],
    components: [ UsersService, UsersGateway ],
    exports: [ UsersService ],
})
```

默认情况下，网关有三个有用的事件：

- `afterInit`，作为本机服务器 `socket.io` 对象参数，
- `handleConnection` 和 `handleDisconnect`，作为本机客户端 `socket.io` 对象参数。
- 有特殊的接口，`OnGatewayInit`, `OnGatewayConnection` 和 `OnGatewayDisconnect` 来帮助我们管理生命周期。

### 什么是消息

在网关中，我们可以简单地订阅发出的消息：
```js
import { WebSocketGateway, SubscribeMessage } from '@nestjs/websockets';

@WebSocketGateway({ port: 81, namespace: 'users' })
export class UsersGateway {
    @SubscribeMessage('drop')
    handleDropMessage(sender, data) {
        // sender is a native socket.io client object
    }
}
```
而在客户端接收如下：
```js
import * as io from 'socket.io-client';
const socket = io('http://URL:PORT/');
socket.emit('drop', { msg: 'test' });
```
## `@WebSocketServer()`

如果要分配选定的 `socket.io` 本地服务器实例属性，你可以使用 `@WebSocketServer()` 装饰器来简单地进行装饰。
```js
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';

@WebSocketGateway({ port: 81, namespace: 'users' })
export class UsersGateway {
    @WebSocketServer() server;

    @SubscribeMessage('drop')
    handleDropMessage(sender, data) {
        // sender is a native socket.io client object
    }
}
```
服务器初始化后将分配值。

### 网关中间件

网关中间件与路由器中间件几乎相同。中间件是一个函数，它在网关消息用户之前被调用。网关中间件函数可以访问本地 `socket` 对象。他们就像屏障一样，如果中间件函数不调用 `next()`，消息将永远不会由用户处理。

例如：
```js
@Middleware()
export class AuthMiddleware implements GatewayMiddleware {
    public resolve(): (socket, next) => void {
        return (socket, next) => {
            console.log('Authorization...');
            next();
        };
    }
}
```
关于网关中间件的一些事实：

你应该使用 `@Middleware()` 注解来告诉 Nest，这个类是一个中间件，
你可以使用 `GatewayMiddleware` 界面，这迫使你实现 `resolve()` 方法，
中间件（和组件一样）可以通过其构造函数注入依赖项（依赖关系必须是模块的一部分），
中间件必须是 `resolve()` 方法，它必须返回另一个函数（高阶函数）
好了，我们已经准备好中间件，但是我们并没有在任何地方使用它。我们来设定一下：
```js
@WebSocketGateway({
    port: 2000,
    middlewares: [ ChatMiddleware ],
})
export class ChatGateway {}
```
如上所示，`@WebSocketGateway()` 接受额外的元数组属性 - `middlewares`，它是一个中间件数组。这些中间件将在消息处理程序前调用。

### 反应流(Reactive Streams)

Nest 网关是一个简单的组件，可以注入到另一个组件中。这个功能使得我们有可能选择将如何对消息做出反应。

当然，只有有必要，我们可以向网关注入组件并调用其适当的方法。

但是还有另一个解决方案，网关反应流（Gateway Reactive Streams）。你可以在这里阅读更多关于他们的信息。

### 微服务(Microservices)支持

将 Nest 应用程序转换为 Nest 微服务是非常简单的。

让我们来看看如何创建 Web 应用程序：
```js
const app = NestFactory.create(ApplicationModule);
app.listen(3000, () => console.log('Application is listening on port 3000'));
```
现在，切换到微服务：
```js
const app = NestFactory.createMicroservice(ApplicationModule, { port: 3000 });
app.listen() => console.log('Microservice is listening on port 3000'));
```
就是这样！

### 通过 TCP 进行通信

默认情况下， Nest 微服务通过 TCP 协议监听消息。这意味着现在 `@RequestMapping()` (以及 `@Post()`，` @Get()` 等等)也不会有用，因为它是映射 HTTP 请求。那么微服务如何识别消息？只是通过模式。

什么是模式？没什么特别的，它是一个对象，字符串或者数字（但这不是一个好注意）。

让我们创建 `MathController` :
```js
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class MathController {
    @MessagePattern({ cmd: 'add' })
    add(data, respond) {
        const numbers = data || [];
        respond(null, numbers.reduce((a, b) => a + b));
    }
}
```
你可能会看到，如果你想创建消息处理程序，你必须使用 `@MessagePattern(pattern)` 进行装饰。在这个例子中，我选择 `{ cmd: 'add' }` 作为模式。

该处理程序方法接收两个参数：

- `data`，它是从另一个微服务器（或者只是 Web 应用程序）发送的数据变量，
- `respond`，接收两个参数（`error` 和 `response`）的函数。

### 客户端

你已经知道了如何接收消息。现在，我们来看看如何从另一个微服务器或者 Web 应用程序发送它们。

在你开始之前，Nest 必须知道你要发送的邮件的位置。很简单，你只需要创建一个 `@Client` 对象。
```js
import { Controller } from '@nestjs/common';
import { Client, ClientProxy, Transport } from '@nestjs/microservices';

@Controller()
export class ClientController {
    @Client({ transport: Transport.TCP, port: 5667 })
    client: ClientProxy;
}
```
`@Client()` 装饰器接收对象作为参数。此对象可以有 3 个属性：

- `transport`，通过这种方式，你可以决定使用哪种方法，TCP 或者 Redis（默认为 TCP）
- `url`，仅用于 Redis 参数（默认为 redis://localhost:6379），
- `port`，端口，默认为 3000。

### 使用客户端

让我们来创建自定义路径来测试我们的通信。
```js
import { Controller, Get } from '@nestjs/common';
import { Client, ClientProxy, Transport } from '@nestjs/microservices';

@Controller()
export class ClientController {
    @Client({ transport: Transport.TCP, port: 5667 })
    client: ClientProxy;

    @Get('client')
    sendMessage(req, res) {
        const pattern = { command: 'add' };
        const data = [ 1, 2, 3, 4, 5 ];

        this.client.send(pattern, data)
            .catch((err) => Observable.empty())
            .subscribe((result) => res.status(200).json({ result }));
    }
}
```
正如你看到的，为了发送消息，你必须使用 `send` 方法，它接收消息模式和数据作为参数。此方法从 `Rxjs` 包返回一个 `Observable` 。

这是非常重要的特性，因为 `Observables` 提供了一组令人惊奇的操作来处理，例如 `combine`, `zip`, `retryWhen`, `timeout` 等等。

当然，如果你想使用 `Promise` 而不是 `Observables`，你可以直接使用 `toPromise()`  方法。

现在，当有人发送 `/test` 请求（`GET`）时，应该如何应用（如果微服务和 web 应用都可用）：
```js
{
    "result": 15
}
```

### Redis

还有另一种方式与 Nest 微服务合作。我们可以使用 `Redis` 的发布/订阅功能，而不是直接 TCP 通信。

在使用之前，你必须安装 Redis。

### 创建 Redis 微服务

要创建 `Redis` 微服务，你必须在 `NestFactory.createMicroservice()` 方法中传递其他配置。
```js
const app = NestFactory.createMicroservice(
    MicroserviceModule,
    {
        transport: Transport.REDIS,
        url: 'redis://localhost:6379'
    }
);
app.listen(() => console.log('Microservice listen on port:', 5667 ));
```
就这样。现在，你的微服务将订阅通过 `Redis` 发布的消息。其它方式依旧相同，如 模式和错误处理等等。

### Redis 客户端

现在让我们来看看如何创建客户端。以前，你的客户端配置如下：
```js
@Client({ transport: Transport.TCP, port: 5667 })
client: ClientProxy;
```

我们想使用 Redis 而不是 TCP， 所以我们必须改变这些配置：
```js
@Client({ transport: Transport.REDIS, url: 'redis://localhost:6379' })
client: ClientProxy;
```
很容易，对么？ 就是这样。其他功能与 TCP 通信中的功能相同。

## 共享模块

Nest 模块可以导出其它组件。这意味着，我们可以轻松地在它们之间共享组件实例。

在两个或者更多模块之间共享实例的最佳方式是创建共享模块。

例如，如果我们要在整个应用程序中共享 `ChatGateway` 组件，我们可以这样做：
```js
import { Module, Shared } from '@nestjs/common';

@Shared()
@Module({
    components: [ ChatGateway ],
    exports: [ ChatGateway ]
})
export class SharedModule {}
```
然后，我们只需要将这个模块导入到另一个模块中，这个模块应该共享组件实例：
```js
@Module({
    modules: [ SharedModule ]
})
export class FeatureModule {}
```

> 注意，也可以直接定义共享模块的范围，了解更多细节。

## 依赖注入

依赖注入是一个强大的机制，可以帮助我们轻松地管理我们类的依赖。它是非常受欢迎的语言，如 C# 和 Java。

在 Node.js 中，这些功能并不重要，因为我们已经有了神奇的模块加载系统，如在文件之间共享实例的很容易的。

模块加载系统对于中小应用足够用了。当代码增长时，顺利组织层之间的依赖就变得更加困难。总有一天会变得爆炸。

它比 DI 构造函数更直观。这就是为什么 Nest 有自己的 DI 系统。

### 自定义组件

你已经了解到了，将组件添加到所选的模块是非常容易的。
```js
@Module({
    controllers: [ UsersController ],
    components: [ UsersService ]
})
```
是还有一些其他情况， Nest 允许你利用。

### 使用 value

```js
const value = {};
@Module({
    controllers: [ UsersController ],
    components: [
        { provide: UsersService, useValue: value }
    ],
})
```
当：

你想要使用具体的值，现在，在这个模式中， Nest 将把 `value` 与 `UsersService` 元类型相关联，
你想要使用单元测试。

### 使用 class
```js
@Component()
class CustomUsersService {}

@Module({
    controllers: [ UsersController ],
    components: [
        { provide: UsersService, useClass: CustomUsersService }
    ],
})
```
当：

你只想在此模块中使用所选的更具体的类。

### 使用工厂
```js
@Module({
    controllers: [ UsersController ],
    components: [
        ChatService,
        {
            provide: UsersService,
            useFactory: (chatService) => {
                return Observable.of('value');
            },
            inject: [ ChatService ]
        }
    ],
})
```
当：

你想提供一个值，它必须使用其他组件（或定制包特性）来计算，
你要提供异步值（只返回 `Observable` 或 `Promise`），例如数据库连接。
请记住：

如果要使用模块中的组件，则必须将它们传递给注入数组。 Nest 将按照相同的顺序传递作为参数的实例。

### 定制 providers

```js
@Module({
    controllers: [ UsersController ],
    components: [
        { provide: 'isProductionMode', useValue: false }
    ],
})
```
当你想提供一个选择的键值。

> 请注意：
> 可以使用各种类型的 `useValue`, `useClass` 和 `useFactory`。

怎么使用？

使用选择的键注入自定义提供组件 / 值，你必须告诉 Nest，就像这样：
```js
import { Component, Inject } from '@nestjs/common';

@Component()
class SampleComponent {
    constructor(@Inject('isProductionMode') isProductionMode: boolean) {
        console.log(isProductionMode); // false
    }
}
```
## ModuleRef

有时候你可能希望直接从模块引用获取组件实例。对于 Nest 并不是一件大事，你只需要在类中注入 ModuleRef ：
```js
export class UsersController {
    constructor(
        private usersService: UsersService,
        private moduleRef: ModuleRef) {}
}
```
ModuleRef 提供一个方法：

get<T>(key)，它返回等效键值实例（主要是元类型）。如 moduleRef.get<UsersService>(UsersService) 从当前模块返回 UsersService 组件的实例
例如：
```js
moduleRef.get<UsersService>(UsersService)
```
它将从当前模块返回 UsersService 组件的实例。

## 测试

Nest 为你提供了一套测试工具，可以提供应用测试过程。可以有两种方法来测试你的组件和控制器，隔离测试和使用专用的 Nest 测试工具。

### 隔离测试

Nest 的控制器和组件都是一个简单的 JavaScript 类。这意味着，你可以很容易的自己创建它们：
```js
const instance = new SimpleComponent();
```
如果你的类还有其它依赖，你可以使用 test doubles，例如 Jasmine 或 Sinon 库：
```js
const stub = sinon.createStubInstance(DependencyComponent);
const instance = new SimpleComponent(stub);
```

### 专用的 Nest 测试工具

测试应用程序构建块的另一种方法是使用专用的 Nest 测试工具。

这些测试工具放在静态 `Test` 类中（`@nestjs/testing` 模块）。
```js
import { Test } from '@nestjs/testing';
```
该类提供两种方法：

- `createTestingModule(metadata: ModuleMetadata)`，它作为参数接收简单的模块元数据（和 `Module()` class 相同）。此方法创建一个测试模块（与实际 Nest 应用程序相同）并将其存储在内存中。
- `get<T>(metatype: Metatype<T>)`，它返回选择的实例（`metatype` 作为参数传递），控制器/组件（如果它不是模块的一部分，则返回 `null`）。
例如：
```js
Test.createTestingModule({
    controllers: [ UsersController ],
    components: [ UsersService ]
});
const usersService = Test.get<UsersService>(UsersService);
```

### Mocks, spies, stubs

有时候你可能不想使用组件/控制器的实例。你可以选择这样，使用 test doubles 或者 自定义 values / objects 。
```js
const mock = {};
Test.createTestingModule({
    controllers: [ UsersController ],
    components: [
        { provide: UsersService, useValue: mock }
    ]
});
const usersService = Test.get<UsersService>(UsersService); // mock
```

## 异常过滤器（Exception Filters）

使用 Nest 可以将异常处理逻辑移动到称为 Exception Filters 的特殊类。

如何工作？
让我们来看下下面的代码：
```js
@Get('/:id')
public async getUser(@Response() res, @Param('id') id) {
    const user = await this.usersService.getUser(id);
    res.status(HttpStatus.OK).json(user);
}
```
想象一下，`usersService.getUser(id)` 方法会抛出一个 `UserNotFoundException` 异常。我们需要在路由处理程序中捕获异常：
```js
@Get('/:id')
public async getUser(@Response() res, @Param('id') id) {
    try {
        const user = await this.usersService.getUser(id);
        res.status(HttpStatus.OK).json(user);
    }
    catch(exception) {
        res.status(HttpStatus.NOT_FOUND).send();
    }
}
```
总而言之，我们必须向每个可能发生异常的路由处理添加 try ... catch 块。还有其它方式么？ 是的，Exception Filters。

让我们创建 NotFoundExceptionFilter ：
```js
import { Catch, ExceptionFilter, HttpStatus } from '@nestjs/common';

export class UserNotFoundException {}
export class OrderNotFoundException {}

@Catch(UserNotFoundException, OrderNotFoundException)
export class NotFoundExceptionFilter implements ExceptionFilter {
    catch(exception, response) {
        response.status(HttpStatus.NOT_FOUND).send();
    }
}
```
现在，我们只需要告诉我们的 `Controller` 使用这个过滤器：
```js
import { ExceptionFilters } from '@nestjs/common';

@ExceptionFilters(NotFoundExceptionFilter)
export class UsersController {}
```
所以如果 `usersService.getUser(id)` 会抛出 `UserNotFoundException`，那么， `NotFoundExceptionFilter` 将会捕获它。

### 更多异常过滤器

每个控制器可能具有无限次的异常过滤器（仅用逗号分隔）。
```js
@ExceptionFilters(NotFoundExceptionFilter, AnotherExceptionFilter)
```

依赖注入
Exception filters 与组件相同，因此可以通过构造函数注入依赖关系。

### HttpException

注意：它主要用于 REST 应用程序。
Nest 具有错误处理层，捕获所有未处理的异常。

如果在某处，在你的应用程序中，你将抛出一个异常，这不是 HttpException（或继承的），Nest 会处理它并返回到下面用户的 json 对象（500 状态码）：
```js
{
    "message": "Unkown exception"
}
```

### 异常层次结构

在应用程序中，你应该创建自己的异常层次结构(Exceptions Hierarchy)。所有的 HTTP 异常都应该继承自内置的 `HttpException`。

例如，您可以创建 `NotFoundException` 和 `UserNotFoundException` 类：
```js
import { HttpException } from '@nestjs/core';

export class NotFoundException extends HttpException {
    constructor(msg: string | object) {
        super(msg, 404);
    }
}

export class UserNotFoundException extends NotFoundException {
    constructor() {
        super('User not found.');
    }
}
```
然后，如果你的应用程序中的某个地方抛出 UserNotFoundException，Nest 将响应用户状态代码 404 及以下 json 对象：
```js
{
    "message": "User not found."
}
```
它允许你专注于逻辑，并使你的代码更容易阅读。
