**注意：安装Kubernetes，默认您已经完成服务器初始化。如何没有请[初始化](服务器初始化.md)**
# 所有主机安装都安装kubelet kubeadm kubectl
[参考文献](https://developer.aliyun.com/mirror/kubernetes)
## 使用apt或apt-get 安装
###  配置k8s下载地址
```bash
apt-get update && apt-get install -y apt-transport-https
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
```



###  安装kubelet kubeadm kubectl
```bash
apt install -y kubelet kubeadm kubectl

systemctl enable kubelet && systemctl start kubelet
```
## 2.使用yum安装
### 配置k8s下载地址

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```


###  安装kubelet kubeadm kubectl

```bash
yum install -y kubelet kubeadm kubectl
# yum install -y kubelet-1.28.2-0 kubeadm-1.28.2-0 kubectl-1.28.2-0 下载指定版本
# 运行kubectl
systemctl enable kubelet && systemctl start kubelet
```
# 初始化k8s集群
## 1. 创建k8s主节点
```bash
kubeadm init --control-plane-endpoint="k8s.master.org" --kubernetes-version=1.24.5 --pod-network-cidr=10.244.0.0/16 --token-ttl=0  --upload-certs
```
**注意：--control-plane-endpoint是主机名称 ，也就是hostnamectl命名名字**
**注意：使用flannel网络插件时 网络命令为 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12**
**注意：使用calico网络插件时 网络命令为--pod-network-cidr=192.168.0.0/16 --service-cidr不需要**

**注意：更换运行时 添加--cri-socket=unix:///var/run/cri-dockerd.sock，后面参数更换**

| 运行环境                              | Col2                                         |
| --------------------------------- | ---------------------------------------------- |
| containerd                        | **unix:///var/run/containerd/containerd.sock** |
| CRI-O                             | **unix:///unix:///var/run/crio/crio.sock**     |
| Docker Engine (using cri-dockerd) | **unix:///var/run/cri-dockerd.sock**      |

**注意：控制台出现以下情况表示安装成功**
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/


You can now join any number of the control-plane node running the following command on each as root:
# 给其他服务器用于添加新的k8s主节点
  kubeadm join k8s.master.org:6443 --token bniqon.k14qk0j5947oefvg \
        --discovery-token-ca-cert-hash sha256:df6202efc0382a91d87e77c15101f9ccae12f0a0748315f129292ef8562752ee \
        --control-plane --certificate-key 7131a2ac937780fee006c3b938ddbb6145a7d4c4925d92afb4e739aa269d5fd1

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:
# 给其他服务器用于添加新的k8s工作节点
kubeadm join k8s.master.org:6443 --token bniqon.k14qk0j5947oefvg \
        --discovery-token-ca-cert-hash sha256:df6202efc0382a91d87e77c15101f9ccae12f0a0748315f129292ef8562752ee 
```
运行以下命令

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

## 2. 安装其他主节点 或者 工作节点
**注意：使用docker作为运行时 所有命令后面 必须加 --cri-socket=unix:///var/run/cri-dockerd.sock**
参考以下命令，*该命令必须是从第一个主节点控制台打印的*
* 需要将目标服务器设置为新的主节点
```bash
  kubeadm join k8s.master.org:6443 --token bniqon.k14qk0j5947oefvg \
        --discovery-token-ca-cert-hash sha256:df6202efc0382a91d87e77c15101f9ccae12f0a0748315f129292ef8562752ee \
        --control-plane --certificate-key 7131a2ac937780fee006c3b938ddbb6145a7d4c4925d92afb4e739aa269d5fd1 
```
*  需要将目标服务器设置为新的工作节点
参考以下命令，*该命令必须是从第一个主节点控制台打印的*
```
kubeadm join k8s.master.org:6443 --token bniqon.k14qk0j5947oefvg \
        --discovery-token-ca-cert-hash sha256:df6202efc0382a91d87e77c15101f9ccae12f0a0748315f129292ef8562752ee --cri-socket=unix:///var/run/cri-dockerd.sock
```

## 3. 在主节点安装网络插件
* flannel网络插件
[flannel下载地址](https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml)
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
自定义pod ip 网络端
```bash
# 修改配置文件kube-flannel.yml 内容
vim kube-flannel.yml
# 文件内容截取
net-conf.json: |
	{
		"Network": "10.244.0.0/16", # 把这里换成自己想要的ip段
		"EnableNFTables": false,
		"Backend": {
		"Type": "vxlan"
		}
	}

```
* calico网络插件

[calico网络插件安装](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)
自定义pod ip 网络端
```
# 修改配置文件kube-flannel.yml 内容
vim custom-resources.yaml
# 文件内容截取
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 192.168.0.0/16 # 把这里换成自己想要的ip段
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
```
## 4. 使用IPVS模式
* 修改kube-proxy配置文件参数 然后重启容器组
```
    kubectl edit configmap -n kube-system kube-proxy
    # 32 行
    iptables:
      masqueradeAll: true
    # 47 行
    mode: "ipvs" 
```
* 删除kube-system名称空间下载的pod，
```
kubectl get pod -n kube-system |grep kube-proxy |awk '{system("kubectl delete pod "$1" -n kube-system")}'
```
* 检验ipvs启动是否生效
```
kubectl logs kube-proxy-xxx -n kube-system
```
* 查看ipvs维护的ip列表,没有就自己安装
```
ipvsadm -ln
```
## 5. 主节点安装metrics-server
metrics-server配置文件的[下载地址](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)
下载完成时必须修改一点内容，因为镜像地址在国外无法下载
修改文件的140行 将其换成以下内容
```
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.2
```
**安装分为两个：测试环境，生产环境**
* 测试环境安装
```
  containers:
  - args:
    - --cert-dir=/tmp
    - --secure-port=4443
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --kubelet-use-node-status-port
    - --metric-resolution=15s
    - --kubelet-insecure-tls # 加上这一行即可
    image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.2
```
* 生产环境安装
    将第一个主节点的证书 复制到其他所有节点 位置相同
```
# 主节点证书位置
/etc/kubernetes/pki/front-proxy-ca.crt
```
然后运行配置文件 命令如下
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## 6. 安装可视化面板Kuboard
* 配置节点角色
```bash
kubectl label node node5 node-role.kubernetes.io/worker=worker
master节点若缺少角色标签，执行：
kubectl label node node1 node-role.kubernetes.io/master=master
```
* 然后参考[文档](https://kuboard.cn/install/v3/install-in-k8s.html)安装 
