---
layout: docs
title:  "Exception Filters"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/exception_filters
---
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
