---
title: 'CentOS7最小安装后配置'
author: ['kyle']
date: '2025-07-22T22:48:13+08:00'
tags:
- Linux
- CentOS

keywords:
- Linux
- CentOS
---

# Linux笔记


## * centos被弃用后yum无法正常使用

下载CentOS 7的repo文件
```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
更新镜像源
```shell
# 清除缓存：
yum clean all
# 生成缓存：
yum makecache
```

## 一： Linux基本配置

### 1. 配置网卡，静态 IP

```shell
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
    
#修改
BOOTPROTO=static
ONBOOT=yes
#增加
IPADDR=192.168.197.129
GATEWAY=192.168.197.2
NETMASK=255.255.255.0
DNS1=144.144.144.144
DNS2=1.1.1.1
    
[root@localhost ~]# systemctl restart network.service
```

### 2. 更新yum,安装系统工具net-tools,常用工具，依赖关系
* mini
```shell
yum -y install net-tools vim-enhanced zip unzip curl telnet wget
```
* 个人常用

```shell
[root@localhost ~]# yum -y install net-tools telnet gcc gcc-c++ make autoconf wget curl curl-devel openssl openssl-devel vim-enhanced zip unzip ntpdate patch expect rsync
```

* 可选
```shell
[root@localhost ~]# yum -y update
[root@localhost ~]# yum -y install net-tools telnet gcc gcc-c++ make autoconf wget lrzsz libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel openssl openssl-devel libxslt-devel libevent-devel libtool libtool-ltdl bison gd gd-devel vim-enhanced pcre-devel zip unzip ntpdate sysstat patch bc expect rsync
```

### 3. 关闭SELINUX

```shell
[root@localhost ~]# vim /etc/sysconfig/selinux
#SELINUX=enforcing
SELINUX=disabled

[root@localhost ~]# setenforce 0
```

### 4. 设置时间时区

```shell
[root@localhost ~]# yum install ntp		#安装ntp服务的软件包
[root@localhost ~]# systemctl enable ntpd		#将ntp服务设置为缺省启动
[root@localhost ~]# vim /etc/sysconfig/ntpd		#修改启动参数，增加-g -x参数

OPTIONS="-g -x"

[root@localhost ~]# systemctl restart ntpd		#启动ntp服务
[root@localhost ~]# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime		#将系统时区改为上海时间 (亦即CST时区)
```

### 5. 关闭防火墙 (可选，要安全些的话还是老老实实的在需要的时候配防火墙吧)

```shell
[root@localhost ~]# systemctl stop firewalld.service		# 关闭 “系统防火墙” 
[root@localhost ~]# systemctl disable firewalld.service		# 关闭 “系统防火墙” 自启动
```

### 6. 修改主机名

```shell
[root@localhost ~]# hostnamectl set-hostname master
```

### 7. 配置ssh

```shell
[root@localhost ~]# vim /etc/ssh/ssh_config
        StrictHostKeyChecking no		#免输入yes进行known_hosts添加
        UserKnownHostsFile /dev/null		#可时时删除known_hosts文件免除known_hosts未更新导致的冲突
           
[root@localhost ~]# vim /etc/ssh/sshd_config
		#解决ssh链接慢问题
		UseDNS no		#UseDNS yes改为no, 如果没有或默认注释就添加UseDNS no
		GSSAPIAuthentication no		#GSSAPIAuthentication yes改为no, 如果没有或默认注释就添加GSSAPIAuthentication no
```

### 8.免密登录 (可选)

```shell
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
