# linux安装Samba
# 服务端安装
### 安装Samba
```bash
# centos
yum install smaba
# debian
apt install samba
```
### 配置共享目录
```bash
mkdir -p /srv/samba/share
chmod -R o+rwx /srv/samba/share
```
### 配置 Samba 文件
- 用户访问
```bash
vim /etc/samba/smb.conf
##在[global]下添加
security = user  ##原来已经存在则不需要修改
##在最后添加下面的内容：
[smb]
comment = smb folder
browseable = yes
path = /smb
create mask = 0700
directory mask = 0700
valid users = smb
force user = smb
force group = smb
public = yes
available = yes
writable = yes
```
- 匿名访问
```bash
vim /etc/samba/smb.conf
##找到 security = user 修改为
security = user
map to guest = Bad User
##在最后添加下面的内容：
[share]
comment = share folder
browseable = yes
path = /share
public = ok
guest ok = yes
writable = yes

```
### 创建 Samba 用户
**注意：匿名访问不用配置**
为了能通过 SMB 访问，您需要为用户创建一个 Samba 用户。首先，你可以创建一个 Linux 用户：

```bash
adduser sambauser
```

然后，使用以下命令为该用户设置 Samba 密码：
```bash
smbpasswd -a sambauser 
smbpasswd -e sambauser
```
- -a: 添加新用户
- -e: 启用该用户
### 重启 Samba 服务
```
systemctl restart smbd
```
## 客户端挂载
### 安装 
```bash
# centos
yum install smaba
# debian
apt install samba
```