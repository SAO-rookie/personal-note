# ingress-nginx 安装
参考文章
1.  [ingress-nginx官方文档](https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/)
2.  [ingress-nginx安装](https://www.cnblogs.com/tangxuliang/p/16922807.html)


## helm 增加仓库地址
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
## 下载对应chart压缩包
```
helm pull ingress-nginx/ingress-nginx
```
## 修改配置文件
```
# 解压chart压缩包
tar -zxvf ingress-nginx-x.x.x.tgz
```
1. 修改第22行和第23行镜像地址**registry.cn-hangzhou.aliyuncs.com\google\_containers\nginx-ingress-controller\:v1.5.1** 注释27、28行
```

    controller:
    name: controller
    image:
    ## Keep false as default for now!
    chroot: false
    registry: registry.cn-hangzhou.aliyuncs.com # 修改为国内镜像地址
    image: google_containers\nginx-ingress-controller # 修改为国内镜像地址
    ## for backwards compatibility consider setting the full image url via the repository value below
    ## use *either* current default registry/image or repository format or installing chart by providing the values.yaml will fail
    ## repository:
    tag: "v1.5.1"
    #digest: sha256:4ba73c697770664c1e00e9f968de14e08f606ff961c76e5d7033a4a9c593c629 # 注释
    #digestChroot: sha256\:c1c091b88a6c936a83bd7b098662760a87868d12452529bad0d178fb36147345 # 注释
    pullPolicy: IfNotPresent
    # www-data -> uid 101
    runAsUser: 101
    allowPrivilegeEscalation: true
```
2.  修改66行，修改 dnsPolicy 的值为 ClusterFirstWithHostNet
```

    # -- Optionally change this to ClusterFirstWithHostNet in case you have 'hostNetwork: true'.

    # By default, while using host network, name resolution uses the host's DNS. If you wish nginx-controller

    # to keep resolving names inside the k8s network, use ClusterFirstWithHostNet.

    dnsPolicy: ClusterFirstWithHostNet
```
3.  修改89行 修改 hostNetwork 的值为 true
```

    # -- Required for use with CNI based kubernetes installations (such as ones set up by kubeadm),

    # since CNI and hostport don't mix yet. Can be deprecated once <https://github.com/kubernetes/kubernetes/issues/23920>

    # is merged

    hostNetwork: true
```
4.  修改194行 修改 kind 类型为 DaemonSet
```

    # -- Use a `DaemonSet` or `Deployment`

    kind: DaemonSet
```
5.  修改509行 修改 service 类型为 ClusterIP
```
type: ClusterIP
```
6. 修改660行和661行还有665行，修改 kube-webhook-certgen 的镜像地址为国内仓库 **registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen\:v20220916-gd32f8c343**

```
    patch:
      enabled: true
      image:
        registry: registry.cn-hangzhou.aliyuncs.com
        image: google_containers/kube-webhook-certgen
        ## for backwards compatibility consider setting the full image url via the repository value below
        ## use *either* current default registry/image or repository format or installing chart by providing the values.yaml will fail
        ## repository:
        tag: v1.5.1
        #digest: sha256:39c5b2e3310dc4264d638ad28d9d1d96c4cbb2b2dcfb52368fe4e3c63f61e10f
        pullPolicy: IfNotPresent

```
7. 修改292行文件，在kubernetes.io/os: linux 后面添加一行 ingress: true
```
  nodeSelector:
    kubernetes.io/os: linux
    ingress: true
```
## 使用helm启动
```
# 对从节点添加标签 ,ingress镜像将自动安装
kubectl label node node5 ingress=true

helm install ingress-nginx -n ingress-nginx --create-namespace .
```
## 使用ingress
```
# 编写配置文件
cat > /opt/test-ingress.yaml << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: ingress-myserviceb
spec:
rules:
- host: myserviceb.foo.org
  http:
	paths:
	  - backend:
		  service:
			name: myserviceb
			port:
			  number: 80
		path: /
		pathType: Prefix
EOF
# 启动
kubectl apply -f /opt/test-ingress.yaml
```