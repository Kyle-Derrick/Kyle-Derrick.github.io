---
title: 'CentOS7 使用高版本gcc'
author: ['kyle']
date: '2025-07-22T22:47:31+08:00'
tags:
- Linux
- CentOS

keywords:
- Linux
- CentOS
- gcc
---

直接安装 比如：devtoolset-8-gcc-g++ , 或其他版本


然后通过 如下命令启用并进入对应版本环境：
```sh
scl enable devtoolset-8 bash
#或
source /opt/rh/devtoolset-8/enable
```

如需直接替换：
```sh
ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc
#其他类似
```