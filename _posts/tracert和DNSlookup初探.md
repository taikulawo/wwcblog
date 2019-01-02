---
title: tracert和DNSlookup初探
tags:
    - 网络
abbrlink: a06d
date: 2018-12-26 00:00:00
---

#### Dig

这是一个Linux自带的DNS查询命令，我将尝试解读它 :)

先糊上我自己Github Page原始域名的解析结果

<!--more-->

```
root@MyCP:~# dig +trace iamwwc.github.io

; <<>> DiG 9.10.3-P4-Ubuntu <<>> +trace iamwwc.github.io
;; global options: +cmd
.			37871	IN	NS	h.root-servers.net.
.			37871	IN	NS	a.root-servers.net.
.			37871	IN	NS	c.root-servers.net.
.			37871	IN	NS	k.root-servers.net.
.			37871	IN	NS	l.root-servers.net.
.			37871	IN	NS	g.root-servers.net.
.			37871	IN	NS	f.root-servers.net.
.			37871	IN	NS	j.root-servers.net.
.			37871	IN	NS	e.root-servers.net.
.			37871	IN	NS	b.root-servers.net.
.			37871	IN	NS	d.root-servers.net.
.			37871	IN	NS	m.root-servers.net.
.			37871	IN	NS	i.root-servers.net.
.			37871	IN	RRSIG	NS 8 0 518400 20190103010000 20181221000000 2134 . kLs0GPJxi19i0TPt347zkbZv+QF60obU3t1IEqsrt7uJDNlDYNveG3b1 H565FwQyZoqzuWs3i5tSyHl5TeeI70wPYUd1U7mX0ZZNBK+sGLzZGKFG zovUYRFg4R7ue6O3/s+wVUR2O6SOxR5Uq7IhCzmQZ2EF4IWaeaUEVf2w JqLT53Z5k27VOdYFPYha6lHzAgYXaWBjCD2Io5hh4DETwwliatsaKCa/ 7CxTPBsyNSieetlXh3MIEFhQerXq+RDdayXjO67r8cfbo6LOO0MWJwzV 4JLOP8iHG08PERjTjyxbMwB/8Z5ufORsRHIIL8JLzy9fz3b7AEYWZegJ +Jy71Q==
;; Received 525 bytes from 100.100.2.138#53(100.100.2.138) in 0 ms

io.			172800	IN	NS	ns-a3.io.
io.			172800	IN	NS	c0.nic.io.
io.			172800	IN	NS	ns-a1.io.
io.			172800	IN	NS	a0.nic.io.
io.			172800	IN	NS	a2.nic.io.
io.			172800	IN	NS	b0.nic.io.
io.			86400	IN	DS	64744 8 2 2E7D661097A76EAC145858E4FF8F3DDAE5EAEDFD527725BC6F8A943E 4FE23A29
io.			86400	IN	DS	57355 8 1 434E91E206134F5B3B0AC603B26F5E029346ABC9
io.			86400	IN	DS	57355 8 2 95A57C3BAB7849DBCDDF7C72ADA71A88146B141110318CA5BE672057 E865C3E2
io.			86400	IN	RRSIG	DS 8 1 86400 20190108200000 20181226190000 2134 . llX1lmA7GdCl1nisIhnXZhbmu8qDwrKpOZSWZ23s24sSqTauGGvxqeA+ jQmhT2xOVWDR84LHOell8MdDuCKDQy7jbaxDR8ShiWP0Iuoqo+/INlnv a0kmgZ3VuC+6dp/zPv79sPoc5cS2dwXhgZy/lbViAB3Y9bNBwcAcviW2 VJ8V3UT+MessxD8/fS3k8BjvfP3ksqlixC4WxtoBQBlgVUYeLQuNVy56 usI+K3RBgfOTG9cflsPoJsKFkXF/DeyVgvoIoMh6b/owk/E9rp60MGNk pbJuvDEIgkylcRSoS8hZ61b8mzxPUSH20ZZckRjfTPGWKsN4SXezN1TF dpCQxg==
;; Received 812 bytes from 192.33.4.12#53(c.root-servers.net) in 237 ms

github.io.		86400	IN	NS	ns1.p16.dynect.net.
github.io.		86400	IN	NS	ns2.p16.dynect.net.
github.io.		86400	IN	NS	ns-692.awsdns-22.net.
github.io.		86400	IN	NS	ns-1339.awsdns-39.org.
github.io.		86400	IN	NS	ns-1622.awsdns-10.co.uk.
2iui5t1khct6c5o8i2i67rppatgvegqo.io. 900 IN NSEC3 1 1 1 D399EAAB 2IV7T2DEE5N8V4AC5IHQK0MNI25BCHD7 NS SOA RRSIG DNSKEY NSEC3PARAM
2iui5t1khct6c5o8i2i67rppatgvegqo.io. 900 IN RRSIG NSEC3 8 2 900 20190117020103 20181227010103 52128 io. yYNIkzG/RJVQFaybJ63eAbmFobfhvdZfuC45ha7y0pFOJ3zJmhseKuik fdAXGrU/rbNi2pvHm4WwgISIeKWWaNR4o4htl4CbFqg1vUXOkea94xQH biP5WplrcGhakGg28MAIFLApioqrZt7h26rh04xD43T5VsodzBH+j+zn AYc=
ldl8iscjau4ngcq5m160b1gbj7je9bj1.io. 900 IN NSEC3 1 1 1 D399EAAB LDRAPKM9450C0KQMR5SC35COSF1MSSKB NS DS RRSIG
ldl8iscjau4ngcq5m160b1gbj7je9bj1.io. 900 IN RRSIG NSEC3 8 2 900 20190115151643 20181225141643 52128 io. AzoqPzzR+eLeunh5mGsjSoje70PhEHF429PvzvdobFHie6LlUL3SaVL4 JZc/1FgoqBy6dg89VgiKVEhAnc7duScvhPUTns7d1g2ciDhsb4HZDXtP SfiUDOk39LBtfLb7ulb5nqWGeD8kmFNIXEsixBjH+iE3Vvl2xlnSlitO 18M=
;; Received 710 bytes from 194.0.1.1#53(ns-a1.io) in 309 ms

iamwwc.github.io.	3600	IN	A	185.199.108.153
iamwwc.github.io.	3600	IN	A	185.199.109.153
iamwwc.github.io.	3600	IN	A	185.199.110.153
iamwwc.github.io.	3600	IN	A	185.199.111.153
github.io.		900	IN	NS	ns-1339.awsdns-39.org.
github.io.		900	IN	NS	ns3.p16.dynect.net.
github.io.		900	IN	NS	ns-1622.awsdns-10.co.uk.
github.io.		900	IN	NS	ns4.p16.dynect.net.
github.io.		900	IN	NS	ns-393.awsdns-49.com.
github.io.		900	IN	NS	ns1.p16.dynect.net.
github.io.		900	IN	NS	ns-692.awsdns-22.net.
github.io.		900	IN	NS	ns2.p16.dynect.net.
;; Received 332 bytes from 204.13.250.16#53(ns2.p16.dynect.net) in 174 ms
```

因为我使用了`+trace` ， dig 将开始从根域开始查询

###### 第一次查询
首先得到对 根域`.`的查询结果， `Received 525 bytes from 100.100.2.138#53(100.100.2.138) in 0 ms`

`100.100.2.138`是电信的运营商DNS，

```
.			37871	IN	NS	h.root-servers.net.
.			37871	IN	NS	a.root-servers.net.
.			37871	IN	NS	c.root-servers.net.
.			37871	IN	NS	k.root-servers.net.
.			37871	IN	NS	l.root-servers.net.
.			37871	IN	NS	g.root-servers.net.
.			37871	IN	NS	f.root-servers.net.
.			37871	IN	NS	j.root-servers.net.
.			37871	IN	NS	e.root-servers.net.
.			37871	IN	NS	b.root-servers.net.
.			37871	IN	NS	d.root-servers.net.
.			37871	IN	NS	m.root-servers.net.
.			37871	IN	NS	i.root-servers.net.
.			37871	IN	RRSIG	NS 8 0 518400 20190103010000 20181221000000 2134 . kLs0GPJxi19i0TPt347zkbZv+QF60obU3t1IEqsrt7uJDNlDYNveG3b1 H565FwQyZoqzuWs3i5tSyHl5TeeI70wPYUd1U7mX0ZZNBK+sGLzZGKFG zovUYRFg4R7ue6O3/s+wVUR2O6SOxR5Uq7IhCzmQZ2EF4IWaeaUEVf2w JqLT53Z5k27VOdYFPYha6lHzAgYXaWBjCD2Io5hh4DETwwliatsaKCa/ 7CxTPBsyNSieetlXh3MIEFhQerXq+RDdayXjO67r8cfbo6LOO0MWJwzV 4JLOP8iHG08PERjTjyxbMwB/8Z5ufORsRHIIL8JLzy9fz3b7AEYWZegJ +Jy71Q==
;; Received 525 bytes from 100.100.2.138#53(100.100.2.138) in 0 ms
```

37871 是 `TTL(time to live)` 表示这个DNS查询结果cache 多长时间，IN是internet，NA是namaspace，

`*.root-servers.net`则是13个采用了anycast的根域服务器集群

具体的每个字段的意思可以在

`http://www.zytrax.com/books/dns/ch15/#type`

找到

###### 第二次查询

下面开始查询 io 域

```
;; Received 812 bytes from 192.33.4.12#53(c.root-servers.net) in 237 ms
```

这里我不太明白的一点，每次查同一个域名，`*.root-servers.net` 的 * 字母一直变化

因为DNS是一个最快相应的查询过程，我猜测是不是对每个根域同时发送了查询请求

但每次挑选了最快的做为结果

###### 第三次查询

```
;; Received 710 bytes from 194.0.1.1#53(ns-a1.io) in 309 ms
```

从 `ns-a1.io` 处查询得到 `github.io.` 的 `namespace`

###### 第四次查询

```
;; Received 332 bytes from 204.13.250.16#53(ns2.p16.dynect.net) in 174 ms
```

最后查询到了 `iamwwc.github.io.`对应的A记录是 `185.199.108.153` ,而这个就是`Github page` 的 `ip`


到这里如果没有自定义域名，那么查询就结束了，往后浏览器拿着这个 `ip` 配合 域名 `iamwwc.github.io`

就可以得到网页了

如果设置了 `chaochaogege.com`，那么 访问 `iamwwc.github.io`之后会得到 301 永久重定向

再对 `chaochaogege.com` 进行 `DNS` 查询

由于我域名用了 `CDN`，最后查询到 `CNAME` 是 `chaochaogege.com.	600	IN	CNAME	chaochaogege.com.cdn.dnsv1.com.`

在国外解析成 `chaochaogege.com.	600	IN	CNAME	iamwwc.github.io.`

反正都是 `CNAME` , 还有走一遍 解析 `iamwwc.github.io`


#### tracert

这里我用了 `Windows` 路由查询工具来看一下自己的路由配置

照例糊 output

先来 `tracert` 的结果
```
Tracing route to baidu.com [220.181.57.216]
over a maximum of 30 hops:

  1    <1 ms    <1 ms    <1 ms  192.168.1.1
  2     2 ms     2 ms     2 ms  10.177.16.1
  3     2 ms     2 ms     2 ms  112.231.64.65
  4     8 ms     6 ms     6 ms  119.164.221.129
  5    11 ms    14 ms    14 ms  219.158.8.221
  6    11 ms    14 ms    14 ms  219.158.4.158
  7     *        *        *     Request timed out.
  8    28 ms    29 ms    26 ms  202.97.88.253
  9     *       26 ms     *     220.181.0.198
 10     *        *        *     Request timed out.
 11    29 ms    29 ms    29 ms  220.181.182.34
 12     *        *        *     Request timed out.
 13     *        *        *     Request timed out.
 14     *        *        *     Request timed out.
 15    28 ms    28 ms    28 ms  220.181.57.216
```

Windows 的 路由表

```
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0      192.168.1.1    192.168.1.104     35
        10.0.75.0    255.255.255.0         On-link         10.0.75.1    271
        10.0.75.1  255.255.255.255         On-link         10.0.75.1    271
      10.0.75.255  255.255.255.255         On-link         10.0.75.1    271
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      169.254.0.0      255.255.0.0         On-link   169.254.201.113    281
  169.254.201.113  255.255.255.255         On-link   169.254.201.113    281
  169.254.255.255  255.255.255.255         On-link   169.254.201.113    281
    172.21.93.240  255.255.255.240         On-link     172.21.93.241    271
    172.21.93.241  255.255.255.255         On-link     172.21.93.241    271
    172.21.93.255  255.255.255.255         On-link     172.21.93.241    271
      192.168.1.0    255.255.255.0         On-link     192.168.1.104    291
    192.168.1.104  255.255.255.255         On-link     192.168.1.104    291
    192.168.1.255  255.255.255.255         On-link     192.168.1.104    291
     192.168.56.0    255.255.255.0         On-link      192.168.56.1    281
     192.168.56.1  255.255.255.255         On-link      192.168.56.1    281
   192.168.56.255  255.255.255.255         On-link      192.168.56.1    281
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link      192.168.56.1    281
        224.0.0.0        240.0.0.0         On-link         10.0.75.1    271
        224.0.0.0        240.0.0.0         On-link     192.168.1.104    291
        224.0.0.0        240.0.0.0         On-link   169.254.201.113    281
        224.0.0.0        240.0.0.0         On-link     172.21.93.241    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link      192.168.56.1    281
  255.255.255.255  255.255.255.255         On-link         10.0.75.1    271
  255.255.255.255  255.255.255.255         On-link     192.168.1.104    291
  255.255.255.255  255.255.255.255         On-link   169.254.201.113    281
  255.255.255.255  255.255.255.255         On-link     172.21.93.241    271
===========================================================================
```


自己虚拟网卡比较多，排除那些没用了，这里只糊上 `Ethernet`


```
Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : DHCP HOST
   Description . . . . . . . . . . . : Intel(R) Ethernet Connection (3) I218-V
   Physical Address. . . . . . . . . : 50-7B-9D-E0-C9-64
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::95c4:cf60:ebd7:ff60%13(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.104(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : 26 December, 2018 20:36:24
   Lease Expires . . . . . . . . . . : 27 December, 2018 00:36:22
   Default Gateway . . . . . . . . . : 192.168.1.1
   DHCP Server . . . . . . . . . . . : 192.168.1.1
   DHCPv6 IAID . . . . . . . . . . . : 89160605
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-23-14-00-F3-50-7B-9D-E0-C9-64
   DNS Servers . . . . . . . . . . . : 202.102.128.68
                                       202.102.134.68
   NetBIOS over Tcpip. . . . . . . . : Enabled

```

先从 tracert开始

```
  1    <1 ms    <1 ms    <1 ms  192.168.1.1
```

在 以太网配置中 可以找到 `192.168.1.1` 是以太网卡的默认网关(DHCP服务器IP)
`192.168.1.104` 是 网卡 的ip

`On-link`指同一个网段，没必要路由，同一个网段只需要`ARP`来得到目的`MAC`

如果跨网段，不知道默认网关的MAC也是需要ARP来确定的

`10.177.16.1` 是 `DHCP` 的网关

最后跳到公网，剩下的就是公网上的网关IP