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

这些常量可用于 Jenkins 中的任何脚本（例如构建步骤、后置操作等），以帮助您编写更灵活和可重用的脚本。
# jenkins需要安装的插件
- Generic Webhook Trigger Plugin 用于webhook监听仓库
- DingTalk 用于叮叮通知
- Git plugin 
- Maven Integration
- publish over SSH 用于远程操作其他服务器
- NodeJs 用于打包前端项目
# jenkins监听分支和tag自动化打包
**注意：必须有Generic Webhook Trigger Plugin才可以使用**

```
pipeline {
    agent any
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref'],
            ],
            token: 'old-gtz-server',
            regexpFilterExpression: '^(refs/heads/(master|develop)|refs/tags/.{0,10})$',
            regexpFilterText: '$ref',
        )
    }
    environment {
        EVENT = env.ref.split("/")[1].trim()
        BRANCH_OR_TAG = env.ref.split("/")[2].trim()
        WORKSPACE = "/opt/workspace/${JOB_NAME}"
        CAUSE = "${currentBuild.getBuildCauses()[0].shortDescription}"
    }
    stages {
        stage('Hello') {
            steps {
                script{
                     git 'https://zengxh:zengXh_8702@gitlab.changtech.cn/gtz/springboot-webserver-framework.git'
                     sh "git tag -l -n"
                     sh 'echo ${BRANCH_OR_TAG}'
                     sh 'echo ${EVENT}'
                    
                }
            }
        }
    }
}

```