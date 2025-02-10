# openvpn
## 使用helm安装openvpn
[参考文档](https://artifacthub.io/packages/helm/devtron/openvpn?modal=install)
### 服务端配置
1.添加openvpn仓库
```
helm repo add devtron http://helm.devtron.ai/
```
2.下载安装包,自定义选择版本
```
helm pull devtron/openvpn --version 4.2.5
# 解压安装包
tar -zxvf openvpn-4.2.5.tgz
```
3.修改配置文件 values.yml
```
image: # 第15行
  repository: jfelten/openvpn-docker
  tag: 1.1.0
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
ipForwardInitContainer: true	#必须开启路由转发   第41行
persistence:   # 第56行
  enabled: true  # 自己选择是否开启
# existingClaim: efs-openvpn-dev-pvc  # 有可用的pvc 直击使用
  storageClass: "nfs-client"	# 使用的是nfs动态存储卷
  accessMode: ReadWriteOnce
  size: 50Mi
openvpn: # 第76行 下面没有就添加
  OVPN_K8S_POD_NETWORK: "10.244.0.0"	#k8s pod地址
  OVPN_K8S_POD_SUBNET: "255.255.0.0"
  OVPN_K8S_SVC_NETWORK: "10.96.0.0"	#k8s svc地址
  OVPN_K8S_SVC_SUBNET: "255.255.0.0"
```
磁盘模式,只能二选一不用的请注释
* existingClaim  有可用的pvc 直击使用
* storageClass 使用nfs动态存储卷 自动创建
1. 运行起来
```
helm install openvpn -n openvpn -create--namespace .
```
2. 获取密钥文件
**注意：运行脚本必须等待服务全部启动**
编写获取密钥的脚本
```
#!/bin/bash

if [ $# -ne 3 ]
then
  echo "Usage: $0 <CLIENT_KEY_NAME> <NAMESPACE> <HELM_RELEASE>"
  exit
fi

KEY_NAME=$1
NAMESPACE=$2
HELM_RELEASE=$3
POD_NAME=$(kubectl get pods -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_NAME=$(kubectl get svc -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_IP=$(kubectl get svc -n "$NAMESPACE" "$SERVICE_NAME" -o go-template='{{range $k, $v := (index .status.loadBalancer.ingress 0)}}{{$v}}{{end}}')
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" /etc/openvpn/setup/newClientCert.sh "$KEY_NAME" "$SERVICE_IP"
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" cat "/etc/openvpn/certs/pki/$KEY_NAME.ovpn" > "$KEY_NAME.ovpn"
```
执行脚本，会报错请无视
```
# 第一个参数是密钥名，第二个是namespace 第三个是软件的名称 也是openvpn
./gen-client-key.sh test-client openvpn openvpn
```
3. 修改以.ovpn结尾的密钥文件
删除第10行到第17行，添加以下内容。
```
remote {ip} {port} tcp
```
* ip: 添加可访问的ip
* port：开发的端口


