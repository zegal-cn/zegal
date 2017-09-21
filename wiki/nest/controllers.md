---
layout: docs
title:  "Controllers"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/controllers
---

## 控制器(Controllers)

控制层（`Controllers`）负责处理传入的 HTTP 请求。在 Nest 中，控制器是一个带有 `@Controller()` 装饰器的类。

在上一节中，我们为应用程序设置了入口点。现在，让我们来构建我们的第一个文件路径 `/users`：
```js
import { Controller, Get, Post } from '@nestjs/common';

@Controller()
export class UsersController {
    @Get('users')
    getAllUsers() {}

    @Get('users/:id')
    getUser() {}

    @Post('users')
    addUser() {}
}
```
我们刚刚创建了一个具有 3 种不同路径的路由：
```
GET: users
GET: users/:id
POST: users
```

Nest 允许我们将额外的元数据传递给 `@Controller()` 装饰器 - 路径，这是每个路由的前缀。让我们重写我们的控制器：
```js
@Controller('users')
export class UsersController {
    @Get()
    getAllUsers(req, res, next) {}

    @Get('/:id')
    getUser(req, res, next) {}

    @Post()
    addUser(req, res, next) {}
}
```
如果你想了解更多关于 `req` （请求），`res`（响应）和 `next`，你可以参考 Express 的[路由文档](https://expressjs.com/en/guide/routing.html)。在 Nest 中，它们是等价的。

但是有一个重要的区别。 Nest 提供了一组自定义的装饰器，你可以使用它们来标记参数。

Nest	| Express
------|--------
`@Request()`	| `req`
`@Response()`	| `res`
`@Next()`	| `next`
`@Session()`	| `req.session`
`@Param(param?: string)`	| `req.params[param]`
`@Body(param?: string)`	| `req.body[param]`
`@Query(param?: string)`	| `req.query[param]`
`@Headers(param?: string)`	| `req.headers[param]`

你可以这样使用它们：
```js
import { Response, Param } from '@nestjs/common';

@Get('/:id')
public async getUser(@Response() res, @Param('id') id) {
    const user = await this.usersService.getUser(id);
    res.status(HttpStatus.OK).json(user);
}
```
`ApplicationModule` 并添加一些元数据，只需要将 `controller` 插入 `controllers` 数组中。
```js
import { Module } from '@nestjs/common';
import { UsersController } from "./users.controller";

@Module({
    controllers: [ UsersController ]
})
export class ApplicationModule {}
```
