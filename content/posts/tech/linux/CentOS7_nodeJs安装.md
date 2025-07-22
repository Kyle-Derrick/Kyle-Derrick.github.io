---
title: 'CentOS7 上安装 NodeJs'
author: ['kyle']
date: '2025-07-22T22:32:31+08:00'
tags:
- Linux
- CentOS
- NodeJs

keywords:
- Linux
- CentOS
- NodeJs
- 安装
---

> 由于 CentOS7 为glibc 2.17，故而无法安装高版本
>
> 但可以获取非官方编译包


```bash
# 下载并解压非官方编译包（示例）
wget https://unofficial-builds.nodejs.org/download/release/v22.16.0/node-v22.16.0-linux-x64-glibc-217.tar.gz
tar -xzf node-v22.16.0-linux-x64-glibc-217.tar.gz -C /usr/local/
ln -s /usr/local/node-v22.16.0-linux-x64/bin/node /usr/bin/node
ln -s /usr/local/node-v22.16.0-linux-x64/bin/npm /usr/bin/npm
```