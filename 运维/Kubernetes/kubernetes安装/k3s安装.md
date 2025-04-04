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
### 配置仓库地址文件
**文档地址: []()**