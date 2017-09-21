---
layout: docs
title:  "依赖注入"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/dependency-injection
---

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
    components: [{
        provide: UsersService,
        useValue: value
    }],
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
