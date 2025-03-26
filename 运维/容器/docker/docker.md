# docker 源

[镜像源](https://github.com/DaoCloud/public-image-mirror/issues/2328)
[国内镜像查询库](https://docker.aityp.com/)

# docker 远程操控
## 下载客户端软件
[下载地址](https://download.docker.com/win/static/stable/x86_64/)
[官方文档](https://docs.docker.com/engine/security/protect-access/)
## 生成 SSH 密钥对（本地操作）
```bash
ssh-keygen -t ed25519 -f ~/.ssh/docker-remote
```
## 创建用户,使用密钥登录
1. 创建用户并加入 `docker` 组：
```bash
# 添加用户
useradd -m -G docker docker-user
# 修改密码
passwd docker-user 
```
2. 将公钥上传到 `docker-user` 账户：
```bash
ssh-copy-id -i ~/.ssh/docker-remote.pub docker-user@远程服务器IP
```
**注意：记得开启密钥登录**
编辑远程服务器 `/etc/ssh/sshd_config`
```
PubkeyAuthentication yes
```
重启 SSH 服务：
```bash
systemctl restart sshd
```
3. 修改 SSH 配置中的 `User` 字段：
```
Host docker-remote
  HostName 172.16.0.1
  User local-docker
  IdentityFile ~/.ssh/docker-remote
  IdentitiesOnly yes
```
## 创建 Docker Context（本地操作）
**注意：名称要和ssh配置文件Host一样**
```bash
docker context create remote-docker --docker "host=ssh://docker-remote" --description "Remote Docker via SSH"

# 切换 Context
docker context use remote-docker

# 执行远程 Docker 命令
docker ps                   # 应直接输出结果，无需密码
```