---
title: 如何调试Vue-SSR?
tags:
  - Vue
abbrlink: '7807'
---

client side render 最好处理，从 开发者工具 找到 webpack 的sourcemap， 然后在每个Vue文件的生命下断点，再配合 Vue dev tools 很简单


但许多的渲染问题是在SSR，对于 Object 对象没办法用 `console.log` 打印变量，(打印出`[object, Object]`)

所以只能一步一步用 `VScode` 下断点调试，之前一直Google却没有找到方法，这次因为实在是不知道哪里出问题了，只能硬着来调试一下


<!--more-->
------------------------------------------------------

**为了能将调试器进行绑定，我们必须从vscode启动，而不是`npm run dev`**


和调试通常的node代码差不多，启动Vue 的 `server.js`

大体launch.json 如下

```json
    {
        "version": "0.2.0",
        "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceFolder}\\server.js",
            "protocol": "inspector"
        }
    ]
    }
```

然后在 server.js 的 render.renderToString() 下断点

这样做是因为整个ssr渲染的代码都被打包了，成为 node 的 `node-internal`，也就是只存在于 内存，没办法直接在 vscode里面下断点

为了能找到我们代码的执行位置，需要断进去找到Vue 运行的 sandbox

在浏览器触发渲染，然后在 `renderToString()` `step into`

进入之后会看到返回一个js object，

```js

renderToString: function (context, cb) {
    var assign;

    if (typeof context === 'function') {
        cb = context;
        context = {};
    }

    var promise;
    if (!cb) {
        ((assign = createPromiseCallback(), promise = assign.promise, cb = assign.cb));
    }

    run(context).catch(function (err) {
        rewriteErrorTrace(err, maps);
        cb(err);
    }).then(function (app) {
        if (app) {
        renderer.renderToString(app, context, function (err, res) {
            rewriteErrorTrace(err, maps);
            cb(err, res);
        });
        }
    });

    return promise
    },

```

在run(context)下断点，然后进入step into

里面是一个闭包函数

最后有一行

```
resolve(runner(userContext));
```

这里下断点然后进入runner代码就可以看到你的`entry-server.js` 了

**因为source map经常会定位错位置，所以推荐在你想到断下的地方下断直接过去**