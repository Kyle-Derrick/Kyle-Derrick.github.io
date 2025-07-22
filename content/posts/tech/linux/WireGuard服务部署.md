---
title: 'WireGuard服务部署'
author: ['kyle']
date: '2025-07-22T22:58:51+08:00'
tags:
- Linux
- WireGuard
- VPN
- Network

keywords:
- Linux
- WireGuard
- VPN
- Network
---

### **一、虚拟机A（Ubuntu 20）配置**
#### **1. 安装WireGuard**
```bash
sudo apt update
sudo apt install wireguard resolvconf  # 安装WireGuard和DNS工具（如需）
```

#### **2. 生成服务器密钥对**
```bash
# 生成私钥和公钥
sudo umask 077  # 确保密钥文件权限安全
sudo wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
```

#### **3. 配置WireGuard服务端**
创建配置文件 `/etc/wireguard/wg0.conf`：
```bash
sudo nano /etc/wireguard/wg0.conf
```
内容如下（根据需求修改）：
```ini
[Interface]
Address = 10.10.20.1/24  # 服务器虚拟IP（WireGuard内网段）
SaveConfig = true
ListenPort = 51820        # 监听端口（确保防火墙开放）
PrivateKey = <服务器私钥>  # 从 /etc/wireguard/server_private.key 复制
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE


# 客户端配置（示例：允许两个客户端）
[Peer]
# 客户端1
PublicKey = <客户端1公钥>
AllowedIPs = 10.10.20.2/32  # 分配给客户端的虚拟IP

[Peer]
# 客户端2
PublicKey = <客户端2公钥>
AllowedIPs = 10.10.20.3/32
```

#### **4. 启用IP转发和NAT**
编辑 `/etc/sysctl.conf`：
```ini
net.ipv4.ip_forward=1
```
加载配置：
```bash
sudo sysctl -p
```

#### **5. 启动WireGuard服务**
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

#### **6. 开放防火墙端口**
```bash
sudo ufw allow 51820/udp  # 允许WireGuard端口
sudo ufw reload
```

---

### **二、客户端配置（以Windows和Android为例）**
#### **1. 生成客户端密钥对**
在客户端机器上生成密钥（以客户端1为例）：
```bash
# Linux/macOS/WSL：
wg genkey | tee client1_private.key | wg pubkey > client1_public.key

# Windows（需安装WireGuard客户端后，在界面中生成密钥）
```

#### **2. 客户端配置文件（示例：client1.conf）**
```ini
[Interface]
PrivateKey = <客户端1私钥>
Address = 10.10.20.2/24  # 客户端虚拟IP（需与服务端AllowedIPs一致）
DNS = 8.8.8.8            # 可选DNS

[Peer]
PublicKey = <服务器公钥>  # 从 /etc/wireguard/server_public.key 复制
Endpoint = <服务器公网IP>:51820  # 虚拟机A的ens1网卡IP
AllowedIPs = 0.0.0.0/0    # 将所有流量路由到VPN（或指定子网）
PersistentKeepalive = 25   # 保持连接
```

#### **3. 将客户端公钥添加到服务端**
在虚拟机A上编辑 `/etc/wireguard/wg0.conf`，添加客户端的`[Peer]`段：
```ini
[Peer]
PublicKey = <客户端1公钥>
AllowedIPs = 10.10.20.2/32
```

重启WireGuard服务：
```bash
sudo systemctl restart wg-quick@wg0
```

#### **4. 客户端连接**
- **Windows**：
  1. 安装 [WireGuard客户端](https://www.wireguard.com/install/)。
  2. 导入 `client1.conf` 文件。
  3. 点击“激活”连接。

- **Android/iOS**：
  1. 安装 WireGuard 客户端。
  2. 扫描配置文件二维码（或手动导入）。

---

### **三、验证和排错**
#### **1. 检查服务端状态**
```bash
sudo wg show  # 查看连接的对等节点和流量统计
```

#### **2. 测试连通性**
- 客户端执行：
  ```bash
  ping 10.10.20.1       # 测试VPN内网连通性
  ping 8.8.8.8          # 测试外网（需服务端NAT配置正确）
  ping <物理机IP>       # 测试访问物理机
  ```

#### **3. 常见问题**
- **无法连接**：
  - 检查服务端防火墙是否开放UDP 51820。
  - 确保客户端和服务端的公钥/私钥配对正确。
  - 检查服务端NAT规则（`iptables -t nat -L`）。
  
- **无网络流量**：
  - 确认服务端已启用IP转发（`sysctl net.ipv4.ip_forward`）。
  - 检查客户端`AllowedIPs`是否为 `0.0.0.0/0`（全局流量）或指定子网。

---

### **四、扩展配置（可选）**
#### **1. 动态IP客户端**
在服务端配置中，允许客户端IP动态分配：
```ini
[Peer]
PublicKey = <客户端公钥>
AllowedIPs = 10.10.20.2/32  # 固定IP
# 或
AllowedIPs = 10.10.20.0/24  # 允许客户端使用整个子网
```

#### **2. 多子网路由**
如果客户端需要访问其他子网（如虚拟机B的网段 `192.168.100.0/24`）：
```bash
# 在服务端添加路由
sudo ip route add 192.168.100.0/24 dev wg0

# 在客户端配置中指定AllowedIPs：
AllowedIPs = 0.0.0.0/0, 192.168.100.0/24
```

---

### **五、安全性建议**
- **限制客户端IP范围**：避免使用 `AllowedIPs = 0.0.0.0/0`，按需分配。
- **定期更换密钥**：删除旧密钥并重新生成。
- **启用防火墙白名单**：仅允许已知IP连接服务端。

---

通过以上步骤，WireGuard VPN服务将部署完成，客户端可通过虚拟机A访问物理机和其他网络资源。