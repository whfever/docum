



## Shell

### 文件命令

- ![image-20230531184159027](https://article.biliimg.com/bfs/article/9d9160bc785baf5cc04fb28c08fa7445ca77a5a2.png)

### 操作命令

- ![image-20230531183254251](https://article.biliimg.com/bfs/article/2d63cadaa1b00351870c963c0ff2afdab261cd79.png)



### 变量

- 常规变量
  - ![image-20230531185443338](https://article.biliimg.com/bfs/article/9040bea85433b0b5e4379e7252cc1de4c3543646.png)

- 特殊变量
  - ![image-20230531190628634](https://article.biliimg.com/bfs/article/6789fc9cfa2e036bc57d7cde3e39885477d18b92.png)



## 运维

### 进程

![image-20230531192259087](https://article.biliimg.com/bfs/article/29ae03d1ae6858335f0cb54c8b3f34e11b38cff5.png)





### 日志

![image-20230531193302419](https://article.biliimg.com/bfs/article/700865d1b219590af344e4f9d304a17cc8cbe9f0.png)

```shell


x = 'cat  hive.log  | grep 100'
echo $x
```





### 集群

#### hdfs/hive/yarn

![image-20230601160133369](https://article.biliimg.com/bfs/article/496dc9643124f27f51e2bd5c55ce1ee4f07aaa15.png)

#### 节点服役

> 扩充集群

1. Add Host
2. Yarn   配置角色类型  NodeManager
3. Hdfs   DataNode
   1. extend  Balancer



#### 节点退役

Hdfs 

```
》》 exclude host：19.2.1.0

》》stop》》delete
```

### Cloudera Manage

#### 告警配置





#### 统一配置

![image-20230601170745914](https://article.biliimg.com/bfs/article/4696b1d1b67141e12bacba78d0a931cd6670f0fa.png)



### 资源管理

#### 存储>内存> 网络

```shell



df -h
du -h -s /* | grep 'G'


curl  www.baidu.com
vim /etc/hosts
```



### 环境配置

![image-20230601181527426](https://article.biliimg.com/bfs/article/985bfedb5c231cb4704211711d9701b69b6634cc.png)





## Prac

### 自动安装脚本

![image-20230601191048965](https://article.biliimg.com/bfs/article/97d85a1a9293776fcd69c2226fa791aa6989107b.png)

![image-20230601191002416](https://article.biliimg.com/bfs/article/475b79b3aebcdcb2945ad48ae0a9b97de5cbb516.png)

![image-20230601192204600](https://article.biliimg.com/bfs/article/c0e71191a355d8ed5e4240afd389dbb4a748187d.png)

```
x="cat test"
$x
hosts=`cat test| awk '{print $2}'`
for host in $hosts
```

![image-20230601192527133](https://article.biliimg.com/bfs/article/1a096c09a87cc29f3cf8de7e7ae614179ae068e0.png)

```shell
基于面试时考察：用一行命令，杀掉我想要干掉的某个服务
pid='ps -ef grep NodeManager grep -v 'grep'awk '[print $2)'
kill -9 spid
kiL9`psef9 re NodeManager9 rep-v Fgrep讽r相数：Vip994979
议
9949
#NodeManager
program=$1
psInfoScript="ps flgrep s{program}lgrep -v 'grep'"
hosts='cat /etc/hosts grep 172 awk '{orint $2)'grep -v 'hadoop104'grep -v 'hadoop105'
for host in shosts
do
echo
echo"
执行代码的当前host:sthost)"
remotePid='ssh root s(host}$psInfoScript awk '{print $2)'
echo"你要杀的进程PID:s{reotePid)”
#echo"占用了以下端口：
#ssh root s(host}"ss -nltp grep sremotePid"awk '{print $4)'awk -F ':'(print $2)'
ssh root sthost}"kill -9 sremotePid"
if【s?=e】:then
ccho"任务执行成功"
else
echo"任务执行失败
fi
done
```

