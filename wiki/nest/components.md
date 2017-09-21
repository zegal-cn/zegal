---
layout: docs
title:  "组件(Components)"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/components
---

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
