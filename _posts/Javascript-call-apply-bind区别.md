---
title: JavaScript-call-apply-bind区别
tags:
  - JavaScript
abbrlink: 62aa
date: 2018-10-14 20:01:27
---



**`call`, `apply` `bind` 都绑定`this`**

#### 1. call

call绑定之后立即调用函数

```javascript
var person = {  
  name: "James Smith",
  hello: function(thing) {
    console.log(this.name + " says hello " + thing);
  }
}

person.hello("world");  // output: "James Smith says hello world"
person.hello.call({ name: "Jim Smith" }, "world"); // output: "Jim Smith says hello world"
```

<!--more-->

#### 2.apply

apply和call很相似，绑定之后立即调用函数，区别在于apply接受的参数为数组形式

```javascript
function personContainer() {
  var person = {  
     name: "James Smith",
     hello: function() {
       console.log(this.name + " says hello " + arguments[1]);
     }
  }
  person.hello.apply(person, arguments);
}
personContainer("world", "mars"); // output: "James Smith says hello mars", note: arguments[0] = "world" , arguments[1] = "mars"                                     
```


#### 3. bind

bind绑定this之后不调用函数，但会返回一个新的函数供使用

```javascript
var person = {  
  name: "James Smith",
  hello: function(thing) {
    console.log(this.name + " says hello " + thing);
  }
}

person.hello("world");  // output: "James Smith says hello world"
var helloFunc = person.hello.bind({ name: "Jim Smith" });
helloFunc("world");  // output: Jim Smith says hello world"
```

或者

```javascript 
var helloFunc = person.hello.bind({ name: "Jim Smith" }, "world");
helloFunc();  // output: Jim Smith says hello world"
```


最后放到一起写一个简单的demo

```javascript
var wwc = {
    name:'wwc'
}

function print(who){
    console.log(`${who} said ${this.name}`)
}
print.apply(wwc,['apply'])
print.call(wwc,'call')
bindwwc = print.bind(wwc)
bindwwc('bind')

```