---
title: 济南校园联通绕过拨号客户端使用PPPOE直接上网
abbrlink: 2ffa
date: 2018-09-04 23:36:38
tags:
    - Other
---


济南校园联通使用PPPOE登录
Windows客户端之后流程

#### 选择学校界面：
向服务器请求学校的具体信息
![http-show](/images/show.png)
<!--more-->

这里服务器直接回复了一个json文件，乱码的地方我用`***`代替了


```json
{  
   "status":0,
   "msg":"***",
   "data":[  
      {  
         "allowCode":0,
         "allowMsgCode":0,
         "autoLogin":"1",
         "base":"1",
         "brasIp":"124.128.40.39|124.128.40.7",
         "checkAccount":0,
         "clientOver":"@jndxedu",
         "clientStart":null,
         "linkUrl":"http://139.198.3.98/sdjd/",
         "name":"***",
         "phoneOver":null,
         "phoneStart":null,
         "pin":"0x01",
         "pinDescribe":1,
         "realUrl":"http://139.198.3.98/mgr/",
         "schoolId":510592
      },
      {  
         "allowCode":0,
         "allowMsgCode":0,
         "autoLogin":"1",
         "base":"1",
         "brasIp":"124.128.40.124",
         "checkAccount":0,
         "clientOver":"@sdldjsxywo201",
         "clientStart":null,
         "linkUrl":"http://139.198.3.98/sdjd/",
         "name":"***",
         "phoneOver":null,
         "phoneStart":null,
         "pin":"0x01",
         "pinDescribe":1,
         "realUrl":"http://139.198.3.98/mgr/",
         "schoolId":668220
      },
      {  
         "allowCode":0,
         "allowMsgCode":0,
         "autoLogin":"1",
         "base":"1",
         "brasIp":"123.233.122.77",
         "checkAccount":0,
         "clientOver":"@lxjxwo201",
         "clientStart":null,
         "linkUrl":"http://139.198.3.98/sdjd/",
         "name":"***",
         "phoneOver":null,
         "phoneStart":null,
         "pin":"0x01",
         "pinDescribe":1,
         "realUrl":"http://139.198.3.98/mgr/",
         "schoolId":640841
      },
      {  
         "allowCode":0,
         "allowMsgCode":0,
         "autoLogin":"1",
         "base":null,
         "brasIp":"124.128.40.39",
         "checkAccount":null,
         "clientOver":"@srzjida",
         "clientStart":"LNP",
         "linkUrl":"http://139.198.4.194:9999/",
         "name":"***",
         "phoneOver":"@srzjida",
         "phoneStart":"LNM",
         "pin":"0x01",
         "pinDescribe":1,
         "realUrl":"http://139.198.4.194/mgr/",
         "schoolId":1044729
      }
   ]
}
```
以我济南大学为例，
`clientover`是**最终**拨号帐号的后缀`@jndxedu`
`clientstart`是**最终**前缀，这里为空
`pin`为**最终**拨号密码的前缀

所以最终key和password格式如下（这个其实放到最后比较好）
```
key => clientstart + phonenumber + clientover 
password => pin + yourpassword
```

至于`phoneStart`干嘛用的不清楚



在学校选择完成点击拨号之后有发送了两个HTTP请求，check和authlogin



#### check 过程
![两个HTTP请求](/images/check-authlogin.png)

从字段名(check)推测是检查用户名和密码是否正确，
然后服务器回复了`success`，接着进行`authlogin`过程。
![http-check](/images/check.jpg)

###### check过程服务器的响应
![check的响应](/images/checkresponse.png)

#### authlogin 过程
![](/images/authlogin.jpg)
发给服务器进行认证的手机号和学校编号，然后紧接着就进行了最关键的`pppoe`拨号
![authlogin过程的HTTP响应](/images/loginresponse.png)


![PPPOE](/images/ppp.jpg)
因为我是济南大学，所以从上面的`json`文件就可以知道帐号是`phonenumber@jndxedu`
密码是你宽带密码然后在前面加上了一个`0x01`，这是一个特殊字符，没办法直接打出来
这里直接用代码打印，然后复制到windows自带`pppoe`拨号客户端的密码开头处
```golang
package main

import (
	"fmt"
)

func main() {
	var b byte = 0x01
	fmt.Println(string(b))
}

```

就这样，完成了，联通的简单，没恶心人的心跳包，认证完之后就可以直接用了，帐号密码固定，电信不发心跳直接掉线

