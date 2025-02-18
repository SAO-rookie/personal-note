# 自定义minio client镜像
```
FROM ubuntu:24.10

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

RUN apt update && apt install -y wget gnupg curl vim cron psmisc

ADD https://dl.minio.org.cn/client/mc/release/linux-amd64/mc /usr/bin/mc

RUN chmod +x /usr/bin/mc
```