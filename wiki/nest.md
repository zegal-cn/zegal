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
