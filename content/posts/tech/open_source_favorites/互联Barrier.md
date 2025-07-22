---
title: '互联Barrier安装记录'
author: ['kyle']
date: '2025-07-22T23:17:36+08:00'
tags:
- Linux
- 开源工具

keywords:
- Linux
- 开源工具
---

## 安装
### Windows & MacOS
直接去 Github 上下载[安装包](https://github.com/debauchee/barrier/releases) 

### Ubuntu
```shell
sudo apt install barrier
```

## 设置
### 服务端设置
#### “设置服务端”
在里边新增屏幕，名字用客户端的设置名字（不能自己随便给，看客户端的设置里）
#### 关闭SSL
#### 防火墙允许端口

### 客户端设置
#### 关闭ssl

#### 设置服务端ip

## 常见问题
### Ubuntu 无法启动，提示Wayland问题
> 日志告警：WARNING: cursor may not be visible
> 也用该方法解决

修改 `/etc/gdm3/custom.conf`
```shell
#取消注释
#WaylandEnable=false
```
```shell
sudo systemctl restart gdm3
```

### 客户端无法连接服务端，日志显示一直在尝试连接，但是连接超时
- 检查客户端和服务端都关闭ssl
- 点击设置服务端按钮，进入有小电脑的配置页面
- 删除旧的客户端小电脑，如果没有，直接进入第三步新增
- 新增客户端小电脑，屏幕名命名为ubuntu的用户名

### 服务端客户端切换之后鼠标锁死无法移动
解决方案：屏蔽缩放都调整为100%

### 服务端控制后，鼠标进入ubuntu桌面后看不到光标