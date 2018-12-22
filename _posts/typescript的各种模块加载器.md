---
title: typescript的各种模块加载器
tags: 
	- TypeScript
	- JavaScript
abbrlink: 8b3
data: 2018-08-25 13:09:49
---


typescirpt在Js上面加上了类型支持，但如果需要使用则必须要编译成js，在使用tsc的过程中需要指定`module`来选择一种模块加载器
<!--more-->

根据typescipt tsconfig.json文件，如果target为es5，那么使用commonjs的require()来加载模块，如果target为es6，则直接使用es6的import
```
module:      target === "ES3" or "ES5" ? "CommonJS" : "ES6"
```

src/test.ts
```
import '../vendor/practice'
```

vendor/practice.js
```
(function(){

console.log("Hello,World")
}
)()
```

当target为es6时，ts编译之后:

```javascipt
import '../vendor/practise.js';
//# sourceMappingURL=test.js.map

```

使用commonjs编译之后得到
test.js
```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
require("../vendor/practise.js");
//# sourceMappingURL=test.js.map
```
这段代码直接放到ChromeDevTools-Console运行会报如下错误：
```
Uncaught ReferenceError: exports is not defined
```
*当然，就算解决了这个错误，还会有一个require is not defined错误。*
typescipt使用tsc编译是针对单独的每一个模块(文件)，tsc会根据module来使用或者不使用module loader，如果想要将所有编译之后的js文件都打包到一个js文件当中则需要module bundler(webpack, browserify)

可以看到，编译之后的代码当中使用了nodejs的require(基于commonjs)，而browser并没有nodejs的运行环境，所以出现了browserify。
由browserify编译之后的代码直接将require的模块包含进了输出文件当中。(practice.js并没有使用export，所以应该不是模块，具体原因不清楚),
将上面commonjs编译之后的结果交由browserify编译之后得到

*你可以在tsconfig.json中指定`module:none`，这样typescript不会允许使用`import`和`export`来导入导出模块*
```javascript
(function(){function r(e,n,t){function o(i,f){if(!n[i]){if(!e[i]){var c="function"==typeof require&&require;if(!f&&c)return c(i,!0);if(u)return u(i,!0);var a=new Error("Cannot find module '"+i+"'");throw a.code="MODULE_NOT_FOUND",a}var p=n[i]={exports:{}};e[i][0].call(p.exports,function(r){var n=e[i][1][r];return o(n||r)},p,p.exports,r,e,n,t)}return n[i].exports}for(var u="function"==typeof require&&require,i=0;i<t.length;i++)o(t[i]);return o}return r})()({1:[function(require,module,exports){
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
require("../vendor/practise.js");

},{"../vendor/practise.js":2}],2:[function(require,module,exports){
(function(){

console.log("Hello,World")
}
)()
},{}]},{},[1]);

```

最后生成的这段代码就可以直接放到Chrome的DevTools-Console运行了。

这里梳理几个关系，也是自己刚开始从后端开始接触前端过程中碰到的一些库。

### Commonjs
同browserify，区别在于nodejs使用的require基于commonjs,是server side.
而browserify则是browser side的解决方案。如果用在browser，需要使用browserify编译js

### browserify
同上。
如果使用typescipt编写，指定`--target es5`，则为了能在浏览器中使用，流程为：
```
typescript's tsc => browserify compile => final js.
```
browserify会将模块所依赖的库全部打包到一起并与模块合并输出到最终的js文件当中。

### AMD
异步模块加载，RequireJS是AMD的实现，区别于Require()的同步加载，
### Requirejs
AMD标准的实现
### Require()
Nodejs上的模块加载器


### Webpack
打包工具，可以将依赖打包到一个文件，不光可以打包js文件，还可以处理css,scss等文件

[require is not defined](https://stackoverflow.com/questions/31931614/require-is-not-defined-node-js)
[define is not defined](https://stackoverflow.com/questions/38817636/uncaught-referenceerror-define-is-not-defined-typescript)