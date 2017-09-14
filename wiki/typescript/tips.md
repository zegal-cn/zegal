---
layout: docs
title:  "维基列表"
categories: jekyll update
toc: true
permalink: /wiki/typescript/tips
---

### tsconfig.json 配置

```js
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": false,
    "noImplicitAny": false,
    "noLib": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "target": "es6"
  },
  "exclude": [
    "node_modules"
  ]
}
```
> 注意 `emitDecoratorMetadata` 和 `experimentalDecorators` 必须设置为 true。
