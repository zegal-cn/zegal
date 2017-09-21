---
layout: docs
title:  "Middlewares"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/middlewares
---

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
