---
layout: docs
title:  "TypeORM"
categories: jekyll update
toc: true
permalink: /wiki/typeorm
---


[TypeORM](https://github.com/typeorm/typeorm)库绝对是node.js世界中最成熟的对象关系映射器（ORM）。 由于它是用TypeScript编写的，它与Nest框架非常相似。 本示例介绍如何将TypeORM与Nest应用程序集成，但在大多数其他情况下，如与`MongoDB`，`Redis`等的连接，该过程几乎相同。

我们从头开始。 首先，我们需要使用从`typeorm`包导入的`createConnection()`函数建立与数据库的连接。 `createConnection()`函数返回`Promise`，因此有必要创建一个异步组件。

```js
// database.providers.ts

import { createConnection } from 'typeorm';

export const databaseProviders = [
  {
    provide: 'DbConnectionToken',
    useFactory: async () => await createConnection({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [
          __dirname + '../**/*.entity.ts',
      ],
      autoSchemaSync: true,
    }),
  },
];
```

```js
export const databaseProviders = [
  {
    provide: 'DbConnectionToken',
    useFactory: async () => await createConnection({
      type: 'sqlite',
      database: './test.sqlite3', // v0.1后，database 替代 storage 
      entities: [
          __dirname + '../**/*.entity.ts',
      ],
      autoSchemaSync: true,
    }),
  },
];
```
> 提示: 按照最佳做法，我们已将分隔的文件中的自定义组件声明为* .providers.ts后缀。

然后，我们需要导出这些提供程序，使其可以访问应用程序的其余部分。
```js
// database.module.ts

import { Module } from '@nestjs/common';
import { databaseProviders } from './database.providers';

@Module({
  components: [...databaseProviders],
  exports: [...databaseProviders],
})
export class DatabaseModule {}
```
It's everything. Now we can inject the Connection object using @Inject() decorator. Each component which would depend on the Connection async component will wait until the Promise would be resolved.

现在我们可以使用`@Inject()`装饰器注入`Connection`对象。 每个依赖于`Connection`异步组件的组件将一直等到`Promise`被解决。

## Repository pattern

TypeORM支持`Repository`设计模式，因此每个实体都有自己的存储库。 这些存储库可以从数据库连接获取。

首先，我们至少需要一个实体。 我们将从官方文档中重用`Phone`实体。

```js
// photo/photo.entity.ts

import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number;

    @Column({ length: 500 })
    name: string;

    @Column('text')
    description: string;

    @Column()
    filename: string;

    @Column('int')
    views: number;

    @Column()
    isPublished: boolean;
}
```
`Phone`实体属于照片目录。 此目录表示`PhotoModule`。 这是你决定在哪里保存你的模型的文件。 从我的角度来看，最好的办法是在适当的模块目录中保存他们几乎他们的域。

我们创建一个`Repository`组件：

```js
// photo.providers.ts

import { Connection, Repository } from 'typeorm';
import { Photo } from './photo.entity';

export const photoProviders = [
  {
    provide: 'PhotoRepositoryToken',
    useFactory: (connection: Connection) => connection.getRepository(Photo),
    inject: ['DbConnectionToken'],
  },
];
```
> 注意：在实际应用程序中，您应该避免使用魔术字符串。 `PhotoRepositoryToken`和`DbConnectionToken`都应保存在分离的`constansts.ts`文件中。

现在我们可以使用`@Inject()`装饰器将`PhotoRepository`注入到`PhotoService`中：
```js
// photo.service.ts

import { Component, Inject } from '@nestjs/common';
import { Repository } from 'typeorm';
import { Photo } from './photo.entity';

@Component()
export class PhotoService {
  constructor(
    @Inject('PhotoRepositoryToken') private photoRepository: Repository<Photo>) {}
}
```

数据库连接是异步的，但是`Nest`使得此过程对最终用户完全不可见。 `PhotoRepository`组件正在等待数据库连接，并且`PhotoService`被延迟，直到存储库准备使用。 当每个组件被实例化时，整个应用程序可以启动。

这是一个最后的`PhotoModule`：

```js
photo.module.ts

import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { photoProviders } from './photo.providers';
import { PhotoService } from './photo.service';

@Module({
  modules: [DatabaseModule],
  components: [
    ...photoProviders,
    PhotoService,
  ],
})
export class PhotoModule {}
```
> 提示不要忘记将`PhotoModule`导入根`ApplicationModule`。
