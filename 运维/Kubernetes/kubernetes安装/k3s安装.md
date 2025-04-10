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
### 配置镜像仓库地址文件
**文档地址: [私有镜像仓库配置参考 | Rancher文档](https://docs.rancher.cn/docs/k3s/installation/private-registry/_index)**
# 单节点创建 
```bash
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

# 多节点创建
**注意：不建议使用默认的网络插件**
[快速创建单节点k3s | Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/k3s/quickstart)