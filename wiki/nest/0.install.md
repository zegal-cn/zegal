---
layout: docs
title:  "First Steps"
categories: nest
toc: true
group: nest
permalink: /wiki/nest/install
---

## 安装

Git:
```
git clone https://github.com/kamilmysliwiec/nest-typescript-starter.git projectname
cd projectname
npm install
npm run start
```

NPM:
```
npm i --save @nestjs/core @nestjs/common @nestjs/microservices @nestjs/websockets @nestjs/testing reflect-metadata rxjs
```
## 设置应用程序

Nest 采用 ES6 和 ES7 （decorators， async / await）功能构建。这意味着，使用它的最简单的方法是 Babel 或 TypeScript。

配置 [TypeScript](/wiki/typescript/tips/)

### 1. 创建入口模块

`app.module.ts`
```js
import { Module } from '@nestjs/common';

@Module({})
export class ApplicationModule {}
```
### 2. 创建文件 index.ts

使用 `NestFactory` 基于我们的模块类来创建 Nest 应用程序实例。

```js
import { NestFactory } from '@nestjs/core';
import { ApplicationModule } from './app.module';

const app = NestFactory.create(ApplicationModule);
app.listen(3000, () => console.log('Application is listening on port 3000'));
```
## Express 实例

如果要完全控制 `express` 实例的生命周期，你可以简单的传递已创建的对象作为 `NestFactory.create()` 方法的第二个参数，像这样：

```js
import express from 'express';
import { NestFactory } from '@nestjs/core';
import { ApplicationModule } from './modules/app.module';

const instance = express();
const app = NestFactory.create(ApplicationModule, instance);
app.listen(3000, () => console.log('Application is listening on port 3000'));
```

这意味着，你可以直接添加一些自定义配置（例如，设置一些插件，如 `morgan` 或 `body-parser`）。
