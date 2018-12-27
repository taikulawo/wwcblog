---
title: 单播ARP
abbrlink: 5b18
date: 2018-12-27 00:00:00
---

嗯，今天挺高产

为了准备网络考试，研究了一些协议，这里说一下ARP

#### 单播ARP

我们说认识的ARP都是广播

也即
Ethernet 的 dst 地址 为 `ff:ff:ff:ff:ff:ff`
而ARP中的 target MAC 置为 `00:00:00:00:00:00`

就像这样

<!--more-->

![](https://chaochaogege.net/images/image_7.png)

这里查询了一下我网卡的MAC地址


所以网上清一色说ARP协议以太网帧都是`ff`

-------------------------------------------

但今天抓包的时候发现一个问题 / 抓包真是好，能有未知发现 :) 

![](https://chaochaogege.net/images/image_6.png)

可以看到，我网卡发送了一个单播ARP

以太网帧的dst竟然是我自己的MAC

这就好像问 wwc ，请问你是 wwc 吗？？

------------------------------------
这其实是协议规定的一种

目的是为了刷新ARP的缓存

操作系统会间歇性的发送给目标一个单独的 ARP ，如果目标没有回复，那么说明目标的 Ethernet 地址更改了，那么OS就可以删除这个条目
回复了就可以刷新这个缓存

通过这种方式可以更新缓存更不需要向**整个网络**发送ARP

更详细的信息可见

`https://networkengineering.stackexchange.com/questions/28803/arp-request-unicast-mac`

> Unicast Poll -- Actively poll the remote host by
periodically sending a point-to-point ARP Request
to it, and delete the entry if no ARP Reply is
received from N successive polls.  Again, the
timeout should be on the order of a minute, and
typically N is 2.


刚开始觉着 ARP 挺简单的，就是发个广播完事了

但 RFC 上规定了许多的情况

#### ARP探测器

我这里开始翻译 RFC 了

在开始使用`IPV4`地址之前（不管这个地址是手动设定的还是 `DHCP` 还是其他的原因），host 必须发送一个ARP广播

比如我们A想要使用 `192.168.8.2`

发送者A必须在 `sender MAC` 填写自己的 `MAC` 地址
sender IP 必须**全部置0**（防止污染同一个链路中的其他主机的缓存，因为如果这个IP被使用，其他的主机会将 sender 的IP 加入自己的缓存，但这样已经说明这个IP不能被sender使用）

`target MAC` 将会被忽略，但**should**全部置0

`target ip` 应该添加 `sender` 想要使用的ip

如果经过一段时间后，在这个网卡接口收到了数据包（不管是`request`还是`reply`），`sender ip` 是 `192.168.8.2` , 说明这个IP被使用了

那么必须进行报错

除此之外，如果收到了数据包，`target ip` 是 `192.168.8.2`，并且`sender MAC` 不是 A 的 `MAC`, 那也要认为出错，出现这种情况的原因可能是两个或者更多的host因为某些原因，不经意间使用了相同的IP，并且都在同时探测这个`IP`是否能够使用

上面的来源

`https://tools.ietf.org/html/rfc5227#page-5`

#### 宣布一个地址

经过上面的探测，如果可以使用 `192.168.8.2`，那么 `ARP` 的实现者必须发送一个广播，来告知其他的主机我拥有了这个IP

整个过程和上面的地址探测差不多，仅需要将 `sender` 和 `target` 的`IP`都设置成新的`IP`（这里是 `192.168.8.2`）