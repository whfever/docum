# Daily Problem

## WPS

-  任务栏只显示一个窗口
层叠窗口  
WinodwsTop
## Linux
- 使用环境变量
```shell
env VARIABLE_NAME
printenv VARIABLE_NAME

export MY_VAR="some_value"
unset MY_VAR
```

- 出错即退出
``` shell
#!/bin/bash
set -e  # 设置脚本在遇到错误时立即退出

command1 || { echo "command1 failed"; exit 1; }
command2 || { echo "command2 failed"; exit 1; }
command3 || { echo "command3 failed"; exit 1; }

```

- 同步执行代码

```shell

#!/bin/bash

# 启动Java程序，使用&将其放入后台执行
java -jar your-java-program.jar &

# 获取Java程序的进程ID
java_pid=$!

# 这里可以执行一些其他命令
echo "Java程序已在后台启动，正在执行其他任务..."

# ... 这里是其他命令 ...

# 等待Java程序完成
wait $java_pid

# Java程序执行完成后，执行后续命令
echo "Java程序执行完毕，所有任务已完成。"
```

- 列出命令
```shell
find /bin /usr/bin -type f -executable
env | grep -o '^[^=]*'
alias 

```


## vim
- 删除当前行到最后
```sh
:.,$d
```

## kubernetes
- 停止服务
```shell
kubectl delete deployment <deployment-name>
kubectl scale deployment <deployment-name> --replicas=0
kubectl delete pod <pod-name>
# 停止容器

kubectl exec <pod-name> -- /bin/bash -c "kill -STOP <container-id>"
# 暂停容器
kubectl patch pod <pod-name> -p '{"spec":{"terminationGracePeriodSeconds":0}}'
```

- ingress events 为 null
  
- 排查k8s nginx-ingress controller不工作问题 

- k8s service 一直处于 pending 状态
  
- 解决Docker无法查看日志和进入容器的问题
- docker exec 和 docker attach 的区别
- 查看容器日志
```sh
kubectl logs -f <pod-name> -n <namespace>
``

## Docker
- 构建java镜像

``` shell

# 使用官方Java镜像作为基础镜像
FROM maven:3.6.3-jdk-11 AS build

# 设置工作目录
WORKDIR /app

# 复制项目文件到工作目录
COPY src ./src
COPY pom.xml .

# 使用Maven编译项目
RUN mvn -f pom.xml clean package

# 构建一个轻量级的运行时镜像
FROM openjdk:11-jre-slim

# 设置工作目录
WORKDIR /app

# 从构建阶段复制编译后的jar文件到运行时镜像
COPY --from=build /app/target/*.jar ./app.jar

# 指定容器启动时执行的命令
CMD ["java", "-jar", "app.jar"]

```