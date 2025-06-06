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
使用以下函数能看到相关信息
```
currentBuild.getBuildCauses()
```
包含 *Generic Cause* 字段表示webhook触发，包含 *Started by user* 字段表示手动触发 以下是简单脚本展示
```
pipeline {
    agent any
    // 这是使用了Generic Webhook Trigger插件
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref']
            ],
            token: 'test',
            regexpFilterExpression: '^refs/heads/(master|develop)$',
            regexpFilterText: '$ref',
        )
    }
    environment {
        CAUSE = "${currentBuild.getBuildCauses()}"
    }
    stages {
        stage('pull-code') {
            steps {
                script{
                    if(CAUSE.contains("Started by user")){
                        echo "手动触发"
                    }else{
                        echo "webhook"
                    }
                }
            }
        }
    }
}
```