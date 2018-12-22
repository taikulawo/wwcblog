---
title: Shadowsocks-Windows-Key派生过程
abbrlink: ed73
date: 2018-06-21 16:00:51
tags:
	- Shadowsocks
---
今天来记录一下debug Shadowsocks-Windows密钥的时候的代码
Shadowsocks有一个MasterKey，这个是用户输入的，并且进行了MD5计算，

每一个Session有一个单独的会话密钥，使用`HKDF_SHA1`算法
[Shadowsocks-Key-Derivation](https://shadowsocks.org/en/spec/AEAD-Ciphers.html)

生成会话密钥
```csharp
        public void DeriveSessionKey(byte[] salt, byte[] masterKey, byte[] sessionKey)
        {
            int ret = MbedTLS.hkdf(salt, saltLen, masterKey, keyLen, InfoBytes, InfoBytes.Length, sessionKey,
                keyLen);
            if (ret != 0) throw new System.Exception("failed to generate session key");
        }
```
<!--more-->
其中的`salt`在Shadowsocks第一次将数据包发送给Server的时候会放在数据包的开头

生成`Master Key`
```csharp
public static void LegacyDeriveKey(byte[] password, byte[] key, int keylen)
        {
            byte[] result = new byte[password.Length + MD5_LEN];
            int i = 0;
            byte[] md5sum = null;
            while (i < keylen) {
                if (i == 0) {
                    md5sum = MbedTLS.MD5(password);
                } else {
                    Array.Copy(md5sum, 0, result, 0, MD5_LEN);
                    Array.Copy(password, 0, result, MD5_LEN, password.Length);
                    md5sum = MbedTLS.MD5(result);
                }
                Array.Copy(md5sum, 0, key, i, Math.Min(MD5_LEN, keylen - i));
                i += MD5_LEN;
            }
        }
```

对于这个while循环，刚开始把我看蒙了，原本以为拿着`password`直接算一遍hash就完事了，结果不是这样。。

一个进入两次while，第一次进`if`,第二次进`else`

`keyLen = 32;`
`byte[] key = new byte[32];`

1. 先计算一遍password的hash,然后存放到key的前16bytes当中
2. else中现将上一次的ms5sum放到result当中，然后将password存放到后面的11字节（因为我的password是11bytes）。
3. 然后进行hash计算，copy到key中的最后，over。

```
+----------------------+                                    +---------------------+-----------------------+
|  password.getBytes() |  ->>>>>> md5sum(A)(16bytes) ->>>>>>| md5sum(A)(16bytes)  |0000000000...(16bytes) |
+----------------------+                                    +---------------------+-----------------------+

+---------------------+---------------------+                                   +---------------------+-----------+
| md5sum(A)(16bytes)  | password.getBytes() |  ->>>>> 16 bytes hashCode(B)  ->>>| md5sum(A)(16bytes)  |B(16bytes) |
+---------------------+---------------------|                                   +---------------------+-----------+
```

最后奉上几张学习`Shadowsocks`的时候debug的图:)


LegacyDeriveKey和HKDF函数的调用关系
![LegacyDeriveKey和HKDF函数的调用关系](https://chaochaogege.net/images/LegacyDeriveKeyAndHKDF.png)

Shadowsocks工程中对于Master key的引用
![Shadowsocks工程中对于Master key的引用](https://chaochaogege.net/images/Shadowsocks-MasterKeyRef.png)

LegacyDeriveKey调用栈
![LegacyDeriveKey调用栈](https://chaochaogege.net/images/LegacyDeriveKeyCallStack.png)

LegacyDerive执行之前各参数
![LegacyDerive执行之前参数](https://chaochaogege.net/images/BeforeLegacyDeriveKey.png)

LegacyDerive执行之后参数
![LegacyDerive执行之后参数](https://chaochaogege.net/images/AfterLegacyDeriveKey.png)
