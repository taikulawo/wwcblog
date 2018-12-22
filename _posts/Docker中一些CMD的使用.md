---
title: Docker中一些CMD的使用
abbrlink: 999b
date: 2018-10-08 16:09:20
tags:
	- Docker
---


#### 通过`docker attach`来绑定`docker container`的`stdin, stdout, stderr`
前提条件是创建docker container指定`-i`选项

全部使用golang的镜像为例
###### 创建并运行container
```
docker run --name container -dti golang bash
```

绑定当前终端的`std*`
```
docker attach container
```

如果在第一步使用如下
```
docker run --name container -dt golang bash
```

则在attach之后虽然也会出现`root&****:/go#`，但是没有办法输入字符，只有指定`interactive(-i)`才可以
这个选项的意思是始终打开`stdin`，即使没有`attach`

<!--more-->



因为要将`mysql`部署到`container`当中，当时启动完容器之后直接

```bash
docker exec -it databse bash
```

结果因为mysql自己有配置文件，初始化还没有完成，当在bash里面登录mysql的时候

总是会出现

```b
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

如果是这种情况，等一下再使用mysql就没有问题了。



#### Dockerfile COPY 目录文件

比如想要将`./sql/`文件夹copy到Container `/go/sql`，

如果使用

```
COPY sql/ /go/
```

那么最后的结果是将`sql`下的文件copy到了`/go/`

看起来是对的，将sql带着文件夹一起copy到`/go/`下，



但事实上应该是

```
COPY sql/ /go/sql/
```



#### Container 远程调试

这里以调试`golang`代码为例

在本地开发完之后部署到`server`的`container`上总会有一些莫名其妙的问题，所以干脆直接远程调试

调试器用的是`delve`

先启动我们的整个服务 `docker-compose up -d`

`docker-compose.yml`文件

```dockerfile
# 一共三个容器，使用时注视到debug-server，调试过程注释掉web

version: "3"
services:
  web:
    ports:
      - "8086:8086"
    image: iamwwc/onlinecode:latest
    volumes:
      - onlinecodetemp:/onlinecodetemp
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      CODE_WORK_DIR: /go/src/chaochaogege.com/onlinecode
    networks:
      - onlinecode-net
    container_name: onlinecodeapp
    restart: on-failure
  db:
    image: mysql:8.0
    container_name: onlinecodedb
    volumes:
      - onlinecode-database:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: mysqlrootpassword
      MYSQL_PASSWORD: mysqlpassword
      MYSQL_USER: mysql
      MYSQL_DATABASE: onlinecode
    ports:
      - "3300:3306"
    networks:
      - onlinecode-net
    restart: always
#  debug-service:
#    image: ccr.ccs.tencentyun.com/wwc-docker-images/onlinecode:debug-server-run-dlv
#    ports:
#      - "8086:8086" #api port
#      - "8088:8088" #debug server
#    container_name: debug-server
#    volumes:
#      - ./:/go/src/chaochaogege.com/onlinecode
#      - onlinecodetemp:/onlinecodetemp
#      - /var/run/docker.sock:/var/run/docker.sock
#    security_opt:
#      - 'seccomp:unconfined'
#    cap_add:
#      - SYS_PTRACE
#    #restart: always
#    networks:
#      - onlinecode-net
#    environment:
#      CODE_WORK_DIR: /go/src/chaochaogege.com/onlinecode
networks:
  onlinecode-net:
    driver: bridge

volumes:
  onlinecodetemp:
  onlinecode-database:
```





由于问题代码在容器里头，我们首先`copy`出来

```bash
docker cp onlinecodeapp:/go/src/chaochaogege.com/onlinecode .
```

然后下载我编译了delve环境的镜像

```bash
docker pull ccr.ccs.tencentyun.com/wwc-docker-images/onlinecode:debug-server
```

这个镜像默认运行`tail -f /dev/null`

容器不会自己退出，便于拿到`shell`进去操作

然后将我们的代码挂载进去，由于我的容器在单独的`onlinecode-net`子网之中，所以还需要加入子网

启动调试容器

```
docker run -d -v /root/copyfromonlinecode/onlinecode/:/go/src/chaochaogege.com/onlinecode --security-opt=seccomp:unconfined -p 8088:8088 -p 9000:8086 --name debug-server --rm --network onlinecode_onlinecode-net ccr.ccs.tencentyun.com/wwc-docker-images/onlinecode:debug-server
```

这里`expose`了容器两个端口，

- 8086是服务端口，提供全部的资源
- 8088是调试端口，用来本地连接

**上面的是容器expose的端口，外部访问需要连接expose的host端口**



拿到调试容器的`shell`

```
docker exec -it sh
```



然后在容器里面启动`debug process`，因为代码已经部署上去，没有源代码，只有一个可执行文件，所以用`exec`来绑定

```
dlv exec ./onlinecode --api-version 2 --listen 0.0.0.0:8088 --headless
```

由于服务器在上海，所以要监听`0.0.0.0`，避免监听不到连接请求



代码是一个web服务，所以Chrome访问，就可以看到前端界面

```
x.x.x.x:9000
```



现在配置本地的连接

我IDE用的是`Goland`，所以直接新建一个配置文件`go remote`

`host`填写服务器`ip`

端口填写`9000`

最后下断点，然后构造条件去触发断点



**Docker Run中 `-p 8086:9000`是将 Container 9000端口映射到host的8086*