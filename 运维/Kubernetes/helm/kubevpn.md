# kubevpn安装
**[官方文档](https://kubevpn.dev/zh/docs/quickstart)**

## 使用helm安装kubevpn的服务端
**注意：记得更换[镜像源](../../容器/docker/docker.md#docker 源)**
1. 添加kubevpn下载地址
```bash
helm repo add kubevpn https://raw.githubusercontent.com/kubenetworks/kubevpn/master/charts
```
2. 使用命令安装
```bash
helm install kubevpn kubevpn/kubevpn -n kubevpn --create-namespace
```
# 安装kubevpn客户端
1. 客户端下载地址,根据自己系统安装
```url
https://github.com/kubenetworks/kubevpn/releases
```
2. 使用脚本，在服务端生成.kube/config文件
```
#!/bin/bash
set -e

# 配置区域，按需修改
NAMESPACE="kubevpn"
SERVICEACCOUNT_NAME="wangyl-dev"
CLUSTERROLE_NAME="kubevpn-dev"
KUBECONFIG_OUTPUT="./wangyl-dev-vpn.yaml"
TOKEN_EXPIRATION="8760h"  # 1年有效期（8760小时）
  
# 1. 创建命名空间（如果不存在）
kubectl get ns $NAMESPACE >/dev/null 2>&1 || kubectl create ns $NAMESPACE

# 2. 创建 ServiceAccount
kubectl get sa $SERVICEACCOUNT_NAME -n $NAMESPACE >/dev/null 2>&1 || kubectl create sa $SERVICEACCOUNT_NAME -n $NAMESPACE

  

# 3. 创建 ClusterRole（自定义权限）
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: $CLUSTERROLE_NAME
  namespace: $NAMESPACE
rules:
  - apiGroups: [""]
    resources:
      - pods
      - pods/exec
      - pods/portforward
    verbs:
      - get
      - list
      - watch
      - create 
  # 读取 Services、Endpoints、Namespaces
  - apiGroups: [""]
    resources:
      - services
      - endpoints
      - namespaces
    verbs:
      - get
      - list
      - watch
  # 读取 ConfigMaps（含更新）
  - apiGroups: [""]
    resources:
      - configmaps
    verbs:
      - get
      - list
      - watch
      - update
      - patch
  # 读取 Secrets
  - apiGroups: [""]
    resources:
      - secrets
    verbs:
      - get
      - list
      - watch
  # 读取 Deployments
  - apiGroups: ["apps"]
    resources:
      - deployments
    verbs:
      - get
      - list
      - watch
EOF 

# 4. 创建 ClusterRoleBinding
kubectl get rolebinding ${SERVICEACCOUNT_NAME}-binding -n ${NAMESPACE} >/dev/null 2>&1 || \
kubectl create rolebinding ${SERVICEACCOUNT_NAME}-binding \
  --role=${CLUSTERROLE_NAME} \
  --serviceaccount=${NAMESPACE}:${SERVICEACCOUNT_NAME} \
  --namespace=${NAMESPACE}

# 5. 获取 API Server 地址和 CA 证书
API_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
CA_DATA=$(kubectl config view --raw --minify -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')

# 6. 获取 ServiceAccount Token 并设置长过期时间
TOKEN=$(kubectl create token $SERVICEACCOUNT_NAME -n $NAMESPACE --duration $TOKEN_EXPIRATION)
  
# 7. 生成 kubeconfig 文件
cat > $KUBECONFIG_OUTPUT <<EOF
apiVersion: v1
kind: Config
clusters:
- name: k3s-cluster
  cluster:
    server: ${API_SERVER}
    certificate-authority-data: ${CA_DATA}
contexts:
- name: vpn-context
  context:
    cluster: k3s-cluster
    user: vpn-user
current-context: vpn-context
users:
- name: vpn-user
  user:
    token: ${TOKEN}
EOF

echo "生成长期有效的 kubeconfig 完成: $KUBECONFIG_OUTPUT"
echo "Token 有效期设置为: $TOKEN_EXPIRATION"
```
2. 使用命令连接
```bash
kubevpn connect --kubeconfig .\kubeconfig-vpn.yaml -n kubevpn
```