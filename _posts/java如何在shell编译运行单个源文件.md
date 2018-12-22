---
title: java如何在shell编译运行单个源文件
data: '2018-08-07 20:57'
abbrlink: 3cb2
date: 2018-08-07 21:07:25
tags:
	- Java
---


一直用IDE，有时候为了看一段代码的文件打开那么笨重的IDE然后创建文件太麻烦了，所以打算直接shell编译运行

如下代码（包名完全是为了一般性），整个文件夹路径`D:\code\javacode\main\main1\main2`
##### 如果源文件中指明了`package`

```java
package main.main1.main2;

public class Main{
    public static void main(String[] argv){
        for(;;){
        }	
 }
}
```

现在main2文件夹下面打开shell，然后
```
javac Main.java
```

这时候会在main2文件夹下面生成Main.class

然后现在到路径的`javacode`文件夹下面，运行
```
java main.main1.main2.Main
```

最后就会运行了

##### 如果源文件中**没有**指明`package`

这样在通过`javac Main.java`生成class文件之后，不用切换目录，直接运行
```
java Main
```
就可以了