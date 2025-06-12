# k8s本体
[官方文档](https://kubernetes.io/zh-cn/docs/home/)
## k8s修改dns配置
解决方案一：
当你感觉本地dns慢时，可以修改所有主节点的dns配置，然后重启**coredns**
解决方案二：
修改coredns的configmap配置
```
kubectl edit cm coredns -n kube-system
```

```
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        # 删除使用本机dns配置
        #forward . /etc/resolv.conf {
        #   max_concurrent 1000
        #}
        # 使用电信dns服务
        forward . 114.114.114.114 {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  annotations: {}
  labels: {}
  name: coredns
  namespace: kube-system
```
## k8s集群升级

k8s版本升级要一级一级的升 [文档](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)
## k8s添加新节点
* TOKEN过期后 重新生产TOKEN，先使用命令重新生成证书
```
kubeadm token create --print-join-command
```
* control-plane的加入
```
kubeadm init phase upload-certs --upload-certs

## 集群的其他master节点， 有 --control-plane
kubeadm join master.k8s.io:6443 --token ime4yx.8fb5jsv0smqkk0aq  \
   --discovery-token-ca-cert-hash 证书
   --control-plane  --certificate-key  主节点证书
  
## 集群的worker节点 ， 没有 --control-plane
kubeadm join master.k8s.io:6443 --token ime4yx.8fb5jsv0smqkk0aq  \
   --discovery-token-ca-cert-hash 证书
```
## 修改节点的pod 限制
* 最后一行添加 maxPods: 300 就可以 数量自己写
```
vim /var/lib/kubelet/config.yaml

apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
	enabled: false
  webhook:
	cacheTTL: 0s
	enabled: true
  x509:
	clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
	cacheAuthorizedTTL: 0s
	cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
	json:
	  infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
maxPods: 300 # 最后一行添加
```
* 然后运行以下命令
```
# 重新启动
systemctl restart kubelet
# 查看是否修改成功
kubectl describe node 节点名 |  grep -i "Capacity\|Allocatable" -A 6
```
## k8s重新刷新证书
[参考文档](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)
1. 查看证书是否失效
```
kubeadm certs check-expiration
```
2. 备份老证书
```
cp -r /etc/kubernetes /etc/kubernetes.old 
```
3. 更新证书
```
kubeadm certs renew all
```
4. 更新 ~/.kube/config 文件
```
mv config config.old
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
sudo chmod 644 $HOME/.kube/config
```
5. 重启相关应用（所有master都要执行）
- docker
```
docker ps |grep -E 'k8s_kube-apiserver|k8s_kube-controller-manager|k8s_kube-scheduler|k8s_etcd_etcd' | awk -F ' ' '{print $1}' |xargs docker restart
```
- containerd

```
crictl ps |grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler|etcd' | awk -F ' ' '{print $1}' |xargs crictl stop
```
# k8s的CRD配置
## k8s pod探针
[官网pod探针介绍](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations: {}
  labels:
    app: question-server
  name: question-server
spec:
  selector:
    matchLabels:
      app: question-server
  template:
    metadata:
      labels:
        app: question-server
    spec:
      containers:
        - image: 'harbor.changtech.cn/gutanzhang/question-server-test:v1.0.47'
          imagePullPolicy: IfNotPresent
          name: question-server-test
          livenessProbe: # 探测容器健康状态
              initialDelaySeconds: 20
              periodSeconds: 5
              httpGet:
                  port: 9090
                  path: /k8s/health
          readinessProbe: # 探测容器准备状态
              initialDelaySeconds: 30
              periodSeconds: 5
              httpGet:
                  port: 9090
                  path: /k8s/ready
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
```
## 使用service 配置公网ip地址
1. 使用以下配置文件创建一个service
```
apiVersion: v1
kind: Service
metadata:
  name: lz-minio
spec:
  ports:
    - protocol: TCP
      name: lz
      port: 80
      targetPort: 80
```
2. 使用以下配置创建一个Endpoints
```
apiVersion: v1
kind: Endpoints
metadata:
  name: lz-minio
subsets:
  - addresses:
      - ip: 10.10.10.1
    ports:
      - port: 30001
        name: lz
        protocol: TCP
```