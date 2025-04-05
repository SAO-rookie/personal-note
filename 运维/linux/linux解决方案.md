# 开启ipvs技术
- 安装 ipset 和 ipvsadm
```shell
# apt安装
apt install -y ipset ipvsadm
# yum安装
yum install -y ipset ipvsadm
```

- 查看是否开启 ipvs 技术
```shell
    [root@node1 ~]# lsmod|grep ip_vs
    ip_vs_sh               12688  0 
    ip_vs_wrr              12697  0 
    ip_vs_rr               12600  0 
    ip_vs                 145497  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
    nf_conntrack          133095  9 ip_vs,nf_nat,nf_nat_ipv4,nf_nat_ipv6,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_ipv4,nf_conntrack_ipv6
    libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
```
- 没有参数，则运行以下命令， 临时开启
```shell
    modprobe ip_vs && modprobe ip_vs_rr && modprobe ip_vs_wrr && modprobe ip_vs_sh
```
- 永久开启
```shell
    echo "ip_vs" >> /etc/modules-load.d/ipvs.conf
    echo "ip_vs_rr" >> /etc/modules-load.d/ipvs.conf
    echo "ip_vs_wrr" >> /etc/modules-load.d/ipvs.conf
    echo "ip_vs_sh" >> /etc/modules-load.d/ipvs.conf
```
# Debian系统下VIM不能使用鼠标复制
```shell
# 在vim 配置文件添加以下信息接口 文件地址/etc/vim/vimrc
let skip_defaults_vim = 1
if has('mouse')
        set mouse-=a
endif
```

# SSH 配置
## 配置密码登录
```bash
vim /etc/ssh/sshd_config
## 配置
PermitRootLogin yes #允许root登录
PasswordAuthentication yes # 设置是否使用口令验证。

PermitEmptyPasswords no #不允许空密码登录
```
## 配置密钥登录登录
```bash
vim /etc/ssh/sshd_config
## 配置
RSAAuthentication yes
PubkeyAuthentication yes
```
重启命令
```bash
systemctl restart sshd
```

# Debian双网卡配置静态ip
```bash
vim /etc/network/interfaces
# 添加静态ip
auto eth1
iface eth1 inet static
    address 192.168.2.100
    netmask 255.255.255.0
    gateway 192.168.2.1
    dns-nameservers 8.8.8.8 8.8.4.4
```
