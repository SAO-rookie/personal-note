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