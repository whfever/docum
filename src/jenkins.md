# jenkins
> 持续集成
插件{git，manage，pipline}

## pipeline
jenkins file
agent - 指定在哪台机器上执行任务，还记得上面配置Node时候填的Label吗，如果这两个label匹配得上，就在该Node中执行
stage - 组成工作流的大的步骤，这些步骤是串行的，例如build，test，deploy等
steps - 描述stage中的小步骤，同一个stage中的steps可以并行
sh - 执行shell命令
input - 需要你手动点击确定，Pipeline才会进入后续环节，常用于部署环节，因为很多时候部署都需要人为的进行一些确认
post- 所有pipeline执行完成后，会进入post环节，该环节一般做一些清理工作，同时还可以判断pipeline的执行状态
```

pipeline {
    agent {
        label 'master' /* 执行节点 */
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
            }
        }
        stage('Deploy - Staging') {
            steps {
                sh './deploy staging'
                sh './run-smoke-tests'
            }
        }
        stage('Sanity check') {
            steps {
                input "Does the staging environment look ok?"
            }
        }
        stage('Deploy - Production') {
            steps {
                echo './deploy production'
            }
        }
    }

    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}

```
![pipelinefile](https://uufefile.uupt.com/PicLib/uunote/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220718164322_1658731487755.png)

## WebHook
