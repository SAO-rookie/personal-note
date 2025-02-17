# Prometheus全套组件安装
[官方文档](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md)
## helm 增加仓库地址
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# 查看仓库有里有什么
helm search repo prometheus-community
```
## 下载对应chart压缩包
```
helm pull prometheus-community/kube-prometheus-stack
```
## 修改配置文件
1. 根据需求给 prometheus和alertmanager添加存储
2. 修改charts/kube-state-metrics/values.yml的
## 使用helm启动
**根据自身情况修改valus.yaml文件**

```
helm install prometheus-server -n monitor-space --create-namespace .
```
安装需要一些，等待安装完成即可