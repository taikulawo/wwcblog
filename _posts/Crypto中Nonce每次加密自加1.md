---
title: Crypto中Nonce每次加密自加1
abbrlink: d6d3
date: 2018-06-25 23:10:07
tags:
	- Crypto
	- Nonce
---


最近在实现一个AEAD加密传输，原本打算把`Nonce`直接当作AE传输，最后实现的时候发现解密需要`Nonce`，但是不解密就得不到`Nonce`，真是尴尬。。
`Shadowsocks``使用了`OpenSSL`的DLL来nonce + 1，但是我没找到代码实现，最后在`StackOverflow`上找到了一个

*Hacker级写法*
```java
package com.wwc;

import org.junit.Test;

import java.util.Arrays;

public class CommonTest {

    @Test
    public void nonceIncrementTest(){
        byte[] nonce = new byte[8];
        int count = 100;
        while(count > 0){
            count --;
            if(++nonce[7] == 0) if( ++ nonce[6] == 0)
                if(++nonce[5] == 0 ) if(++ nonce[4] == 0)
                    if(++ nonce[3] == 0) if(++ nonce[2] == 0)
                        if(++ nonce[1] == 0) if(++ nonce[0] == 0) break;

               
            //将Nonce转换成Long输出
            //   long value = 0;
            //   for(int i = 0 ; i < nonce.length ; ++i){
            //       value = (value << 8) + (nonce[i] & 0xff);
            //   }
            //   System.out.println(value);

            System.out.println(Arrays.toString(nonce));
    }
}

```


去除掉多余的就是下面这样。

`byte[] nonce = new [12];`
```java
        if(++nonce[11] == 0) if( ++ nonce[10] == 0)
            if(++nonce[9] == 0 ) if(++ nonce[8] == 0)
                if(++ nonce[7] == 0) if(++ nonce[6] == 0)
                    if(++ nonce[5] == 0) if(++ nonce[4] == 0)
                        if(++ nonce[3] == 0) if(++ nonce[2] == 0)
                            if(++ nonce[1] == 0) if(++ nonce[0] == 0)
```

还有一个C#版本的，都测试过了，都很好
```csharp
unsafe static void Test() {
    byte[] data = new byte[8];
    fixed (byte* first = data) {
        ulong* value = (ulong*)first;
        do {
            // use data here
            *value = *value + 1;
        } while (*value != 0);
    }
}
```

[我是勤劳的搬运工:D](https://stackoverflow.com/questions/1444089/increment-a-byte)