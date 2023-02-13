---
title: 一分钟搭建Hyperf
tags: [Hyperf, 手把手教你]
date: 2019-07-11
---

# 前置依赖
1. Docker
2. Git
3. PHPStorm

# 计时开始

### 1.下载Hyperf Docker 镜像

```bash
docker pull hyperf/hyperf
```

![title](5d272e3cab6441760500347f.png)
### 2.下载Hyperf

```bash
git clone https://github.com/hyperf/hyperf-skeleton.git
```

![title](https://leanote.com/api/file/getImage?fileId=5d272edbab6441760500349c)
### 3.等待下载完成

> ======================> 98%

### 4.配置PHPStorm

打开`hyperf-skeleton`, 如下图所示配置`PHPStorm`
![title](https://leanote.com/api/file/getImage?fileId=5d273552ab644178040035bf)

> 如果Server 选项里无法找到Docker,可能是Docker 没有暴露守护进程

![title](https://leanote.com/api/file/getImage?fileId=5d2732c3ab64417804003538)

点击配置旁边的运行按钮, 如下图示说明容器已经欢快的运行起来了

![title](https://leanote.com/api/file/getImage?fileId=5d27367eab644176050035ff)

### 5.安装Composer 组件

进入刚才部署的容器

```bash
docker exec -it hyperf /bin/bash
 ```
进入容器后运行

```bash
cd /home && composer install
```

![title](https://leanote.com/api/file/getImage?fileId=5d27381cab6441760500366d)

这一步到了安装`Hyperf` 组件的界面, 有需要的可以选择安装, 不需要可以全选`n`

安装完成后运行`php bin/hyperf.php start`, 然后打开浏览器请求[http://127.0.0.1:9501](http://127.0.0.1:9501)

![title](https://leanote.com/api/file/getImage?fileId=5d273a40ab644176050036d7)

完成!
### 6.最后
配置中的`Command`修改为`php /home/bin/hyperf.php start`, 就可以愉快的一键重启了
![title](https://leanote.com/api/file/getImage?fileId=5d273b0fab6441760500370a)

