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
 run -p 8080:8080 your-image-name
```


```bash
docker run -d --name mysql_container -e MYSQL_ROOT_PASSWORD=6606 -p 6606:3306 mysql


docker run -p 3307:3306 --name mysql6607  -v D:\bakup\mysql\log:/var/log/mysql  -v /Users/hyc/DockerStudy/mysql/data:/var/lib/mysql  -v D:\bakup\mysql\conf:/etc/mysql  -e MYSQL_ROOT_PASSWORD=6607  -d mysql

docker run -p 6607:3306 --name mysql6607  -v /d/bakup/mysql/log:/var/log/mysql  -v /d/bakup/mysql/data:/var/lib/mysql  -v /d/bakup/mysql/conf:/etc/mysql  -e MYSQL_ROOT_PASSWORD=6607  -d mysql

docker cp mysql:/etc/mysql/my.cnf F:\docker\mysql8\conf

```

```bash

apt update
apt upgrade -y
apt install curl vim wget gnupg dpkg apt-transport-https lsb-release ca-certificates


curl -sS https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
docker version
docker compose version
systemctl restart docker
service   docker start
```