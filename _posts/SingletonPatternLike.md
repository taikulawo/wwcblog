---
title: Java使用Interface的default来实现假单例模式
abbrlink: da65
data: 2018-06-18 23:00:00
tags:
	- Java
---

刚开始想要通过工厂模式来实现，但发现为了创建不同的Outbound实例需要强制规定Outbound的实现者去实现一个`public static Object getInstance()`，虽然JDK1.8提供了Interface static，但实现者却不能Override。最终有了下面的解决办法：
<!--more-->

传统的`Singleton pattern`模式：
```java
public class Socks5{
    private static Socks5 socks5 = null;
    private Socks5(){
    }
    
    public static Socks5 getInstance(){
        if(socks5 == null){
            return socks5 = new Socks5();
        }
        return socks5;
    }
}
```

但是我想实现一个对象的创建全部由Socks5的编写者来控制的模式，StackOverflow上面没找到答案，所以有了下面写的样子。

***一个仿`Singleton pattern`模式***

```java
public interface Inbound {
    default Inbound getInstance(){
        try {
            return this.getClass().getConstructor().newInstance();
        } catch (InstantiationException 
                | IllegalAccessException 
                | InvocationTargetException 
                | NoSuchMethodException e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

```java
public class Socks5 implements Inbound {
    private static Socks5 socks5 = null;

    @Override
    public Inbound getInstance(){
        if(socks5 == null){
            socks5 = new Socks5();
        }
        return socks5;
    }

    public static void main(String[] argv){
        Inbound inbound = new Socks5();
        System.out.println(inbound.getClass().getName());
        Inbound in1 = inbound.getInstance();
        Inbound in2 = inbound.getInstance();
        System.out.println(in1 == in2);
    }
}
```

Result:
```
Socks5
true
```

通过这种方式可以让对象的创建由继承了`Inbound`接口的编写者来负责，但是调用者必须保证只能使用`new`来创建一个`Socks5`的实例。

还有一种方法可以强制调用者只使用一次`new`：

```java
public class Socks5 implements Inbound {
    private static Socks5 socks5 = null;
    private static boolean created = false;
    public Socks5() throws Exception {
        if(created){
           throw new Exception("cannot create");
        }
        created = true;
    }

    private Socks5(int a ){
    }

    @Override
    public Inbound getInstance(){
        if(socks5 == null){
            try {
                socks5 = new Socks5(1);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return socks5;
    }

    public static void main(String[] argv) throws Exception {
        Inbound inbound = new Socks5();
        Inbound inbound2 = new Socks5();
        System.out.println(inbound.getClass().getName());
        Inbound in1 = inbound.getInstance();
        Inbound in2 = inbound.getInstance();
        System.out.println(in1 == in2);
    }
}

```

Result:
```
Exception in thread "main" java.lang.Exception: cannot create
	at Socks5.<init>(Socks5.java:6)
	at Socks5.main(Socks5.java:29)
```
通过构造函数抛出异常来强行禁止调用者多次使用`new`创建。缺点是`new`的时候需要`try` `catch`，不美观。

我本身想要实现让继承自`Inbound`接口的编写者来自己控制对象的创建，这样我只负责将参数传递给对象也不管是否是同一个对象

这样我会保证`new`操作符只是用一次，所以第一种方法很整洁。

