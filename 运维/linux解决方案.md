# 开启ipvs技术
- 安装 ipset 和 ipvsadm
```
# apt安装
apt install -y ipset ipvsadm
# yum安装
yum install -y ipset ipvsadm
```

- 查看是否开启 ipvs 技术
```
    [root@node1 ~]# lsmod|grep ip_vs
    ip_vs_sh               12688  0 
    ip_vs_wrr              12697  0 
    ip_vs_rr               12600  0 
    ip_vs                 145497  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
    nf_conntrack          133095  9 ip_vs,nf_nat,nf_nat_ipv4,nf_nat_ipv6,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_ipv4,nf_conntrack_ipv6
    libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
```
- 没有参数，则运行以下命令， 临时开启
```powershell
    modprobe ip_vs && modprobe ip_vs_rr && modprobe ip_vs_wrr && modprobe ip_vs_sh
```
- 永久开启
```powershell
    echo "ip_vs" >> /etc/modules-load.d/ipvs.conf
    echo "ip_vs_rr" >> /etc/modules-load.d/ipvs.conf
    echo "ip_vs_wrr" >> /etc/modules-load.d/ipvs.conf
    echo "ip_vs_sh" >> /etc/modules-load.d/ipvs.conf
```
# Debian系统下VIM不能使用鼠标复制
```powershell
# 在vim 配置文件添加以下信息接口 文件地址/etc/vim/vimrc
let skip_defaults_vim = 1
if has('mouse')
        set mouse-=a
endif
```