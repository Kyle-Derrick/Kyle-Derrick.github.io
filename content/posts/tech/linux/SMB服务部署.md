---
title: 'SMB服务部署'
author: ['kyle']
date: '2025-07-22T22:56:40+08:00'
tags:
- Linux
- SMB

keywords:
- Linux
- SMB
- 安装
---

### **一、安装Samba服务**
1. **安装软件包**  
   使用yum安装Samba核心组件：
   ```bash
   yum install samba samba-client samba-common -y
   ```
   *说明：`samba`为服务端程序，`samba-client`提供客户端工具，`samba-common`包含通用文件。* 

2. **启动服务并设置开机自启**  
   ```bash
   systemctl start smb nmb   # 启动Samba服务
   systemctl enable smb nmb  # 设置开机自启
   ```

---

### **二、配置Samba共享**
1. **创建共享目录并设置权限**  
   ```bash
   mkdir -p /srv/shared      # 创建共享目录
   chmod -R 777 /srv/shared  # 设置目录权限（按需调整）
   chown -R nobody:nobody /srv/shared  # 指定所有者（匿名访问时使用）
   ```

2. **编辑主配置文件 `/etc/samba/smb.conf`**  
   备份原文件后，添加以下内容：
   ```ini
   [global]
      workgroup = WORKGROUP      # 工作组名（与Windows一致）
      security = user            # 安全模式：user（需密码）或share（匿名）
      passdb backend = tdbsam    # 用户数据库类型
      map to guest = Bad User    # 允许匿名访问（仅share模式需要）

   [shared]
      comment = Public Share     # 共享描述
      path = /srv/shared         # 共享目录路径
      browseable = yes           # 允许浏览
      writable = yes             # 允许写入
      public = yes               # 允许匿名访问（share模式）
      valid users = user1, @group1  # 仅允许指定用户或组访问（user模式）
   ```
   *说明：选择`security = user`时需配置用户密码，`security = share`则允许匿名访问。* 

3. **配置用户验证（user模式）**  
   - 创建系统用户并设置Samba密码：
     ```bash
     useradd user1               # 创建用户
     smbpasswd -a user1          # 设置Samba密码
     ```
   - 查看Samba用户列表：
     ```bash
     pdbedit -L
     ```

---

### **三、防火墙与SELinux设置**
1. **防火墙配置**  
   - **开放SMB端口**：
     ```bash
     firewall-cmd --permanent --add-service=samba
     firewall-cmd --reload
     ```
   - **或关闭防火墙（测试环境）**：
     ```bash
     systemctl stop firewalld && systemctl disable firewalld
     ```

2. **禁用SELinux**  
   - 临时关闭：
     ```bash
     setenforce 0
     ```
   - 永久关闭（编辑`/etc/selinux/config`）：
     ```ini
     SELINUX=disabled
     ```
   *说明：SELinux可能阻止共享访问，建议测试时关闭。* 

---

### **四、测试与访问**
1. **Windows客户端访问**  
   - 打开资源管理器，输入地址：`\\<服务器IP>\shared`。
   - 输入用户名和密码（user模式）或直接访问（share模式）。

2. **Linux客户端挂载共享**  
   - 安装客户端工具：
     ```bash
     yum install cifs-utils -y
     ```
   - 挂载共享目录：
     ```bash
     mount -t cifs //<服务器IP>/shared /mnt -o username=user1,password=123
     ```

---

### **五、高级配置**
1. **多用户权限控制**  
   - 在`smb.conf`中通过`valid users`和`write list`指定不同用户的读写权限。例如：
     ```ini
     [department]
        path = /srv/department
        valid users = @group1
        writable = yes
        write list = user1
     ```
   *说明：使用`@group1`允许组访问，`write list`限制仅user1可写。* 

2. **禁用旧协议（SMB1）**  
   - 编辑`/etc/modprobe.d/local.conf`：
     ```bash
     echo "options cifs disable_legacy_dialects=Y" >> /etc/modprobe.d/local.conf
     ```
   *说明：提升安全性，需内核版本≥4.18。* 

---

### **常见问题**
- **无法写入文件**：检查目录权限（`chmod 777`）及SELinux状态。
- **匿名访问失败**：确认`security = share`，并设置`public = yes`。
- **端口被阻止**：确保防火墙开放139/TCP、445/TCP、137-138/UDP。

通过以上步骤，您可以在CentOS系统上快速部署Samba服务，并根据需求灵活配置共享权限与安全策略。