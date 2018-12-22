---
title: 关于CJS和ESM互相引入的问题探究
tags:
  - JavaScript
abbrlink: 1bff
date: 2018-12-01 00:00:00
---

ESM支持命名导入和默认导入

```js
//命名导入
import { namedExport } from 'myModule'

// 默认导入
import defaultExport from 'myModule'
```

这两种方式可以共存

<!--more-->

而CommonJs就简单了

```js

// 命名导入
module.exports = {
    name
}

const { name } = require('myModule')
```

```js
// 默认导入

module.exports = name

const name = require('myModule')

```

对比于ES6， CJS的问题是只能存在命名导入**或者**默认导入

或者简单概括一点 ES6是CJS的超集，ES6可以直接导入CJS

但是由于CJS中不能同时存在 `named import` 和 `default import`

所以CJS中引入ES6比较麻烦

因此 `default` 只能挂到 `module.exports.default`上

我们写nodejs使用ES6，最后会被tsc或者babel编译成 CJS 模块, 最后发布的就是这些 CJS 代码

看一下用ES6的时候babel是如何处理的

```js
import server from 'server'
```

```js
'use strict';

var _server = require('server');

var _server2 = _interopRequireDefault(_server);

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
```

可以看到，默认导入的时候会检测一下是否含有`__esModule`这个属性，如果有，就表明server这个模块是ES6写CJS发布的，

这里直接返回了 obj， 也就是 module.exports对象


```js
const defaultVariables = require('myModule').default

```

如果模块本身就是 CJS编写， 那么__esModule 为 undefined， 最后返回

```js
{
    default:'defaultExport',
    age: '20' //这些是这个模块的命名导出对象
}
```

而这一个对象实际上挂载到 `module.exports`

```
module.exports = {
    default:'defaultExport',
    age:'20'
}
```

因为我们拿到的是 exports对象，可以将它像 CJS 模块一样使用

```js
const {age } = require('myModule')
```

不过对于default导入比较尴尬了

只能通过

```js
const defaultExport = require('myModule').default
```

来获取

更多的可以参考

```
https://www.zhihu.com/question/288322186/answer/460742151
```