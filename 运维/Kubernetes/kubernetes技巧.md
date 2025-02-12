# k8s本体
## 切换k8s 命名空间
```
kubectl config set-context --current --namespace=devops
```
## 配置节点角色
```
kubectl label node node5 node-role.kubernetes.io/worker=worker
master节点若缺少角色标签，执行：
kubectl label node node1 node-role.kubernetes.io/master=master
```
## 主节点参与调度
```
kubectl taint nodes <节点> node-role.kubernetes.io/master:NoSchedule-
kubectl taint nodes <节点> node-role.kubernetes.io/control-plane:NoSchedule-
```
# kubernetes创建新用户
- 创建create-user.sh文件
```
#!/bin/bash

# 创建 Kubernetes 用户脚本
# 需要: openssl, kubectl, jq 工具

set -e

# 配置变量
USERNAME="dev-user"
NAMESPACE="development"
CLUSTER_NAME="my-cluster"
API_SERVER="https://api-server:6443" # 替换为你的API服务器地址
CA_CERT="/etc/kubernetes/pki/ca.crt" # CA证书路径
CA_KEY="/etc/kubernetes/pki/ca.key" # CA私钥路径
VALID_DAYS=365 # 证书有效期

# 创建私钥
openssl genrsa -out ${USERNAME}.key 2048

# 创建证书签名请求 (CSR)
openssl req -new -key ${USERNAME}.key \
  -subj "/CN=${USERNAME}/O=developers" \
  -out ${USERNAME}.csr

# Base64 编码 CSR
CSR_BASE64=$(cat ${USERNAME}.csr | base64 | tr -d "\n")

# 创建 Kubernetes CSR 资源
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ${USERNAME}-csr
spec:
  signerName: kubernetes.io/kube-apiserver-client
  groups:
  - system:authenticated
  request: ${CSR_BASE64}
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF

# 批准 CSR
kubectl certificate approve ${USERNAME}-csr

# 等待证书签发
sleep 5 # 等待时间可能需要根据实际情况调整

# 提取已签名证书
kubectl get csr ${USERNAME}-csr -o jsonpath='{.status.certificate}' | base64 --decode > ${USERNAME}.crt

# 创建 kubeconfig 文件
kubectl config set-cluster ${CLUSTER_NAME} \
  --certificate-authority=${CA_CERT} \
  --embed-certs=true \
  --server=${API_SERVER} \
  --kubeconfig=${USERNAME}.kubeconfig

kubectl config set-credentials ${USERNAME} \
  --client-certificate=${USERNAME}.crt \
  --client-key=${USERNAME}.key \
  --embed-certs=true \
  --kubeconfig=${USERNAME}.kubeconfig

kubectl config set-context ${USERNAME}-context \
  --cluster=${CLUSTER_NAME} \
  --namespace=${NAMESPACE} \
  --user=${USERNAME} \
  --kubeconfig=${USERNAME}.kubeconfig

kubectl config use-context ${USERNAME}-context \
  --kubeconfig=${USERNAME}.kubeconfig

echo "用户创建完成！"
echo "Kubeconfig 文件: ${USERNAME}.kubeconfig"
echo "私钥文件: ${USERNAME}.key"
echo "证书文件: ${USERNAME}.crt"
```
- 添加执行权限
```
chmod +x create-user.sh
```
- 需要具有集群管理员权限执行
- 根据实际情况修改以下参数：
	- `CLUSTER_NAME`
	- `API_SERVER`
	- `CA_CERT` 和 `CA_KEY` kubernetes证书路径
	- `USERNAME`
	- `NAMESPACE`
-  创建 Role 和 RoleBinding 或 ClusterRole  和 ClusterRoleBinding ，以下是 Role 和 RoleBinding的例子
```
# 创建 Role 和 RoleBinding
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: ${NAMESPACE}
  name: ${USERNAME}-role
rules:
- apiGroups: ["", "apps", "networking.k8s.io"]
  resources: ["pods", "deployments", "services", "ingresses"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: ${NAMESPACE}
  name: ${USERNAME}-role-binding
subjects:
- kind: User
  name: ${USERNAME}
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: ${USERNAME}-role
  apiGroup: rbac.authorization.k8s.io
EOF
```
# k8s的CRI配置
## deployment 等运行时 修改hosts文件
```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations: {}
  labels:
    app: file-server
  name: file-server
  namespace: public-space
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: file-server
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: file-server
    spec:
      containers:
        - image: 'public-server/file-server-test:v1'
          imagePullPolicy: IfNotPresent
          name: public-service-web-file-test
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      hostAliases: #  根据自身需求 修改IP和域名
        - hostnames:
            - minio.int
          ip: 172.16.0.240
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```
## 远程调试Java
- 修改对应运行时的yaml文件
```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations: {}
  labels:
    app: file-server
  name: file-server
  namespace: public-space
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: file-server
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: file-server
    spec:
      containers:
        - env: # 添加环境参数 
           - name: JAVA_TOOL_OPTIONS
             value: -agentlib:jdwp=transport=dt_socket,server=y,address=8848,suspend=y # 端口自定义
          image: 'public-server/file-server-test:v1'
          imagePullPolicy: IfNotPresent
          name: public-service-web-file-test
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```
- 然后运行 以下命令 端口根据上面填的更改
```
kubectl port-forward <your-pod-name> 8848:8848 -n <your namespace> --address=0.0.0.0
```
