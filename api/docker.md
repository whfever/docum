# Dokcer
## account
```
cat /root/.docker/config.json
echo [auth后面的那一串序列] | base64 -d -
```
Secret

## Dockerfile

```bash
# 使用带有Maven的官方Java镜像作为基础镜像  
FROM maven:3.6.3-jdk-11 as build  
  
# 将当前目录设置为工作目录  
WORKDIR /app  
  
# 将项目的pom.xml和src目录复制到镜像中  
COPY pom.xml ./  
COPY src ./src  
  
# 运行Maven构建命令  
RUN mvn clean package  
  
# 使用官方Java运行时镜像作为基础镜像  
FROM openjdk:11-jre-slim  
  
# 将构建好的jar文件从上一个阶段复制到当前镜像中  
COPY --from=build /app/target/*.jar /usr/app/app.jar  
  
# 设置环境变量，指定JAR文件的路径  
ENV JAVA_OPTS=""  
ENV APP_JAR=/usr/app/app.jar  
  
# 容器启动时执行的命令  
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar $APP_JAR" ]
```

```bash
docker build -t your-image-name .
docker run -p 8080:8080 your-image-name
```