# pipline流水线脚本模板
```
pipeline {
    agent any
    options {
        timeout(time: 15, unit: 'MINUTES')
    }
    tools {
        maven 'maven-3.8.5'
        jdk 'jdk-11'
    }
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref']
            ],
            token: 'server',
            regexpFilterExpression: '^refs/heads/(master|develop)$',
            regexpFilterText: '$ref',
        )
    }
    environment {
        BRANCH = env.ref.split("/")[2].trim()
        WORKSPACE = "/opt/workspace/${JOB_NAME}"
        REPOSITORY_URL = 'harbor.cn/public'
        CAUSE = "${currentBuild.getBuildCauses()[0].shortDescription}"
    }
    stages {
        stage('pull-code') {
            steps {
                script{
                    if("develop".equals(BRANCH) || "Started by user root".equals(CAUSE)){
                        echo '测试环境'
                        git branch: 'develop', credentialsId: 'd8aba4d3-6696-4b49-ac1f-79a226d0541e', url: 'https://zengxh:zengXh_8702@gitlabland-mass-service.git'
                        sh '''
                            git checkout release
                            git merge develop
                        '''
                    }
                    if("master".equals(BRANCH)){
                        echo '正式环境'
                        git branch: 'master', credentialsId: 'd8aba4d3-6696-4b49-ac1f-79a226d0541e', url: 'https://zengxh:zengXh_8702@cnland-mass-service.git'
                    }
                }

            }
        }
        stage('Fill Tags') {
            steps {
                script {
                    try {
                        // 设置10秒超时，10秒后自动继续执行
                        timeout(time: 10, unit: 'SECONDS') {
                            // 用户输入，若超时则会触发 timeout 错误
                            def userChoice = input(
                                    id: 'UserInput',
                                    message: '请输入标签号并点击继续（10秒超时后自动使用默认版本）:',
                                    parameters: [
                                            string(name: 'tag', description: '请输入标签号')
                                    ]
                            )
                            // 将用户输入的标签赋值给环境变量
                            env.CUSTOM_TAG = userChoice
                        }
                    } catch (err) {
                        // 超时或错误，使用默认值
                        echo "未输入标签，使用默认标签..."
                    }
                }
            }
        }
        stage('Checkout') {
            steps {
                script {
                    if (env.CUSTOM_TAG) {
                        echo "正在检出标签: ${env.CUSTOM_TAG}"
                        sh "git checkout ${env.CUSTOM_TAG}"
                    } else {
                        echo "没有选择标签，跳过 git checkout"
                    }
                }
            }
        }
        stage('compile'){
            steps{
                echo '编译项目'
                sh 'rm -rf ./target'
                sh 'mvn clean install -Dmaven.test.skip=true'
            }
        }
        stage('file-transfer'){
            steps{
                sh '''
                    rm -rf ${WORKSPACE}
                    mkdir -p ${WORKSPACE}
                    cp ./src/main/resources/Dockerfile ${WORKSPACE}/Dockerfile
                    cp ./target/*.jar ${WORKSPACE}/root.jar
                '''
            }
        }
        stage('build-docker-image'){
            steps{
                script{
                    if("develop".equals(BRANCH) || "Started by user root".equals(CAUSE)){
                        sh "docker build -t ${REPOSITORY_URL}/${JOB_NAME}-test:v1.0.${BUILD_NUMBER} --build-arg envType=test ${WORKSPACE}"
                    }
                    if("master".equals(BRANCH)){
                        sh "docker build -t ${REPOSITORY_URL}/${JOB_NAME}-prod:v1.0.${BUILD_NUMBER} --build-arg envType=prod ${WORKSPACE}"
                    }
                }
            }
        }
        stage('push-image'){
            steps{
                script{
                    if("develop".equals(BRANCH) || "Started by user root".equals(CAUSE)){
                        echo '测试环境'

                        echo '上传项目镜像'
                        sh "docker push ${REPOSITORY_URL}/${JOB_NAME}-test:v1.0.${BUILD_NUMBER}"
                    }
                    if("master".equals(BRANCH)){
                        echo '正式环境'

                        echo '上传项目镜像'
                        sh "docker push ${REPOSITORY_URL}/${JOB_NAME}-prod:v1.0.${BUILD_NUMBER}"
                    }
                }

            }
        }
        stage('push-code'){
            steps{
                echo '上传代码'
                sh 'git config --global user.name "曾星红"'
                sh 'git config --global user.email "2542823317@qq.com"'
                script{
                    if (env.CUSTOM_TAG){
                        if("develop".equals(BRANCH) || "Started by user root".equals(CAUSE)){
                            sh 'git push origin release'
                        }
                    }

                }
            }
        }
        stage('deploy'){
            steps{
                script{
                    if("develop".equals(BRANCH) || "Started by user root".equals(CAUSE)){
                        echo '测试环境'

                        echo '部署项目'
                        sh '''

                        '''
                    }
                    if("master".equals(BRANCH)){
                        echo '正式环境'

                        echo '部署项目'
                        sh '''

                        '''

                    }
                }
            }
        }
    }
    post{
        success{
            script{
                def STATUS = false
                def HASH_PATH = sh(script: "git log -1 '--pretty=format:%h'", returnStdout: true).trim()
                def AUTHOR = sh(script: "git log -1 '--pretty=format:%an'", returnStdout: true).trim()
                def COMMIT_DATA = sh(script: "git log -1 '--pretty=format:%cd'", returnStdout: true).trim()
                def COMMIT_MSG = sh(script: "git log -1 '--pretty=format:%s'", returnStdout: true).trim()
                def TEXT_CONTENT = [
                                                           '# 测试环境',
                                                           '- 平台名称：**浙江考古研究所**',
                                                           '- 项目名称：' + JOB_NAME,
                                                           '- 编译编号：' + BUILD_NUMBER,
                                                           '- 作者：' + AUTHOR,
                                                           '- 提交时间：' + COMMIT_DATA,
                                                           '- 提交信息：' + COMMIT_MSG,
                                                           '- git-hash：' + HASH_PATH,
                                                           '- 部署状态：<font color=blue>成功</font>',
                                                       ]
                if("master".equals(BRANCH)){
                    STATUS = true
                    TEXT_CONTENT[0] = '# 正式环境'
                    def TAG_NAME = sh(script: "git tag --sort=-creatordate | head -1", returnStdout: true).trim()
                    TEXT_CONTENT.add('- 标签号：' + TAG_NAME)
                }
                if (env.CUSTOM_TAG){
                    TEXT_CONTENT[0] = '# 自定义标签'
                    TEXT_CONTENT.add('- 标签号：' + env.CUSTOM_TAG)
                }
                dingtalk(
                    robot: 'e0bbcf81-0432-45e4-84a1-4667fb4b2483',
                    type: 'MARKDOWN',
                    atAll: STATUS,
                    title: '自动化部署',
                    text: TEXT_CONTENT
                )
            }
        }
        failure{
            sh 'rm -rf ./*'
            script{
                def HASH_PATH = sh(script: "git log -1 '--pretty=format:%h'", returnStdout: true).trim()
                def AUTHOR = sh(script: "git log -1 '--pretty=format:%an'", returnStdout: true).trim()
                def COMMIT_DATA = sh(script: "git log -1 '--pretty=format:%cd'", returnStdout: true).trim()
                def COMMIT_MSG = sh(script: "git log -1 '--pretty=format:%s'", returnStdout: true).trim()
                def TEXT_CONTENT = [
                                                           '# 测试环境',
                                                           '- 平台名称：**浙江考古研究所**',
                                                           '- 项目名称：' + JOB_NAME,
                                                           '- 编译编号：' + BUILD_NUMBER,
                                                           '- 作者：' + AUTHOR,
                                                           '- 提交时间：' + COMMIT_DATA,
                                                           '- 提交信息：' + COMMIT_MSG,
                                                           '- git-hash：' + HASH_PATH,
                                                           '- 部署状态：<font color=red>失败</font>',
                                                       ]
                if("master".equals(BRANCH)){
                    STATUS = true
                    TEXT_CONTENT[0] = '# 正式环境'
                    def TAG_NAME = sh(script: "git tag --sort=-creatordate | head -1", returnStdout: true).trim()
                    TEXT_CONTENT.add('- 标签号：' + TAG_NAME)
                }
                if (env.CUSTOM_TAG){
                    STATUS = true
                    TEXT_CONTENT[0] = '# 自定义标签'
                    TEXT_CONTENT.add('- 标签号：' + env.CUSTOM_TAG)
                }
                dingtalk(
                    robot: 'e0bbcf81-0432-45e4-84a1-4667fb4b2483',
                    type: 'MARKDOWN',
                    atAll: true,
                    title: '自动化部署',
                    text: TEXT_CONTENT
                )
            }
        }
        always{
            script{
                if("develop".equals(BRANCH) || "Started by user root".equals(CAUSE)){
                    sh "docker rmi ${REPOSITORY_URL}/${JOB_NAME}-test:v1.0.${BUILD_NUMBER}"
                }
                if("master".equals(BRANCH)){
                    sh "docker rmi ${REPOSITORY_URL}/${JOB_NAME}-prod:v1.0.${BUILD_NUMBER} "
                }
            }
        }
    }
}
```
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