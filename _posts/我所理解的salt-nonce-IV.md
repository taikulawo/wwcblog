---
title: '我所理解的salt,nonce,IV'
tags: 
	 - Salt
	 - Nonce
	 - IV
abbrlink: '5761'
date: 2018-06-23 11:15:41
---


一个salt的用途就是抗彩虹表和字典攻击

现在有个密码`Hello,World`，经过hash计算得到hash值
`8edd29822e413ae1620edc483810229e`

由于这个密码许多人用过，所以攻击者的表中就会有

| password    | MD5                             |
|-------------|---------------------------------|
| Hello,World |8edd29822e413ae1620edc483810229e |

由于现在数据库存放的都是对于密码的hash值，所以攻击者只需要寻找`8edd29822e413ae1620edc483810229e`，就可以知道密码是"Hello,World"

加salt之后，比如现在的密码是`Hello,Worldjksaydw@?E`，由于这个密码很少有人用到，攻击者没有对应的hash值，所以就没办法知道密码原文。
所以加盐然后再对密码hash可以增强密码的健壮性

nonce(a number used only once)
只能使用一次，为了使得每一次的密文都不相同，可以配合timestamp来防止重放攻击。

IV(Initialization vector)

IV需要随机化，与nonce比较
- 如果需要密文随机化并且不可被预测，那么需要使用iv
- 如果只需要密文随机化，那么使用nonce
- iv, nonce, salt都不需要保密，但SecretKey需要保密，如果iv, nonce, salt需要保密，那为什么不直接拼装到SecretKey中呢？
- 由于iv需要随机不可预测，所以在TCP传输中iv需要发送给Endpoint
- 由于nonce只是需要每次使用都不相同，所以在Shadowsocks中直接初始化为`byte[12]{00000.....}`，每次的加密解密nonce自加1，这样就减少了需要传递给对端的数据

下面的链接解释挺不错

[nonce and salt diffenence](https://www.reddit.com/r/cryptography/comments/73v67p/whats_the_difference_between_a_nonce_and_a_salt/#bottom-comments)
[iv wiki](https://en.wikipedia.org/wiki/Initialization_vector)

