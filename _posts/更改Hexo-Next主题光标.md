---
title: 更改Hexo-Next主题光标
tags:
  - 花哨
abbrlink: 3b57
date: 2018-06-19 20:16:10
---

之前Google Host还能用的时候在[老D博客](https://laod.cn)看到网站图标更改，点击超链接的时候会光标会在滑稽和阴险之间变化，所以就想在Next主题中也加上。

<!--more-->
先去人家的网站上把他的光标的CSS扒下来。
然后定位到
`\themes\next\source\css\_custom`，打开custom.styl，然后加上
```
body {
	cursor: url(/images/huaji.png), auto;
}


a:hover {
	cursor: url(/images/yinxian.png), auto;
}
```
最后在***站点***目录下`source/`新建images文件夹，然后将两个光标图片放进去。
```
https://chaochaogege.com/images/huaji.png
https://chaochaogege.com/images/yinxian.png
```
最后重新push到page上面就OK了