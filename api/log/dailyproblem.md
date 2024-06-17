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

```dockerfile
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