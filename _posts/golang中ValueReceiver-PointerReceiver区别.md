---
title: golang中ValueReceiver-PointerReceiver区别
tags: 
	- Golang
abbrlink: 8e0d
date: 2018-08-28 23:47:00
---



如下代码
```golang
func main(){
	var buf bytes.Buffer
	fmt.Fprint(&buf,"test")
}
```

将`"test"`写入buf，由于bytes.Buffer对于io.Writer接口使用了如下实现
```golang
func (b *Buffer) Write(p []byte) (n int, err error) {
	b.lastRead = opInvalid
	m, ok := b.tryGrowByReslice(len(p))
	if !ok {
		m = b.grow(len(p))
	}
	return copy(b.buf[m:], p), nil
}
```
<!--more-->
可以看到bytes.Buffer使用了Pointer Receiver，既然如此，当尝试调用bytes.Buffer的Write时，调用者必须使用pointer
上面这里成立因为`Buffer`本身就是一个`struct`
```golang
type Buffer struct{

}
```

一个相反的例子
```golang
var w http.ResponseWriter
fmt.Fprint(w,http.StatusText(http.StatusBadRequest),http.StatusBadRequest)
```

这里http.ResponseWriter则是一个接口，而不是一个具体的实现类(struct)，接口已经是一个pointer，他指向底层的数据结构，所以不需要`&`
```golang
type ResponseWriter interface{
	
}
```

再来看一个结构体value-pointer receiver的例子
```
import "fmt"

type list struct{
	greet string
	count int
}

func (l * list)append(){
	l.greet = "Hello"
}

func (l list)sub(){
	l.count --
}

func main(){
	var l0 list
	l0.append()

	l2 := &list{
		count:100,
	}
	l2.sub()

	fmt.Println(l0.greet)
	fmt.Println(l2.count)

}
```

对于两种类型`list`, `*list`
其方法集分别包含
```
list
	-len()

*list
	-len()
	-append()
```

`list`并不能直接调用`append()`，但编译器将`l0.append()`翻译成了`&(l0).append()`

`*list`由于是`pointer receive`，但编译器将`l2.append()`翻译成了`(*l2).len()`

所以最后的输出结果是
```
Hello
100
```


以上`list`没有涉及接口，现在看一下当有接口参与的时候如何
这一次直接给出golang的例子

```golang
type List []int

func (l List) Len() int        { return len(l) }
func (l *List) Append(val int) { *l = append(*l, val) }

type Appender interface {
	Append(int)
}

func CountInto(a Appender, start, end int) {
	for i := start; i <= end; i++ {
		a.Append(i)
	}
}

type Lener interface {
	Len() int
}

func LongEnough(l Lener) bool {
	return l.Len()*10 > 42
}

func main() {
	// A bare value
	var lst List
	CountInto(lst, 1, 10) // INVALID: Append has a pointer receiver
	if LongEnough(lst) {  // VALID: Identical receiver type
		fmt.Printf(" - lst is long enough")
	}

	// A pointer value
	plst := new(List)
	CountInto(plst, 1, 10) // VALID: Identical receiver type
	if LongEnough(plst) {  // VALID: a *List can be dereferenced for the receiver
		fmt.Printf(" - plst is long enough")
	}
}

```

存储在interface里面的值不能被寻址，

对于`bare value`
	-> `Countinto`，由于`1st`是值类型，`1st`值存储在`interface`，其不能被计算地址，而`Append`要求`pointer receiver`，所以会编译错误
	-> `LingEnough`, `1st`是值类型，本身就是`value`, `List`中`Len`实现仅仅需要`value receiver`，编译通过

对于`pointer value`
	-> 'Countinto` `plst`现在是`pointer`, `Append`需要`pointer receiver`，编译通过
	-> 'LongEnough' `Len`需要`value`类型，`plst`通过解引用可以得到原始值，满足`value receiver`，编译通过
	
	
这些例子来自于
https://github.com/golang/go/wiki/MethodSets#interfaces