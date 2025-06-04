1. 进入修改需要的文件
```
vim /etc/containerd/config.toml
```
2. 添加仓库地址和账户密码
```
# 在[plugins."io.containerd.grpc.v1.cri".registry.configs]后面添加
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
         [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.changtech.cn".auth]
          username = "admin"
          password = "iKTpUuoNvnPcMPwL"

# 在[plugins."io.containerd.grpc.v1.cri".registry.mirrors]后面添加
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors] 
                [plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.changtech.cn"]
          endpoint = ["https://harbor.ten.com"]
```