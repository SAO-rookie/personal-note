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

### 无LoadBalance IP
**方法一：使用主机网络**
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
controller:
    kind: DaemonSet
```
4.  修改controller.service.type的业务类型
```yaml
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
**方法二：使用NodePort服务类型**
1. 修改controller.service.type的业务类型
```yaml
controller:
	service:
	    type: NodePort
	    nodePorts:
		    http: 30080
		    https: 30443
```
2. 修改 controller.kind 类型为 DaemonSet
```yaml
controller:
    kind: DaemonSet
```
3. 修改controller.nodeSelector，在kubernetes.io/os: linux 后面添加一行 ingress: true
```yaml
controller:
  nodeSelector:
    kubernetes.io/os: linux
    ingress: "true"
 ```  
 4. 需要对外提供服务的服务器，都要配置 iptables 转发规则​
```bash
#如果安装成功就别运行了
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sysctl -p
#配置iptables
# HTTP: 80 -> 30080
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 30080
# HTTPS: 443 -> 30443
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 30443
# 本地回环接口访问 (可选) 
iptables -t nat -A OUTPUT -o lo -p tcp --dport 80 -j REDIRECT --to-port 30080 
iptables -t nat -A OUTPUT -o lo -p tcp --dport 443 -j REDIRECT --to-port 30443
```
5. 保存 iptables 规则（永久生效）
```bash
apt-get install iptables-persistent -y 
netfilter-persistent save
```
### 有LoadBalance IP
 1. 修改controller.service.type 的业务类型
```yaml
controller:
	service:
	    type: LoadBalancer
```
2. 修改controller.replicaCount 的副本数
```yaml
controller:
	replicaCount: 3
```

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