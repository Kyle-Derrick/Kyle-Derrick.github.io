---
title: 'IPsec/IKEv2服务部署'
author: ['kyle']
date: '2025-07-22T23:00:17+08:00'
tags:
- Linux
- IPsec
- VPN
- Network

keywords:
- Linux
- IPsec
- IKEv2
- Network
---

> https://www.cnblogs.com/aresxin/p/17864829.html
> 
## 开启内核转发
```
# tail -6 /etc/sysctl.conf 

net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# sysctl -p
```


## ipsec.conf配置
```
# cat /etc/ipsec.conf
config setup
        charondebug="ike 1, knl 1, cfg 0, net 1"
        strictcrlpolicy=no
        uniqueids=yes  # 如果同一个用户在不同的设备上重复登录,yes 断开旧连接,创建新连接;no 保持旧连接,并发送通知; never 同 no, 但不发送通知.
        cachecrls=no 
conn ipsec-ikev2-vpn   #XAUTH认证需要
      auto=add			# 当服务启动时, 应该如何处理这个连接项,add 添加到连接表中
      compress=no		# 是否启用压缩, yes 表示如果支持压缩会启用
      type=tunnel	
      keyexchange=ikev2
      fragmentation=yes
      forceencaps=yes
      dpdaction=clear	# 当意外断开后尝试的操作
      dpddelay=300s		# dpd时间间隔
      rekey=no			# 不自动重置密钥
      left=%any			# 服务器端标识，可以是魔术字 %any，表示从本地ip地址表中取
      leftid=123.13.12.1		# 服务器端ID标识,这里为你的公网ip，或者@你的域名
      leftcert=server.cert.pem	# 服务器端证书
      leftsendcert=always		# 是否发送服务器证书到客户端
      leftsubnet=0.0.0.0/0
      right=%any		# 客户端标识，%any表示任意
      rightid=%any
      rightauth=eap-mschapv2			#KEv2 EAP(Username/Password)
      rightsourceip=192.168.137.0/24 	 # 客户端IP地址分配范围
      rightdns=223.5.5.5,223.6.6.6		# DNS
      rightsendcert=never				# 客户端不发送证书
      eap_identity=%identity			# 指定客户端eap id
      ike=chacha20poly1305-sha512-curve25519-prfsha512,aes256gcm16-sha384-prfsha384-ecp384,aes256-sha1-modp1024,aes128-sha1-modp1024,3des-sha1-modp1024!			# 密钥交换协议加密算法列表，可以包括多个算法和协议。
      esp=chacha20poly1305-sha512,aes256gcm16-ecp384,aes256-sha256,aes256-sha1,3des-sha1!	# 数据传输协议加密算法列表,对于IKEv2，可以在包含相同类型的多个算法（由-分隔）
conn xauth_psk    #PSK认证需要
      keyexchange=ikev2		# 使用 IKEv2
      left=%defaultroute
      leftauth=psk			# 服务器校验方式
      leftsubnet=0.0.0.0/0
      right=%any
      rightauth=psk
      rightsourceip=192.168.137.0/24
      auto=add
conn IKEv2-pubkey	#RSA认证
    # 服务器端根证书 DN 名称
    leftca="CN=ares"
    # 服务器证书，可以是 PEM 或 DER 格式
    leftcert=server.cert.pem
    # 不指定客户端证书路径
    # rightcert = <path>
    # 指定服务器证书的公钥
    leftsigkey=server.pub.pem
    # rightsigkey = <raw public key> | <path to public key>
    # 是否发送服务器证书到客户端
    leftsendcert=always
    # 客户端不发送证书
    rightsendcert=never
    # 服务端认证方法，使用证书
    leftauth=pubkey
    # 客户端认证使用 EAP 扩展认证，貌似 eap-mschapv2 比较通用
    rightauth=eap-mschapv2
    # 服务端 ID，可以任意指定，默认为服务器证书的 subject，还可以是魔术字 %any，表示什么都行
    leftid=%any
    # 客户端 id，任意
    rightid=%any
```