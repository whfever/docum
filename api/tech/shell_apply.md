# shell

## listen port

```shell
#!/bin/bash

# 定义Java JAR文件路径
JAR_FILE="/path/to/your/yourfile.jar"

# 定义Java虚拟机选项（根据需要进行调整）
JAVA_OPTS="-Xmx512m -Xms256m"

# 定义Java应用的端口号
PORT=your_port_number

# 检查端口是否可用
netstat -an | grep LISTEN | grep ":$PORT" > /dev/null

if [ $? -ne 0 ]; then
    # 端口不可用，重新启动Java应用
    echo "Java application not running on port $PORT. Restarting..."
    java $JAVA_OPTS -jar "$JAR_FILE" &
else
    echo "Java application is running on port $PORT."
fi
```

##   if

```shell
echo "001 name" | awk -F'[ :]+' '{print $1}'
echo "id:001 name" | awk -F'id:| ' '{print $2}'
line="id:001 gid:xp0"
id=$(echo $line | sed 's/.*id:\(.*\)/\1/')   #001 gid:xp0
sed 's/id:\([^ ]*\)/id:/'  #001

# Perl表达式
echo "hello id：001 name：888" | grep -oP 'id:\K\d+' #001
echo "hello id:001 name:888" | tr -s ' ' '\t' | cut -d':' -f2 |tr -s ' ' | cut -d$'\t' -f2 # name
```