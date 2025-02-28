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
		- DRONE_GITEA_SERVER=https://gitea.com # gitea仓库地址
		- DRONE_GITEA_CLIENT_ID=0cd27bee-0d0f-4243-be16-2bb1ecf34cd1
		- DRONE_GITEA_CLIENT_SECRET=gto_v6hmtktutt4bdffsxrhkvabekmgktldg5yqvdc3kvowos3ddjamq
		- DRONE_RPC_SECRET=a82d353bc1d32b8046c4e0b1b8ba5ad8
		- DRONE_SERVER_HOST=drone.changtech.cn
		- DRONE_SERVER_PROTO=https
		- DRONE_USER_CREATE=username:zengxh,admin:true
		ports:
		- 9090:80
		- 9091:443
	drone-runner:
		image: drone/drone-runner-docker:1.8.3
		container_name: drone-runner
		restart: always
		dns:
		- 119.29.29.29
		volumes:
		- /var/run/docker.sock:/var/run/docker.sock
		environment:
		- DRONE_RPC_PROTO=https
		- DRONE_RPC_HOST=drone.changtech.cn
		- DRONE_RPC_SECRET=a82d353bc1d32b8046c4e0b1b8ba5ad8
		- DRONE_RUNNER_CAPACITY=2
		- DRONE_RUNNER_NAME=local-drone-runner
		ports:
		- 3000:3000

```

