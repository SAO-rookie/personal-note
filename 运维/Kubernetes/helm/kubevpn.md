# kubevpn安装
## 使用helm安装kubevpn的服务端
**注意：记得更换[镜像源](../../容器/docker/docker.md#docker 源)**
1. 添加kubevpn下载地址
```bash
helm repo add kubevpn https://raw.githubusercontent.com/kubenetworks/kubevpn/master/charts
```
2. 使用命令安装
```bash
helm install kubevpn kubevpn/kubevpn -n kubevpn --create-namespace
```
# 安装kubevpn客户端
1. 客户端下载地址
```
https://github.com/kubenetworks/kubevpn/releases
```
1. 