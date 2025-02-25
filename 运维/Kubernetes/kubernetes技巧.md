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
## k8s权限
### k8s创建.kubeconfig文件
#### 创建基本用户脚本
**注意：公网ip必须要要SSL证书**
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
  --namespace=${NAMESPACE} \  # 设置ClusterRole可以不创建
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
	- `CA_CERT` 和 `CA_KEY` 证书路径
	- `USERNAME`
	- `NAMESPACE`
绑定权限请看**创建 RBAC模型**

### 创建ServiceAccount
[官方文档](https://kubernetes.io/zh-cn/docs/concepts/security/service-accounts/)
```
kubectl create serviceaccount <sa-name> -n <namespace>
```
绑定权限请看**创建 RBAC模型**

### 创建 RBAC模型
**注意：更复杂的操作请看[官方文档](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/)**
#### API 对象
RBAC API 声明了四种 Kubernetes 对象：**Role**、**ClusterRole**、**RoleBinding** 和 **ClusterRoleBinding**。
#### [Role 和 ClusterRole的区别](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/#role-and-clusterole)

| **属性**   | **Role**                     | **ClusterRole**                    |
| -------- | ---------------------------- | ---------------------------------- |
| **作用范围** | 限定于某个命名空间（namespace）         | 覆盖整个 Kubernetes 集群（cluster-wide）   |
| **资源类型** | 只限于命名空间内的资源                  | 可以访问命名空间资源和集群级别的资源（如 `nodes`）      |
| **绑定方式** | 通过 `RoleBinding` 绑定到命名空间中的用户 | 通过 `ClusterRoleBinding` 绑定到集群级别的用户 |
| **适用场景** | 适用于命名空间级别的权限控制               | 适用于集群级别的权限控制，或者跨命名空间控制权限           |
| **例子**   | 限制开发人员只能访问某个命名空间中的资源         | 允许某个用户查看所有命名空间的 `pods` 或集群级别资源     |

```
#创建一个可以在default命令空间下只能操作pods的Role
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role 
metadata:
  namespace: default
  name: dev-user-role
rules:
- apiGroups: ["", ]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
EOF
#创建一个可以在default命令空间下只能操作pods的Role
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole 
metadata:
  name: dev-user-cluster-role
rules:
- apiGroups: ["", ]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
EOF
```
 rules的三个参数解释 
1. `apiGroups` 表

| **API Group**                  | **描述**                                                             |
| ------------------------------ | ------------------------------------------------------------------ |
| `""` (core API group)          | 核心 API 组，包含最基本的资源，如 `pods`, `services`, `namespaces` 等。            |
| `apps`                         | 与应用相关的资源，如 `deployments`, `statefulsets`, `replicasets` 等。         |
| `batch`                        | 与批处理相关的资源，如 `jobs`, `cronjobs` 等。                                  |
| `networking.k8s.io`            | 与网络相关的资源，如 `ingresses`, `networkpolicies` 等。                       |
| `rbac.authorization.k8s.io`    | 与角色和权限相关的资源，如 `roles`, `rolebindings` 等。                           |
| `admissionregistration.k8s.io` | 与 Kubernetes admission 控制器相关的资源，如 `mutatingwebhookconfigurations`。 |
2. `resources` 表

| **资源**            | **描述**                         |
| ----------------- | ------------------------------ |
| `pods`            | Pod 资源，用于描述运行的容器。              |
| `deployments`     | 部署资源，用于声明和管理应用程序的部署。           |
| `services`        | 服务资源，用于定义应用程序访问点。              |
| `ingresses`       | 入口资源，用于外部访问 Kubernetes 集群中的服务。 |
| `jobs`            | 作业资源，用于批处理任务的管理。               |
| `statefulsets`    | 有状态集资源，管理有状态应用程序的副本。           |
| `replicasets`     | 副本集资源，确保某些副本数的 Pods 存在。        |
| `networkpolicies` | 网络策略资源，用于定义 Pods 之间的通信规则。      |
| `roles`           | 角色资源，定义对资源的访问控制规则（命名空间级别）。     |
| `rolebindings`    | 角色绑定资源，将 `Role` 绑定到具体用户或服务账户。  |
3. `verbs` 表

| **操作（Verb）**       | **描述**                      |
| ------------------ | --------------------------- |
| `get`              | 查看资源的详细信息。                  |
| `list`             | 列出指定资源的所有实例。                |
| `watch`            | 监听资源的变更，实时获取资源的更新。          |
| `create`           | 创建新的资源实例。                   |
| `update`           | 更新已有资源实例的配置或状态。             |
| `delete`           | 删除指定资源实例。                   |
| `patch`            | 更新资源的部分内容，通常是通过合并更新来修改部分属性。 |
| `deletecollection` | 删除资源集合中的多个资源实例。             |
#### RoleBinding 和 ClusterRoleBinding
[RoleBinding 和 ClusterRoleBinding区别](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)
- 创建RoleBinding
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: dev-user-role
subjects:
- kind: User
  name: dev-user-role
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-user-role
  apiGroup: rbac.authorization.k8s.io
```
- 创建ClusterRoleBinding
```
apiVersion: rbac.authorization.k8s.io/v1
# 此集群角色绑定允许 “manager” 组中的任何人访问任何名字空间中的 Secret 资源
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager      # 'name' 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
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
