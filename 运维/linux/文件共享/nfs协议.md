# Linux安装NFS系统
## NFS 安装
### apt安装nfs
* 安装 NFS 服务端
```
apt install nfs-kernel-server -y
```
* 安装 NFS 客户端
```
apt install nfs-common -y
```
### yum安装 NFS
* 安装NFS工具
```
yum install -y nfs-utils
```
##  服务端暴露目录
* 创建文件并且授权
```
# 创建文件夹
mkdir -p /nfs/data/

# 给文件授权
chmod 777 /nfs/data/
```
* nfs 暴露目录配置
```
# 暴露文件夹
echo "/nfs/data/ 172.16.0.0/24(rw,sync,no_subtree_check,insecure,no_root_squash)" > /etc/exports
# 使上述配置生效
exportfs -r
```
* 重启nfs服务
 apt安装的nfs重启
```
systemctl restart nfs-kernel-server
systemctl enable nfs-kernel-server
```
yum安装的nfs重启
```
# 启动rpcbind服务
systemctl enable rpcbind --now
# 启动nfs服务器
systemctl enable nfs-server --now
```
* 检查nfs
```
# 运行exportfs命令， 出现一些以下 表示安装成功
[root@k8s~]# exportfs
/nfs/data/   <world>
```
# 客户端挂载
* 临时挂载
```
mount <NFS服务器IP>:/{暴露目录} /{本地目录}
```
* 永久挂载
```
vim /etc/fstab
# 添加以下代码
<NFS服务器IP>:/{暴露目录} /{本地目录} nfs defaults 0 0
```