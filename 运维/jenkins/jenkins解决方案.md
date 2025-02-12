# Jenkins 的一些常量
Jenkins 中有许多常量可以在脚本中使用，以下是一些常见的常量：
1.  `JENKINS_HOME`：Jenkins 安装目录的路径。
2.  `BUILD_ID`：当前构建的唯一标识符。
3.  `BUILD_NUMBER`：当前构建的编号。
4.  `BUILD_TAG`：当前构建的唯一标签。
5.  `BUILD_URL`：当前构建的 URL。
6.  `EXECUTOR_NUMBER`：当前执行构建的执行器编号。
7.  `JOB_NAME`：当前构建的作业名称。
8.  `NODE_NAME`：当前执行构建的节点名称。
9.  `WORKSPACE`：当前构建的工作空间路径。
更多请看：
- http://jenkins-server-url/env-vars.html/
- http://jenkins-server-url/pipeline-syntax/globals
这些常量可用于 Jenkins 中的任何脚本（例如构建步骤、后置操作等），以帮助您编写更灵活和可重用的脚本。

# 判断是否时手动构建
使用以下命令能看到相关信息
```
currentBuild.getBuildCauses()
```