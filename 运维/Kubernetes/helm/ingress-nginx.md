# ingress-nginx 安装
参考文章
1.  [ingress-nginx官方文档](https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/)
2.  [ingress-nginx安装](https://www.cnblogs.com/tangxuliang/p/16922807.html)
## helm 增加仓库地址
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
## 下载对应chart压缩包
```bash
helm pull ingress-nginx/ingress-nginx
```
## 修改配置文件
```bash
# 解压chart压缩包
tar -zxvf ingress-nginx-x.x.x.tgz
```
### 有LoadBalance IP
1.  修改66行，修改 dnsPolicy 的值为 ClusterFirstWithHostNet
```yaml

    # -- Optionally change this to ClusterFirstWithHostNet in case you have 'hostNetwork: true'.

    # By default, while using host network, name resolution uses the host's DNS. If you wish nginx-controller

    # to keep resolving names inside the k8s network, use ClusterFirstWithHostNet.

    dnsPolicy: ClusterFirstWithHostNet
```
2.  修改89行 修改 hostNetwork 的值为 true
```yaml

    # -- Required for use with CNI based kubernetes installations (such as ones set up by kubeadm),

    # since CNI and hostport don't mix yet. Can be deprecated once <https://github.com/kubernetes/kubernetes/issues/23920>

    # is merged

    hostNetwork: true
```
3.  修改194行 修改 kind 类型为 DaemonSet
```yaml
    # -- Use a `DaemonSet` or `Deployment`

    kind: DaemonSet
```
4.  修改controller.service.type的业务类型
```yaml
# 根据自身业务需求修改成 LoadBalancer，NodePort，ClusterIP
# 使用NodePort，端口可以自定义选择
	service
	    type: LoadBalancer
```

5. 修改292行文件，在kubernetes.io/os: linux 后面添加一行 ingress: true
```yaml
  nodeSelector:
    kubernetes.io/os: linux
    ingress: "true"
```
### 无LoadBalance IP

## 使用helm启动
```
# 对从节点添加标签 ,ingress镜像将自动安装
kubectl label node node5 ingress=true

helm install ingress-nginx -n ingress-nginx --create-namespace .
```
## 使用ingress
```yaml
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