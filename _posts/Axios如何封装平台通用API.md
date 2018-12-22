---
title: Axios如何封装平台通用API
tags:
  - JavaScript
abbrlink: ee6f
---

使用Vue SSR的时候将在server和client准备两套API，server使用nodejs的http实现，client使用XHR实现

所以检测代码的运行环境至关重要

这里直接是分析了Axios这个HTTP库的检测逻辑

<!--more-->

```js
function getDefaultAdapter() {
  var adapter;
  // Only Node.JS has a process variable that is of [[Class]] process
  if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // For node use HTTP adapter
    adapter = require('./adapters/http');
  } else if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    adapter = require('./adapters/xhr');
  }
  return adapter;
}
```

实现就是检测是否存在process并且toString之后是否等于 `[object process]`

然后分别`require`不同的API实现


这里再记录一些东西

ReferenceError 和 TypeError区别是什么


RefError表示这个变量根本不存在， 如果变量的值是undefined那么这个值是存在的

即使这个变量不存在

但是

```js
typeof a === 'undefined' // 假设没有a这个变量
```

TypeError表示这个变量存在，但是你执行了不适合的行为

经常出现的就是引用了Undefined的变量

```js
var a = undefined
a.to()

//TypeError
```

如何检测一个对象的类型？

var a = []

当我们不知道a是什么的时候，如何判断a是数组？

```js
typeof a !== 'undefined' && Object.prototype.toString.call(a) === '[object Array]'
```

这里不能用

```js
typeof a !== 'undefined' && a.toString() === '[object Array]'
```

因为Array中override了Array对象的toString()

所以`a.toString()`(假设a是Array对象)会得到空字符串 `""`