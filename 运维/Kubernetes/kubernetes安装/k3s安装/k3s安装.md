参考文档：[k3s文档一](https://docs.k3s.io/zh/quick-start),[K3s - 轻量级 Kubernetes | Rancher文档](https://docs.rancher.cn/docs/k3s/_index)

# 初始化服务器
**注意：系统是debain12**

### 设置主机名和解析
```bash
hostnamectl set-hostname <主机命>
# 配置 host文件
172.16.0.1 k8s.master.org
172.16.0.1 k8s.slave.one.org
172.16.0.1 k8s.slave.two.org
```

### 单节点配置内核参数
在 `/etc/sysctl.d/k3s-optimization.conf` 中添加：
```conf
# 文件系统优化
fs.inotify.max_user_instances = 16384    # 使用最大值
fs.inotify.max_user_watches = 1048576
fs.inotify.max_queued_events = 65536
fs.file-max = 4194304
fs.nr_open = 2097152

# 网络连接优化
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.core.netdev_max_backlog = 65536
net.ipv4.tcp_fastopen = 3

# 内存管理优化
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.overcommit_memory = 1
vm.overcommit_ratio = 90
vm.vfs_cache_pressure = 50

# 容器专用
user.max_cgroup_namespaces = 16384
user.max_user_namespaces = 16384
kernel.pid_max = 4194304
```
使用命令修改
```bash
sysctl -p /etc/sysctl.d/k3s-optimization.conf
```
### 更换dns

```bash
echo "nameserver 114.114.114.114" | tee /etc/resolv.conf
```
### 时间同步

```bash
apt install ntpdate -y
ntpdate cn.pool.ntp.org 
ls -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
```
### 配置iptables
```
apt install iptables
```

### 配置镜像仓库地址文件
**文档地址: [私有镜像仓库配置参考 | Rancher文档](https://docs.rancher.cn/docs/k3s/installation/private-registry/_index)**
# 单节点创建 
```bash
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

# 多节点创建
**注意：不建议使用默认的网络插件**
[快速创建单节点k3s | Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/k3s/quickstart)