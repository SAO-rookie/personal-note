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
1.  修改 controller.dnsPolicy 的值为 ClusterFirstWithHostNet
```yaml
controller:
    dnsPolicy: ClusterFirstWithHostNet
```
2.  修改 controller.hostNetwork 的值为 true
```yaml
controller:
    hostNetwork: true
```
3.  修改 controller.kind 类型为 DaemonSet
```yaml
    # -- Use a `DaemonSet` or `Deployment`
controller:
    kind: DaemonSet
```
4.  修改controller.service.type的业务类型
```yaml
# 根据自身业务需求修改成 LoadBalancer，NodePort，ClusterIP
# 使用NodePort，端口可以自定义选择
controller:
	service:
	    type: ClusterIP
```
 5. 修改controller.nodeSelector，在kubernetes.io/os: linux 后面添加一行 ingress: true
```yaml
controller:
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