---
title: js中for...in与for...of区别
tags:
  - JavaScript
abbrlink: 78f3
date: 2018-12-24 00:00:00
---

首先，我们可以将 Array看作以下标作为key，值作为value的js对象

```js
;(function(){
    let arr = [1,2,3]

    for(a in arr){
        console.log(a)
    }

    console.log(`---------------------------------`)

    for(a of arr){
        console.log(a)
    }
    console.log(`下一个代码块`)
    let obj = {
        name:'wwc',
        age:21
    }

    for(o in obj){
        console.log(o)
    }

    console.log(`---------------------------------`)
    
    for(o of obj){
        console.log(o)
    }
})();
```

<!--more-->

运行结果


```
0
1
2
---------------------------------
1
2
3
下一个代码块
name
age
---------------------------------
TypeError: obj is not iterable 
```

可以看到两个 `for in` 分别枚举了全部的key，这里就是数组的下标和js对象的key

而第一个 `for of` 直接输出了 对应key的value

等到第二个 `for of`则直接抛出了异常

### for...of

`for of`是ES6 新添加的，如果使用 `for of`，那么被迭代的目标必须实现相应的迭代器协议

比如 `String` `Array`等，js已经内置了`如何迭代`这个对象
而`Object`却没有内置，所以会抛出异常


> The for...of statement creates a loop iterating over iterable objects, including: built-in `String`, `Array`, `Array-like objects` (e.g., `arguments` or `NodeList`), `TypedArray`, `Map`, `Set`, and **`user-defined iterables`**. It invokes a custom iteration hook with statements to be executed for the value of each distinct property of the object.

你如果想要用`for of`迭代Object，只需要自己实现一个迭代器

```js
;(function(){
    let obj = {
        name:'wwc',
        age:21
    }

    for(o in obj){
        console.log(o)
    }

    console.log(`---------------------------------`)
    
    
    Object.prototype[Symbol.iterator] = function*(){
        for(let key of Object.keys(this)){
            yield key
        }
    }

    for(o of obj){
        console.log(o)
    }
})();
```

这样就可以迭代Object，上面的代码两个`console.log()`输出一致

通过自定义迭代器协议，你可以按照自己的想法来迭代任意对象

### for...in

`for in` 遍历全部的可枚举对象(enmuerable)

```js
var a = {
    name: 'wwc'
}

Object.getOwnPropertyDescriptor(a,'name').enmuerable === true

```
也就是 全部的enmuerable为真的属性