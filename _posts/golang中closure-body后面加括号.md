---
title: golang 中closure body后面加括号
abbrlink: '3994'
date: 2018-08-04 14:10:19
tags:
	- Golang
---


最近在看v2ray的代码，到了核心的goroutine执行阶段，看到如下的表达式
**tasks是函数集合**

```golang
for _, task := range tasks {
        <-s.Wait()
        go func(f func() error) {
            if err := f(); err != nil {
                select {
                case done <- err:
                default:
                }
            }
            s.Signal()
        }(task)
    }
```


第一次看到在函数体结尾`{}`后面加括号的，这个的意思就是定义一个匿名函数，并且传递的参数为括号中的`task`

比如下面的代码
```golang
package main

import (
    "fmt"
)

func main() {
    print := func (){
        fmt.Println("Hello,World")
    }

    start(print)
}

func start(tasks ...func()){
    c := make(chan int)
    for _, task := range tasks{
        go func(f func()){
            f()
            c <- 1
        }(task)
        <- c
    }
}

```
执行后输出`Hello,World`

