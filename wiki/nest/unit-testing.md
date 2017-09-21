---
layout: docs
title:  "测试"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/unit_testing
---
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
