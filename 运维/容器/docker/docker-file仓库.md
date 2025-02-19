# 自定义minio client镜像
```DockeFile
FROM debian:bookworm

# 修改时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN sed -i 's|http://deb.debian.org|http://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list.d/debian.sources && \
apt update && \
apt install -y vim cron

ADD https://dl.minio.org.cn/client/mc/release/linux-amd64/mc /usr/bin/mc

RUN chmod +x /usr/bin/mc
```

# 自定义Openvpn镜像
****
```DockeFile
FROM debian:bookworm

# 修改时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN sed -i 's|http://deb.debian.org|http://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list.d/debian.sources && \
apt update && \
apt install -y wget gnupg curl openvpn vim cron psmisc 

VOLUME ["/storage","vpn-config","back-script"]
```
根据需求下载软件
- [cifs-utils](../../linux/文件共享/smb协议.md#客户端挂载) smb协议将window共享文件挂载到容器里
- default-mysql-client  mysq客户端
- postgresql-client  postgresql客户端
- nfs-common nfs客户端挂载linux共享文件到容器里