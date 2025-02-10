# nfs-subdir-external-provisioner的安装
*作用：搭建nfs客户端，统一管理存储，实现存储分离*
*环境：k8s 1.24.5，nfs系统,helm 3.11.0*
**注意：所有工作节点，必须安装nfs工具包，否则无法使用nfs**
* [NFS安装文档](../../../NFS安装.md)
* [HELM安装文档](https://helm.sh/zh/docs/intro/install/)
* [nfs-subdir-external-provisioner安装文档](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/blob/master/charts/nfs-subdir-external-provisioner/README.md)
## 以命令行方式安装
*   helm添加仓库地址
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```
*   helm命令行直接安装
```
helm install nfs-client \ nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=x.x.x.x \
--set nfs.path=/exported/path \
--set image.repository=registry.cn-hangzhou.aliyuncs.com/xzjs/nfs-subdir-external-provisioner \
--set image.tag=v4.0.0 \
-n nfs-space \
--create-namespace 
```

*  检查是否安装成功
```
# 使用以下命令查看

kubectl get sc

# 出现数据表示安装成功

NAME         PROVISIONER                                                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-client-nfs-subdir-external-provisioner   Delete          Immediate           true                   7d19h
```
## 以压缩包方式安装
*   helm添加仓库地址
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```
* 下载nfs-subdir-external-provisioner压缩包
```
helm pull nfs-subdir-external-provisioner/nfs-subdir-external-provisioner
```
* 解压压缩包，修改相关配置文件
```
 tar -zxvf nfs-subdir-external-provisioner-4.0.18.tgz
 # 进入文件夹，修改nfs-subdir-external-provisioner\values.yaml配置文件，第5行和第12行代码
image:
  repository: registry.cn-hangzhou.aliyuncs.com/xzjs/nfs-subdir-external-provisioner
  tag: v4.0.0
  pullPolicy: IfNotPresent
imagePullSecrets: []

nfs:
  server: x.x.x.x  # nfs服务器的ip地址
  path: /nfs-storage # nfs
```
* 启动
```
helm install nfs-client -f values.yaml -n nfs-space --create-namespace
```
## 如何使用

编写pvc配置，pv由nfs客户端自动创建，绑定成功就可以使用了

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data # pvc名称
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi # 限制使用空间大小
  storageClassName: nfs-client # 选择自己搭建好的
```

编写StatefulSet运行时
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  podManagementPolicy: OrderedReady
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: mysql
  serviceName: ''
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mysql
    spec:
      containers:
        - env:
            - name: TZ
              value: Asia/Shanghai
            - name: MYSQL_ROOT_PASSWORD
              value: 123456
          image: 'mysql:8.0.31'
          imagePullPolicy: IfNotPresent
          name: mysql
          ports:
            - containerPort: 3306
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-date
      dnsConfig:
        options:
          - name: ndots
            value: '2'
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql-data-one
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate

```

运行起来就可以了