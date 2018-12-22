---
title: Golang-HTTP包下面的Handle和HandleFunc区别
tags:
  - Golang
abbrlink: d102
date: 2018-10-11 10:16:02
---


Handle是一个接口

```golang
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```


也就是如果想要响应HTTP请求就必须实现这个接口

但对于一些HTTP请求我们只需要简单的处理，不需要保持状态
所以出现了`HandleFunc`

<!--more-->

```golang
type HandleFunc func(w http.ResponseWriter, r *http.Request)
```

HandleFunc只是一种简化的写法，最终还是要调用`ServeHTTP`接口函数

来看一下它的封装

```golang
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers. If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

我是第一次看到`type`定义了函数类型之后竟然还能在其上实现接口

最后这个新定义的类型`HandlerFunc`初始化也不同

```golang
mux.Handle(pattern, HandlerFunc(handler))
```

`handler`是我们编写的实际的处理器，HandlerFunc初始化了一个新的变量`handler`

我试了一下`int(1)`，结果可行，说明基本数据类型是可以用类型加括号来初始化，`handler`的函数签名和`HandlerFunc`完全相同

--------------------------------------------------------

我们经常写类的时候总是使用type来定义一个结构体

```golang
type Controller struct{
}
```

当实现接口的时候只需要在`Controller` struct下面有全部的接口函数

也就是Golang的接口可以在所有被定义的类型（type）上实现

