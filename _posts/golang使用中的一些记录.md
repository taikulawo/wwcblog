---
title: golang使用中的一些记录
abbrlink: bd5
date: 2018-10-08 16:09:53
tags:
	- Golang
---




一直在想为什么使用包内的函数设置一些状态之后别的函数能够直接使用

比如golang下的log包，可以`New`一个log来用。也可以直接setFlags，然后直接log.Printf来用

刚才看了一下log包下的实现

在log下有一个全局变量，`stdlog`，实际上使用包内的函数就是在设置这个对象的状态



#### Dep包管理工具

1. `[[constraint]]`

   这就是你想要的library的源，可以选择分支或者版本号，`source`则是指想要从哪里下载这些库

2. `[[override]]`

   覆写，如果由dep下载的依赖不符合你的要求，就可以指定这个选项，来重新指定直接依赖（你项目使用的库）或者间接依赖（你依赖的库的依赖）的版本号或者分支

<!--more-->

因为我用dep来管理自己的代码，其中使用了`docker for golang SDK`，但是自动解决的依赖总是会报`unsolved reference`的错误，所以直接去`github`上找了一个含有需要的reference的版本

```toml
[[override]]
  name = "github.com/docker/distribution"
  branch = "master"

[[constraint]]
  branch = "master"
  name = "github.com/docker/docker"

```

运行`dep ensure`之后锁定的版本如下：

```toml
[[projects]]
  branch = "master"
  digest = "1:4189ee6a3844f555124d9d2656fe7af02fca961c2a9bad9074789df13a0c62e0"
  name = "github.com/docker/distribution"
  packages = [
    "digestset",
    "reference",
  ]
  pruneopts = "UT"
  revision = "16128bbac47f75050e82f7e91b04df33775e0c23"

[[projects]]
  branch = "master"
  digest = "1:1e52d460eb324ffe9431c4035582c66f2658d2291395b2b8446806de19d906db"
  name = "github.com/docker/docker"
  packages = [
    "api",
    "api/types",
    "api/types/blkiodev",
    "api/types/container",
    "api/types/events",
    "api/types/filters",
    "api/types/image",
    "api/types/mount",
    "api/types/network",
    "api/types/registry",
    "api/types/strslice",
    "api/types/swarm",
    "api/types/swarm/runtime",
    "api/types/time",
    "api/types/versions",
    "api/types/volume",
    "client",
    "errdefs",
    "pkg/stdcopy",
  ]
  pruneopts = "UT"
  revision = "be79d286ea32a7ab8ea7f9f9f36feb5407318d8a"
```

revision就是这个库最后的commit的id

比如`docker distribution`，因为master分支代码一直会变动，使用这种方式锁定版本，只要代码仓库源没有被删除那就可以一直使用

[这个就是当前使用的版本（以commit id为标准）](https://github.com/docker/distribution/commit/16128bbac47f75050e82f7e91b04df33775e0c23)



#### Golang Module

`golang` 1.11加入了 `module`支持，现在终于不用把代码放到`$GOPATH`下面了

开一个文件夹，假如你的项目名叫做`opensoftware`，域名是`myexample.com`

如果没有域名可以用`github`仓库地址来初始化

```
go mod init myexample.com/opensoftware
```



或者

```
go mod init github.com/your-github-name/your-project-name
```



Go会在文件夹里面创建一个`go.mod`文件，里面第一行标识了你的项目路径

如果你想要引用自己写的代码，需要用你的域名加项目路径引用

比如`opensoftware`下有一个`common`包

通过如下方式引用

```
import "myexample.com/opensoftware/common
```

表示初始化当前目录。



`Go FAQ`

> ###### Do modules work with relative imports like `import "./subdir"`?
>
> In modules, there finally is a name for the subdirectory. If the parent directory says "module m" then the subdirectory is imported as "m/subdir", no longer "./subdir".



这个玩意和`npm init`不同，`mod`必须指明目录

否则会出现

```
go: cannot determine module path for source directory /your/path (outside GOPATH, no import comments)
```



下载依赖的话我是直接在当前文件夹下面go get

