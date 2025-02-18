# jenkins安装

## 使用docker compose 安装
```
version: '3.9'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins:2.492.1
    restart: always
    user: root
    environment:
     - JAVA_OPTS=-Duser.timezone=Asia/Shanghai
    volumes:
     - "/storage/jenkins-volume/jenkins-data:/var/jenkins_home"
     - "/storage/jenkins-volume/work-tools:/opt/work-tools"
     - "/var/run/docker.sock:/var/run/docker.sock"
     - "/usr/bin/docker:/usr/bin/docker"
    ports:
     - 9090:8080
```
使用命令启动后，等代即可  
```
docker compose -f docker-compose.yaml up -d
```
## 配置jenkins
### 解锁jenkins
用浏览器访问http://127.0.0.1:9090,等待加载完毕 ,会出现以下画面
![](./image/jenkins-install/0.png)
解锁jenkins。获取密钥有两个方法
- 使用 **docker logs jenkins**查看日志获取密钥
- 使用**docker exec -it jenkins /bin/bash**进入容器 再使用 **cat /var/jenkins_home/secrets/initialAdminPassword**获取密钥
### 使用默认插件
**注意：这个步骤很需要时间请耐心等待，大约要5-15分钟左右**
使用密钥进入后选择 *安装推荐的插件*  按钮
![](./image/jenkins-install/1.png)
选择后会出现以下情况,等待即可
![](./image/jenkins-install/2.png)
### 创建第一个用户
根据需求创建第一个用户
![](./image/jenkins-install/3.png)
### 进入控制面板
一直点下一步即可
![](./image/jenkins-install/4.png)
### 安装所需插件
进入*插件管理*的点击流程
**系统管理->插件管理**
进入以下界面即可
![](./image/jenkins-install/5.png)
选择 **Available plugins**，搜索以下插件安装
- Generic Webhook Trigger 用于webhook监听仓库
- Git plugin 用于安装 git
- Maven Integration 用于安装 maven
- NodeJs 用于安装Node.js
其他插件，按需选择
- Docker 使用jenkins封装好的docker api 打包
- publish over SSH 用于远程操作和传输文件到其他服务器
- [DingTalk](https://jenkinsci.github.io/dingtalk-plugin/guide/getting-started.html) 用于钉钉通知
### 修改全局工具配置
进入*全局工具配置*的点击流程
**系统管理-># 全局工具配置**
进入以下界面即可
![](./image/jenkins-install/6.png)
1. 配置全局maven配置文件

![](./image/jenkins-install/7.png)
 1. 配置jdk
 ![](./image/jenkins-install/8.png)
 2. 配置maven
 ![](./image/jenkins-install/9.png)
 3. 配置node.js
 ![](./image/jenkins-install/10.png)
 点击保存即可 **现在你就可以使用jenkins了**