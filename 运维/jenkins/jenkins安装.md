# jenkins安装
## 原来的镜像没有权限 自制作带权限的镜像
```
构建镜像给与授权
FROM jenkins/jenkins:{版本号}
USER root
```
