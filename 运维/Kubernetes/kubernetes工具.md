# 安装命令行可视化工具 k9s
[官方文档](https://k9scli.io/topics/install/)
## 命令安装
- debian
```
# debian
wget https://github.com/derailed/k9s/releases/download/{版本号}/k9s_linux_amd64.deb
# 安装
apt install -y ./k9s_linux_amd64.deb
```
- centos
```
# centos
wget https://github.com/derailed/k9s/releases/download/{版本号}/k9s_linux_amd64.rpm
# 安装
yum install -y ./k9s_linux_amd64.deb
```
## 二进制安装
# krew的安装

```
# 安装git
yum install -y git

# 下载krew-linux_amd64.tar.gz
wget https://github.com/kubernetes-sigs/krew/releases/download/v0.4.3/krew-linux_amd64.tar.gz

# 解压下载krew-linux_amd64.tar.gz
tar -zxvf krew-linux_amd64.tar.gz

# 编写配置文件
cat >> ~/.bashrc <<EOF
export PATH="${PATH}:${HOME}/.krew/bin"
EOF

# 刷新配置
source ~/.bashrc

#注意在解压出来的目录执行
./krew-linux_amd64 install krew

#验证安装是否成功
kubectl krew list
```