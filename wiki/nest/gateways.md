---
layout: docs
title:  "Gateways"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/gateways
---

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
