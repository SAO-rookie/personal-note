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
# k8s的CRD配置
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
