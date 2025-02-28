# 安装drone
**注意：当git仓库使用https使用 drone 也要使用https**
根据[官方文档](https://docs.drone.io/)]安装
* docker-compose.yaml 仅用于参考
```docker compose
version: '3.9'
services:
	drone-server:
		image: drone/drone:2.25.0
		container_name: drone-server
		restart: always
		dns:
		- 119.29.29.29
		volumes:
		- /storage/drone-volume/:/data
		environment:
		- DRONE_GITEA_SERVER=http://127.0.0.1:9090 # gitea仓库地址
		- DRONE_GITEA_CLIENT_ID=0cd27  # gitea客户id
		- DRONE_GITEA_CLIENT_SECRET=gto_v6hmtktutt # gitea客户密钥
		- DRONE_RPC_SECRET=a82d353bc1d32b8046c4e0b1b8ba5ad8 # 与drone-runner交互密钥
		- DRONE_SERVER_HOST=127.0.0.1
		- DRONE_SERVER_PROTO=http
		- DRONE_USER_CREATE=username:zengxh,admin:true # 配置管理员权限
		ports:
		- 80:80
		- 443:443
	drone-runner:
		image: drone/drone-runner-docker:1.8.3
		container_name: drone-runner
		restart: always
		dns:
		- 119.29.29.29
		volumes:
		- /var/run/docker.sock:/var/run/docker.sock
		environment:
		- DRONE_RPC_PROTO=http
		- DRONE_RPC_HOST=127.0.0.1
		- DRONE_RPC_SECRET=a82d353bc1d32b8046c4e0b1b8ba5ad8
		- DRONE_RUNNER_CAPACITY=2
		- DRONE_RUNNER_NAME=local-drone-runner
		ports:
		- 3000:3000
```

# 配置CI仓库
在浏览器访问`http://127.0.0.1`出现以下界面然后点击继续即可
![](./image/1.png)
登陆git仓库账号密码
![](./image/2.png)