---
title: Web编辑器-AceEditor使用过程踩坑记录
abbrlink: afbc
date: 2018-10-08 16:10:18
tags:
	- AceEditor
---




我想使用者可以打开多个不同的文件，所以使用一个Editor,就需要给每一个文件绑定一个Document,

从官网上写的

> Contains the text of the document. Document can be attached to several [`EditSession`](https://ace.c9.io/edit_session.html)s.
>
> At its core, `Document`s are just an array of strings, with each row in the document matching up to the array index.

看起来很美好，对每一个文件绑定一个Document对象，当进行文件切换的时候直接使用API`setDocument(doc)`就行了

<!--more-->

但这样做了之后会出现打字困难，光标不动等问题

![](https://chaochaogege.net/images/2018-09-20-1.gif)





正常的话应该是下面的样子

![](https://chaochaogege.net/images/201809201156.gif)



整个编辑框的状态都存放在`EditSession`中，所以如果想要在一个`Editor`中创建更多的文件，那么需要为每一个文件都创建一个新的`EditSession`

```javascript
import * as Ace from 'ace-builds'
let editor = Ace.edit('DOM')
let EditSession = Ace.require('ace/edit_session').EditSession
let newSession = new EditSession('','ace/mode/golang')
editor.setSession(newSession)
```

