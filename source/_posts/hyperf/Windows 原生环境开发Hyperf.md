---
title: Windows 原生环境开发Hyperf
tags: [Hyperf, Box, Windows]
date: 2021-02-28
---

很多同学在开发Hyperf 时, 会选择使用Docker, 但是Docker 在Windows 下的体验并不是很好, 有时候会出现各种各样的问题, 本文将介绍如何在Windows 原生环境下开发Hyperf

# Box 介绍
> Box 致力于帮助提升 PHP 应用程序的编程体验，尤其有助于 Hyperf 应用，管理 PHP 环境和相关依赖，同时提供将 PHP 应用程序打包为二进制程序的能力，还提供反向代理服务来管理和部署 Swoole/Swow 服务

- 管理PHP 版本及依赖, 规避多个PHP版本并存时兼容问题
- 打包代码为**二进制程序**, 一个`.exe`文件直接跑起来
- 反向代理(不是重点🤣)

# 安装
以Windows 为例, 直接从Github releases 下载对应的[二进制文件](https://github.com/hyperf/box/releases/download/v0.5.5/box_x64_windows.exe), 注意后续版本可能会有变化, 自行下载最新版本或执行`box self-update` 升级

正如官方文案所说, 下载的Box.exe 自身就是一个基于Box 打包出来的Hyperf 应用, 轻松实现自举

执行`box build-prepare` 初始化Box 所需的依赖, 梯子请自备

# 使用
Box 下载完成后, 即可使用Box 的Composer 来安装Hyperf , 这里选用Swow 版本骨架

```bash
box composer create-project hyperf/swow-skeleton 
```
耐心等待, 安装完成后, 进入项目目录, 执行
```bash
box php .\bin\hyperf.php start
// 也可以使用
box hyperf start
```
启动项目成功后, 可以愉快的在Windows 下开发Hyperf 了!

# 打包
打包是Box 另一项`神奇的能力`, 必须要尝试一下

当我们开发完成后, 可以使用Box 打包成二进制文件, 以便于交付部署

```bash
box build
```
> 打包完成后, 当前路径会出现一个名为hyperf.exe 的二进制文件，后续只需要通过hyperf start 命令即可启动该 Hyperf 应用

本文也算标题党了, 因为Box 了解的人还是非常少的, 但Windows 下开发Hyperf 的需求很多, 所以引申出Box 的使用

当然目前Box 还是个实验品, 有许多改进的空间, 有兴趣的同学可以参与进来, 一起完善Box, 全部都由PHP 代码实现, 认真看一下就可以参与贡献了
