---
title: 【手把手教你】用Swoft 搭建微服务(TCP RPC)
tags: [Swoft, 手把手教你]
date: 2018-06-06
---

## Swoft 是什么?

> Swoft 框架是首个基于Swoole 原生协程的新时代 PHP高性能协程全栈框架，内置协程网络服务器及常用的协程客户端，常驻内存，不依赖传统的 PHP-FPM
>
> 全异步非阻塞 IO 实现，以类似于同步客户端的写法实现异步客户端的使用，没有复杂的异步回调，没有繁琐的 yield，有类似 Go 语言的协程，灵活的注解
>
> 强大的全局依赖注入容器、完善的服务治理、灵活强大的 AOP、标准的 PSR 规范实现等

上面是官网描述, 感觉太官方, 我总结一下:

- 常驻内存
- 协程
- 学习曲线平滑
- 国内框架
- 开箱即用的RPC

## 如何搭建微服务?
首先确保已经可以正确搭建Swoft, 不清楚的可以查看[Swoft 官方文档](https://doc.swoft.org/master/zh-CN/quickstart/enviroment.html)

鉴于每个人的开发环境都不同, 这里选用官方`Docker` 作为开发环境

[Docker下载地址>>>](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

## 拉Docker 镜像

```bash
docker pull swoft/swoft
```

非常的简单
![title](https://leanote.com/api/file/getImage?fileId=5b17488cab644162a600088f)
这样就是成功了

为了方便理解

我们把swoft 复制两份

命名为`swoft-rpc`和`swoft-http`

`swoft-rpc`只开启`TCP` 服务
`swoft-http`只开启`HTTP` 服务

## 修改配置文件
把根目录的`.env.example`复制一份为`.env`

> .env 文件为swoft 配置文件, 最高优先级(覆盖config 下配置)

HTTP 用到的配置

```bash
    # Server
    PFILE=/tmp/swoft.pid
    PNAME=php-swoft
    TCPABLE=false        //是否同时启动TCP 服务器,这里用不到改为false
    CRONABLE=false
    AUTO_RELOAD=true
    AUTO_REGISTER=false
    ...
    # HTTP
    HTTP_HOST=0.0.0.0   //监听的网卡
    HTTP_PORT=80        //监听的端口
    HTTP_MODE=SWOOLE_PROCESS    //不用管
    HTTP_TYPE=SWOOLE_SOCK_TCP   //不用管
    ...//mysql 和redis 略过
    # User service (demo service)
    USER_POOL_NAME=user //别名
    USER_POOL_URI=192.168.1.214:8099,192.168.1.214:8099 //负载均衡,URI填写为RPC 的地址,注意Docker和宿主之间的关系
    USER_POOL_MIN_ACTIVE=5  //下面都不用管
    USER_POOL_MAX_ACTIVE=10
    USER_POOL_MAX_WAIT=20
    USER_POOL_TIMEOUT=200
    USER_POOL_MAX_WAIT_TIME=3
    USER_POOL_MAX_IDLE_TIME=60
    USER_POOL_USE_PROVIDER=false
    USER_POOL_BALANCER=random
    USER_POOL_PROVIDER=consul
```

RPC 用到的配置
```bash
    # TCP
    TCP_HOST=0.0.0.0    //监听的网卡
    TCP_PORT=8099       //监听的端口
    TCP_MODE=SWOOLE_PROCESS     //不用管
    TCP_TYPE=SWOOLE_SOCK_TCP    //不用管
    TCP_PACKAGE_MAX_LENGTH=2048 //最大链接数
    TCP_OPEN_EOF_CHECK=false    //不用管
```

## 启动Docker 容器

```bash
docker run -it --rm -p 8099:8099 -v E:\WWW\swoft-rpc:/var/www/swoft  swoft/swoft /bin/bash
```
这里用`-it`和`-v`方便调试
![title](https://leanote.com/api/file/getImage?fileId=5b175328ab64416496000aa3)
这样就是成功启动了
## 启动RPC 服务

```bash
php bin/swoft rpc:start
```

`RPC` 服务只需要单独启动`TCP` 服务器

有的同学`RPC` 和`TCP` 的关系可能还没弄清楚

这里`RPC` 服务和`TCP` 服务器可以类比为`Web` 服务和`HTTP` 服务器

> 监听HTTP 来实现Web 服务
> 监听TCP 来实现RPC 服务

就这样理解吧
![title](https://leanote.com/api/file/getImage?fileId=5b17581bab64416496000b69)
这样就是成功启动了

## 启动Web服务
也就是启动`HTTP` 服务器XD

新开一个终端来创建新容器

```bash
docker run -it --rm -p 9501:80 -v E:\WWW\swoft-http:/var/www/swoft swoft/swoft /bin/bash
```

这里端口改成9501, 因为本地开发环境已经用了80了:b

```bash
php bin/swoft server:start
```

![title](https://leanote.com/api/file/getImage?fileId=5b175d89ab64416496000c3b)
因为在之前把自动开启TCP 服务器禁用了

所以显示Disabled

这样也就是成功了!

访问一下`http://127.0.0.1:9501/`看下有没有问题

没问题的话, 可以看下官方提供的RPC demo `http://127.0.0.1:9501/rpc/call`

![title](https://leanote.com/api/file/getImage?fileId=5b17a444ab644162a60016c5)

大功告成!

是不是很简单!

这里只教大家如何搭建, 下一篇来理解如何使用RPC
