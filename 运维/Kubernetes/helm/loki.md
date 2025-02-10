# loki 安装
[官方文档](https://github.com/grafana/helm-charts/blob/main/charts/loki-stack/README.md)
## helm 增加仓库地址
```
helm repo add grafana https://grafana.github.io/helm-charts
# 查看仓库有里有什么
helm search repo loki
```
## 下载对应chart压缩包
```
helm pull grafana/loki-stack
```
## 使用helm启动
**根据自身情况修改valus.yaml文件**
```
helm install loki . -n monitor-space --create-namespace
```