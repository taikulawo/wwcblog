---
title: 如何将任意TCP流量封装进代理？
---

很久没写东西了，一直都是扔在印象笔记里头，今天兴起写一篇 :)

先说我对于HTTP代理的理解

1. Explicit HTTP
    需要在client上配置，GET http://www.baidu.com/index.html HTTP/1.1    这种情况下直接检索请求行可以获得主机与端口

    CONNECT，需要client主动配合进行TLS预握手
    但许多应用并不支持CONNECT，如何让他们也走代理？下面会提到

2. no-Explicit HTTP
    透明代理，client无法感知到proxy的存在，往往使用iptables等内核特性实现

不管哪一个代理，协议上都指明了目的地，端口二元组，并且client主动将流量发向了代理端口

#### 如何在client不知情的情况下拦截流量进行代理？

我想到了透明代理，感觉是唯一的方法了，不过透明代理的实现方式多种多样

这里以Linux iptables为例子

为了能够拦截ip层的流量，需要内核提供特性支持，利用iptables的NAT重写

iptables -t nat -A OUTPUT -p tcp --dport 666 -j REDIRECT --to 8080

但NAT重写之后端口号就变了，代理如何知道源端口号并发送出去？

Linux下提供

getsockopt(fd, SOL_IP, SO_ORIGINAL_DST, destaddr, &socklen);

```
struct sockaddr_in {
  __kernel_sa_family_t    sin_family;    /* Address family        */
  __be16        sin_port;    /* Port number            */
  struct in_addr    sin_addr;    /* Internet address        */

  /* Pad to size of `struct sockaddr'. */
  unsigned char        __pad[__SOCK_SIZE__ - sizeof(short int) -
            sizeof(unsigned short int) - sizeof(struct in_addr)];
};
```

通过这个调用，可以获得源IP与port，这依赖于iptables对重写的包进行追踪

 Redsocks就是这么做的
https://github.com/darkk/redsocks/blob/df6394cf95d897b07974375e0d0bad13dcfcb4e8/base.c#L223

有了源流量的IP:Port，就可以转换成HTTP CONNECT
CONNECT 192.168.1.2:666

一个可以 wrap any tcp traffic 的http 代理就这么诞生了
socks代理同理

额外再写点

#### 既然这个代理可以承载任意tcp流量，如果tcp中承载的是TLS，那我们如何去解密这个流量？

看了许多mitmproxy的文档，有了点思路

为了解密TLS，我们需要MITM，自己伪装成root ca来签发证书

以mitmproxy为例，mit开启http代理，监听 8080 端口

1. 当收到 client hello之后，通过 SO_ORIGINAL_DST 获得原始二元组，然后用TLS连接server，获得证书，找到证书域名回身签发假证书给client，成功欺骗，之后可以SSLLOGFILE获得 masterkey，wireshark解密TLS（直接用fiddler，mitmproxy也可以）
2. 使用上述我们封装的代理，获得二元组，找到证书，然后回身签发

但由于 subject alternative name的存在，client想要连接的不一定是证书上签发的，可能是 SAN 中的，所以我们需要在 证书中嗅探出 SAN，伪造证书的时候一起加上

还有一个需要注意的是TLS的 server name indication 扩展

这个扩展允许一台主机托管多个ssl网站，client通过 SNI 指明想要访问的域名，然后server返回正确的证书，为了在这种情况下伪造证书，我们需要收到SNI之后和server 进行 TLS
取得正确的证书之后签发，欺骗 client

https://docs.mitmproxy.org/stable/concepts-howmitmproxyworks/

mitmproxy整个官网的文档都推荐阅读，涉及到TLS协议本身的特性，对整个网络路由也能有理解

还想再写点东西

这个我没有试验过，不知道是否可行，只是根据上面的想法和经验推测的

#### 稀奇古怪的想法
client 使用 TLS与 example.com:8443 通讯，如何在不使用iptables等内核特性的情况下解密TLS

这种方式需要已知拦截域名，自定义DNS解析

为了能够拦截example.com的流量，需要将example.com的DNS解析到本地，进行DNS欺诈

1. 修改host，将example.com 指向 127.0.0.1，然后我们的代理直接监听 8443 端口，这样 client的请求直接指向了我们的代理
2. 由于是TLS通讯，代理进行SNI嗅探，获得域名
3. 为了避免代理解析example.com的时候被host定向，我们需要指定DNS解析服务，获得ip之后SNI获得服务器证书，代理回身签发假的证书

#### 最后再加点东西，最近看的比较多，笔记里零散记的东西太多了，需要汇总起来

当内核启用IP转发之后，整个OS也就成了路由器

默认情况下destination ip不指向自身时内核会将流量丢弃，如果需要实现对任意流量的透明代理，首先需要接受任意流量，然后利用iptables进行NAT重写