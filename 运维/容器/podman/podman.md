# podman
## podman的Rootless模式问题
### 1.用户退出导致服务关闭
在 Rootless 模式下，Podman 进程是用户会话的子进程。一旦 SSH 断开，系统会默认杀掉该用户的所有进程。
```shell
# 执行此命令开启驻留
sudo loginctl enable-linger $(whoami)
```
### 2.服务器重启，服务没同时启动

