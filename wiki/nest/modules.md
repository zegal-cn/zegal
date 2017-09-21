---
layout: docs
title:  "模块(Modules)"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/modules
---

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
