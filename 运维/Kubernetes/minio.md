# minio k8s集群版
## 安装前置环境
### 安装Minio DirectPV
根据文档安装，第六步不需要做
[DirectPV文档](https://min.io/directpv)
## 安装集群Minio
根据文档安装
[Minio文档](https://www.minio.org.cn/docs/minio/kubernetes/upstream/operations/installation.html)

# minio单机版
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations: {}
  labels:
    app: minio
  name: minio
  namespace: minio-space
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Retain
  podManagementPolicy: OrderedReady
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: minio
  serviceName: ''
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: minio
    spec:
      containers:
        - args:
            - server
            - /data
            - '--console-address'
            - ':9090'
          env:
            - name: MINIO_BROWSER_REDIRECT_URL
              value: 'http://minio-console.cn'
            - name: MINIO_SERVER_URL
              value: 'http://minio.cn'
            - name: MINIO_ROOT_USER
              value: e4da512
            - name: MINIO_ROOT_PASSWORD
              value: jhkh131q1423
          image: 'minio:RELEASE.2024-02-24T17-11-14Z'
          imagePullPolicy: IfNotPresent
          name: minio
          ports:
            - containerPort: 9000
              protocol: TCP
            - containerPort: 9090
              protocol: TCP
          resources:
            limits:
              cpu: 1500m
              memory: 6656Mi
            requests:
              cpu: '1'
              memory: 3Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /data
              name: minio-data
      dnsConfig:
        options:
          - name: ndots
            value: '2'
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
        - effect: NoSchedule
          key: minio.node
          operator: Equal
          value: minio
      volumes:
        - name: minio-data
          persistentVolumeClaim:
            claimName: minio-data
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-data
  namespace: minio-space
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Ti
  storageClassName: nfs-client
```
