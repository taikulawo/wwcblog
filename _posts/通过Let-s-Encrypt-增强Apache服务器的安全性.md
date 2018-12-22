---
title: 通过Let-s-Encrypt-增强Apache服务器的安全性
abbrlink: 42476
date: 2018-05-25 11:15:29
tags:
	- Apache
	- 翻译
	- LetsEncrypt
---


###  **通过Let's Encrypt 增强Apache服务器的安全性**

####  **介绍：**
这个教程将向你展示如何在运行有Apache Web Server的CentOS系统上设置TLS/SSL证书。另外，我们还将涉及如何通过自动化的工作来延长证书的时间。
在现今的Web服务中，SSL证书被用于加密服务器与客户端之间的流量，让用户通过额外的安全措施来访问你的应用，Let's Encrypt 提供一个免费而又简单的措施来获得和安装受信任的证书。
<!-- more -->

####  **前提条件**
为了完成这个教程，你需要：

> 一个没有root权限但是有sudo权限的用户账户。
> 至少有一个已经为你的服务器配置过A或者GNAME记录的域名。确切的过程取决于您的域名注册商或者托管的服务。如果你已经购买了一个域名并且希望和DigitalOcean的域名服务一起使用，你可以阅读关于[设置你自己的域名服务][1]的指导来设置正确的记录。

通过这篇教程，我们会为 example.com域名安装 Let’s Encrypt 的证书。这个域名将会通篇引用，但是你当你按照教程做的时候应该用你自己的域名替换它。

当你准备继续的时候，通过你的sudo用户账户来登录你的服务器。
###  **第一步------安装需要的软件**
在我们安装`certbot` Let's Encrypt 客户端并且生成SSL证书之前，我们需要安装Apache Web服务器(如果你还没有安装它),我们也需要安装`mod_ssl`模块来传输被加密的流量。最终，我们还会需要添加EPEL仓库，它能够为CentOS提供额外的软件包，当然包括了我们需要的`certbot`。

为了安装EPEL仓库，首先敲入如下命令：

    $ sudo yum install epel-release
现在你已经可以接入额外的仓库了，为了安装需要的所以软件包，仅需要敲入一下命令：

    $ sudo yum install httpd mod_ssl python-certbot-apache
你现在应该有了所有增强你的网站安全性的软件包了。
###  **第二步-----配置访问Apache**
在我们请求一个证书之前，我们需要确保Apache在我们的服务器上正在运行并且可以被外部世界访问。
为了确保Apache正常运行，敲入：

    $ sudo systemctl start httpd
通过检查服务的状态来识别Apache是否在运行

    Output
    httpd.service - The Apache HTTP Server
    Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
    Active: active (running) since Thu 2017-01-05 16:47:06 UTC; 1h 7min ago
     Docs: man:httpd(8)
           man:apachectl(8)
    Main PID: 9531 (httpd)
    Status: "Total requests: 10; Current requests/sec: 0; Current traffic:   0 B/sec"
    CGroup: /system.slice/httpd.service
           ├─9531 /usr/sbin/httpd -DFOREGROUND
           ├─9532 /usr/sbin/httpd -DFOREGROUND
           ├─9533 /usr/sbin/httpd -DFOREGROUND
           ├─9534 /usr/sbin/httpd -DFOREGROUND
           ├─9535 /usr/sbin/httpd -DFOREGROUND
           └─9536 /usr/sbin/httpd -DFOREGROUND

    Jan 05 16:47:05 centos-512mb-nyc3-01 systemd[1]: Starting The Apache HTTP Server...
    Jan 05 16:47:05 centos-512mb-nyc3-01 httpd[9531]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using ::1. Set the 'ServerName' directive globally to suppress this message
    Jan 05 16:47:06 centos-512mb-nyc3-01 systemd[1]: Started The Apache HTTP Server.
顶部状态应该是Active。
下一步，防火墙中80和443端口开启，如果你的防火墙没有运行，你可以跳过这一步。
如果你的防火墙是firewalld，并且其正在运行，你可以通过下面的命令打开端口：

    $ sudo firewall-cmd --add-service=http
    $ sudo firewall-cmd --add-service=https
    $ sudo firewall-cmd --runtime-to-permanent
如果你正在运行的防火墙是iptables，那么你使用的命令将取决于你当前的规则集。对于一个基础规则集，你可以通过下面命令来田间HTTP和HTTPS访问：

    $ sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
    $ sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
使用curl来确定你的网站可以被访问：

    $ curl example.com
你的网站的主页面应该呈现的是HTML网页。在mod_ssl包默认配置了一个自签的SSL证书之后，如果你使用`-k`参数允许不信任的证书，你可以通过HTTPS检查你的域名：

    $ curl -k https://example.com
如果成功，你应该会看到同样的输出结果并且可以断定SSL端口已经打开了。
### 第三步---从Let's Encrypt申请一个SSL证书
Apache已经配置完成，我们现在要给我们的域名申请一个SSL证书。
通过Let's Encrypt的客户端`certbot`生成Apache的SSL证书是非常直截了当的。
客户端会自动的获取和安装一个新的将域名作为参数提交的有效的SSL证书。
如果你想要为多个域名或者子域名安装单独的一个证书，你可以将域名作为额外的参数来运行下面的命令。命令中的第一个域名会被Let's Encrypt作为基础域名来创建证书，所以因为这个原因，我们推荐你将顶级域名作为第一个参数，后面加上任何额外的子域名或者别名：

    $ sudo certbot --apache -d example.com -d www.example.com

如果想要进行交互式的安装并且只想获得一个给单一域名的证书，运行下面的命令：

    $ sudo certbot --apache -d example.com
`certbot`也可以在证书请求过程中提示你域名的信息。如果想要使用这个功能。不加任何域名地调用`certbot`：

    $ sudo certbot --apache
你将会看到一步步的指南来自定义你的证书选项。你需要提供一个Email用作口令回复和通知。如果你在命令中没有为你的域名加入定制选项，你将会看到上面所提及的讯息。如果你的虚拟主机文件没有通过`ServerName`指令明确域名，你将会被要求选择虚拟主机文件（默认的`ssl.conf`应该会起到作用）。
你也可以选择同时开启`http`和`https`访问或者强制重定向到`https`。为了更安全，如果你没有特殊的需求去允许未经加密的链接，那么推荐使用安全选项。

当安装成功完成，你应该看到一条相似的消息：

    IMPORTANT NOTES:
    - Congratulations! Your certificate and chain have been saved at
      /etc/letsencrypt/live/example.com/fullchain.pem. Your cert
     will expire on 2016-04-21. To obtain a new version of the
     certificate in the future, simply run Let's Encrypt again.
     - If you lose your account credentials, you can recover through
      e-mails sent to user@example.com.
     - Your account credentials have been saved in your Let's Encrypt
    configuration directory at /etc/letsencrypt. You should make a
     secure backup of this folder now. This configuration directory will
      also contain certificates and private keys obtained by Let's
     Encrypt so making regular backups of this folder is ideal.
     - If you like Let's Encrypt, please consider supporting our work by:

      Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
     Donating to EFF:                    https://eff.org/donate-le`

生成的证书文件应该放在`/etc/letsencrypt/live`中，并且是在你的主域名命名的子目录下。
在我们检查SSL证书之前，我们应该修改CentOS默认SSL证书配置来使它更安全。

###  第四步--为Apacha选择更安全的SSL证书
CentOS的默认Apache配置有点过时了，很容易受到近期的安全问题的困扰。

为了配置更安全的SSL相关的选项，打开`ssl.conf`文件（或者在安装Let's Encrypt在提示时你选择的那个虚拟主机文件）：

    $ sudo nano /etc/httpd/conf.d/ssl.conf
我们首先应该找到`SSLProtocol`和`SSLCipherSuite`那一行，删除它们或者将它们注释。下面的我们将要粘贴的配置文件将提供比CentOS的Apache默认文件更安全的设置。

                /etc/httpd/conf.d/ssl.conf
    . . .
    # SSLProtocol all -SSLv2
    . . .
    # SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5:!SEED:!IDEA
下一步，为了使得Apache的SSL证书更安全，我们将会使用由
[Remy Van Elst][2] 在[Cipherli.st][3] 网站上的建议。这个网站被设计用来为流行软件提供easy-to-consume的加密设置。
你可以在[这里][4]阅读更多关于他有关Apache选择的决定。

>Note: The suggested settings on the site linked to above offer strong security. Sometimes, this comes at the cost of greater client compatibility. If you need to support older clients, there is an alternative list that can be accessed by clicking the link on the page labelled "Yes, give me a ciphersuite that works with legacy / old software." That list can be substituted for the items copied below.
The choice of which config you use will depend largely on what you need to support. They both will provide great security.

我们可以复制整个的设置而仅仅只需要做两个细微的改动。
花点时间阅读[HTTP Strict Transport Security, or HSTS][5] 特别是关于预加载的功能。预加载HSTS提高了安全性。但是如果意外启用或者启用不当会造成深远影响。在这个教程当中，我们不会预加载这个设置，但是你如果去欸的那个你能够清楚结果，那么你可以修改这个设置。
我们将要做的另一个改变就是注释`SSLSessionTickets`指令，因为在CentOS 7中的Apache版本中不可获得。

粘贴以下内容

>     . . .
>    </VirtualHost>
>    . . .
>
>   \# Begin copied text
>    \# from https://cipherli.st/
>    \# and https://raymii.org/s/tutorials/Strong_SSL_>Security_On_Apache2.html
>
>    SSLCipherSuite >EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
>SSLProtocol All -SSLv2 -SSLv3
>SSLHonorCipherOrder On
>\# Disable preloading HSTS for now.  You can use the >commented out header line that includes
>\# the "preload" directive if you understand the >implications.
>\#Header always set Strict-Transport-Security >"max-age=63072000; includeSubdomains; preload"
>Header always set Strict-Transport-Security >"max-age=63072000; includeSubdomains"
>Header always set X-Frame-Options DENY
>Header always set X-Content-Type-Options nosniff
>\# Requires Apache >= 2.4
>SSLCompression off
>SSLUseStapling on
>SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
>\# Requires Apache >= 2.4.11
>\# SSLSessionTickets Off

当你已经结束了上面的操作之后，你可以保存关闭文件了。
下一步，通过敲入下面的命令来检查你的配置文件的语法错误。

    $ sudo apachectl configtest
输出是

    . . .
    Syntax OK
只要最后一行是`Syntax OK`，你就可以继续。如果你没有看到这个，在继续之前检查文件的错别字。
现在，通过敲入下面的命令重启Apache服务：

    $ sudo systemctl restart httpd
你的服务器现在应该已经可以通过安全的SSL设置来支持你的网页的。

###  第五步--检查你的证书状态
你可以通过下面的链接来识别你的SSL证书的状态（不要忘了用你的主域名来替代example.com）

    https://www.ssllabs.com/ssltest/analyze.html?d=example.com&latest
你现在应该能够使用`https`前缀来访问你的网页了，在写作本文的，这些设置给与了A+的评分。
###  第六步--设置自动续期
Let's Encrypt证书有90天的有效期，但是推荐你每60天就续期来提高你的容错率。`certbot` Let's Encrypt客户端有一个续期的命令来自动检查当前安装的证书并且如果他们还有30天到期那就去续期。
为了给所有安装的域名触发续费进程，你应该运行：

    $ sudo certbot renew
因为我们最近安装了证书，这个命令仅仅会检查到期日期并且打印一条信息通知证书还不需要续费。输出应该和下面相似：
>Saving debug log to >/var/log/letsencrypt/letsencrypt.log
>
>--------------------------------------------------->----------------------------
>Processing /etc/letsencrypt/renewal/example.com.conf
>--------------------------------------------------->----------------------------
>Cert not yet due for renewal
>
>The following certs are not due for renewal yet:
  /etc/letsencrypt/live/example.com/fullchain.pem (skipped)
No renewals were attempted.

注意：如果你为多个域名创建了多个帧数，只有主域名会被输出，但是续期应该对所有包含在这个证书里面的域名都有效。
确保你的证书不会过期的一个实用方法是创建一个自动续期的作业。例如，因为续期首先检查到期日期并且只有当证书的有效期少于30天时才会续期，所以创建一个每一周或者每一天自动运行的作业是安全的。
让我们创建一个`crontab`来创建一个新的作业来每一天都运行续期的命令。通过root用户来编辑`crontab`。

    $ sudo crontab -e
包含下面的内容，所有的都放在一行。

    . . .
    30 2 * * * /usr/bin/certbot renew >> /var/log/le-renew.log
保存并且退出。这是创建一个每天在下午两点半运行`certbox renew`命令的cron作业。由命令产生的输出将会被重定向到`/var/log/le-renew.log`。


本文翻译自[How to Secure Apache with Let's Encrypt on CentOS 7][6]。


  [1]: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean
  [2]: https://raymii.org/s/static/About.html
  [3]: https://cipherli.st/
  [4]: https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html
  [5]: https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security
  [6]: https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-centos-7
