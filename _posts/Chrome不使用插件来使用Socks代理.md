---
title: Chrome不使用插件来使用Socks5代理
abbrlink: d7b8
date: 2018-06-20 19:28:14
tags:
	- Chrome
	- Socks
---


先上一则新闻
>However, we continue to receive large volumes of complaints from users about unwanted extensions causing their Chrome experience to change unexpectedly — and the majority of these complaints are attributed to confusing or deceptive uses of inline installation on websites. As we’ve attempted to address this problem over the past few years, we’ve learned that the information displayed alongside extensions in the Chrome Web Store plays a critical role in ensuring that users can make informed decisions about whether to install an extension. When installed through the Chrome Web Store, extensions are significantly less likely to be uninstalled or cause user complaints, compared to extensions installed through inline installation.
Later this summer, inline installation will be retired on all platforms. Going forward, users will only be able to install extensions from within the Chrome Web Store, where they can view all information about an extension’s functionality prior to installing.


[原文Improving extension transparency for users](https://blog.chromium.org/2018/06/improving-extension-transparency-for.html)

也就是Google打算以后让你只能在Chrome web store安装插件。
但是我打算在学校机房临时科学上网，`Shadowsocks`也不用了，手里只有控制台的`v2ray`，全局代理是不用想了。。用代理的话需要`Switchyomage`这些插件来让Chrome走代理，但是不能科学上网就不能安装插件，最后成了一个死循环，所以只能给Chrome加参数启动来解决了。

假定现在在Chrome的目录下，Windows平台`shift`加鼠标右键打开cmd，然后
`chrome.exe --proxy-server="socks5://127.0.0.1:1080"` 运行就OK了

当然 `socks5`可以换成`http`,`socks`,`socks4`。

[原文How to set Google Chrome’s proxy settings in command line on Linux?](https://www.systutorials.com/241062/how-to-set-google-chromes-proxy-settings-in-command-line-on-linux/)