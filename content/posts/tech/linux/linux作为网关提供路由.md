---
title: 'linux作为网关提供路由'
author: ['kyle']
date: '2025-07-22T23:02:49+08:00'
tags:
- Linux
- Network

keywords:
- Linux
- Network
---

## 1. 配置网卡
```
# 编辑网络配置文件
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens1
# 内容如下（示例IP，根据实际情况修改）：
DEVICE=ens1
BOOTPROTO=dhcp  # 或静态IP（需与物理网络同网段）
ONBOOT=yes

sudo vi /etc/sysconfig/network-scripts/ifcfg-ens2
# 内容如下：
DEVICE=ens2
BOOTPROTO=static
IPADDR=192.168.10.1
NETMASK=255.255.255.0
ONBOOT=yes
```

## 2. 启用IP转发
```
sudo vi /etc/sysctl.conf
# 取消注释或添加：
net.ipv4.ip_forward = 1
# 生效配置
sudo sysctl -p
```

## 3. 配置防火墙/NAT
```
# 使用firewalld配置
sudo firewall-cmd --permanent --zone=public --add-interface=ens1
sudo firewall-cmd --permanent --zone=internal --add-interface=ens2
sudo firewall-cmd --permanent --zone=public --add-masquerade
sudo firewall-cmd --reload
```

* 或使用iptables（如果未使用firewalld）
```
# 或使用iptables（如果未使用firewalld）
sudo iptables -t nat -A POSTROUTING -o ens1 -j MASQUERADE
sudo iptables -A FORWARD -i ens2 -j ACCEPT
# 保存规则（需安装iptables-services）
sudo service iptables save
sudo systemctl restart iptables
```

## DHCP服务
```
sudo yum install dhcp -y
```
* 配置dhcp
```
sudo vi /etc/dhcp/dhcpd.conf
```
```
# 定义DHCP全局选项（可选）
option domain-name "internal.net";
option domain-name-servers 8.8.8.8, 8.8.4.4;  # DNS服务器
default-lease-time 600;                        # 默认租约时间（秒）
max-lease-time 7200;                           # 最大租约时间

# 定义子网段
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.100 192.168.10.200;       # 分配的IP范围
    option routers 192.168.10.1;               # 网关（指向虚拟机A的ens2）
    option subnet-mask 255.255.255.0;
}
```

* 指定DHCP监听的接口
```
sudo vi /etc/sysconfig/dhcpd
```
```
DHCPDARGS="ens2"  # 仅监听内部网络接口
```

* 启动DHCP服务并设置开机自启
```
sudo systemctl start dhcpd
sudo systemctl enable dhcpd
```

* 防火墙放行DHCP流量
```
sudo firewall-cmd --permanent --add-service=dhcp
sudo firewall-cmd --reload
```