# linux安装Samba
## 安装Samba
```bash
# centos
yum install smaba
# debian
apt install samba
```
## 配置共享目录
```bash
mkdir -p /srv/samba/share
chmod 777 /srv/samba/share
```
## 配置 Samba 文件
```bash
vim /etc/samba/smb.conf

# 在文件的末尾添加一个共享配置块：
[Share]
   path = /srv/samba/share
   available = yes
   valid users = @samba
   read only = no
   browsable = yes
   public = yes
   writable = yes
```
- `path`: 指定共享的文件夹路径。
- `valid users`: 设置允许访问的用户。
- `read only`: 设置为 `no` 允许写入。
- `public`: 设置为 `yes` 表示任何人都可以访问。
## 创建 Samba 用户
为了能通过 SMB 访问，您需要为用户创建一个 Samba 用户。首先，你可以创建一个 Linux 用户：

```bash
adduser sambauser
```

然后，使用以下命令为该用户设置 Samba 密码：
```bash
smbpasswd -a sambauser &&  smbpasswd -e sambauser
```
- -a: 添加新用户
- -e: 启用该用户
## 重启 Samba 服务
```
systemctl restart smbd
```