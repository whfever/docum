# 工作经验分享：Spark调优【效率提升！建议收藏】

2023年06月22日 15:13160阅读 · 8喜欢 · 0评论

[![img](https://i1.hdslb.com/bfs/face/785b7d261cd9fe23c0ee0da3915ba9310e77c42a.jpg@96w_96h_1c_1s_!web-avatar.webp)](https://space.bilibili.com/621235564)

[涤生大数据](https://space.bilibili.com/621235564)

粉丝：3.5万文章：31

已关注

## **1.优化背景**



业务数据不断增大, Spark运行时间越来越长, 从最初的半小时到6个多小时

某日Spark程序运行6.5个小时后, 报“Too large frame...”的异常：

org.apache.spark.shuffle.FetchFailedException: Too large frame: 2624680416

**2.原因分析**



## **2.1 抛出异常的原因**

```shell
Spark uses custom frame decoder
(TransportFrameDecoder) which does not support frames larger than 2G.
This lead to fails when shuffling using large partitions.
```

根本原因: 源数据的某一列（或某几列）分布不均匀,当某个shuffle操作是根据此列数据进行shuffle时，就会造成整个数据集发生倾斜，即某些partition包含了大量数据，超出了2G的限制。

异常，就是发生在业务数据处理的最后一步left join操作。

 

## **2.2 临时解决方案**

增大partition数, 让partition中的数据量<2g

由于是left join触发了shuffle操作, 而spark默认join时的分区数为200(即spark.sql.shuffle.partitions=200), 所以增大这个分区数, 即调整该参数为800, 即spark.sql.shuffle.partitions=800

## **2.3 临时解决方案**

Spark不再报错,而且“艰难”的跑完了, 跑了近6个小时!

通过Spark UI页面的监控发现, 由于数据倾斜导致, 整个Spark任务的运行时间是被少数的几个Task “拖累的”。

![图片](https://i0.hdslb.com/bfs/article/e0e89168d635918d5071ff7a27bebc0809df3afc.png@1256w_!web-article-pic.webp)



## **3.思考优化**

### **3.1 确认数据倾斜**

方法一: 通过sample算子对DataSet/DataFrame/RDD进行采样, 找出top n的key值及数量；

方法二: 源数据/中间数据落到存储中(如HIVE), 直接查询观察。

### **3.2 可选优化方法**

### **1.HIVE ETL 数据预处理**

把数据倾斜提前到 HIVE ETL中, 避免Spark发生数据倾斜

这个其实很有用，如果不是自己负责，请友好提醒你的上游负责同事

![图片](https://i0.hdslb.com/bfs/article/495eed10292cfdca96234827ba23e74bb288db64.jpg@1256w_!web-article-pic.webp)



### **2.过滤无效的数据 (where / filter)**

a.NULL值数据

b.“脏数据”(非法数据)

c.业务无关的数据

无效数据需要结合自己负责的业务和场景去判断哈，慎重处理业务的无关数据！！以免造成客户Bug

### **3.分析join操作, 左右表的特征, 判断是否可以进行小表广播 broadcast**

a.这样可避免shuffle操作，特别是当大表特别大;

b.默认情况下, join时候, 如果表的数据量低于;

spark.sql.autoBroadcastJoinThreshold参数值时(默认值为10 MB), spark会自动进行broadcast, 但也可以通过强制手动指定广播;

```shell
visitor_df.join(broadcast(campaign_df), Seq("random_bucket", "uuid", "time_range"), "left_outer")
```



业务数据量是100MB

c.Driver上有一个campaign_df全量的副本, 每个Executor上也会有一个campaign_df的副本;

d.JOIN操作, Spark默认都会进行 merge_sort (也需要避免倾斜)。

### **4.数据打散, 扩容join**

分散倾斜的数据, 给key加上随机数前缀

```shell
A.join(B)
```



![图片](https://i0.hdslb.com/bfs/article/84f8c36b7c8eddacfec2069c1d08a5ffbd2e01c8.png@1256w_!web-article-pic.webp)

1）提高shuffle操作并行度

```shell
spark.sql.shuffle.partitions
```

2）多阶段

aggregate操作: 先局部聚合, 再全局聚合；

给key打随机值, 如打上1-10, 先分别针对10个组做聚合；

最后再统一聚合；

join操作: 切成多个部分, 分开join, 最后union。

判断出造成数据倾斜的一些key值 (可通过观察或者sample取样)；

如主号，单独拎出来上述key值的记录做join, 剩余记录再做join；

独立做优化, 如broadcast；

结果数据union即可。

**3.3 实际优化方法**

```shell
a.HIVE 预处理
b.过滤无效的数据
c.broadcast
d.打散 --> 随机数
f.shuffle 并行度
```

示例：

```clike
visitor_leads_fans_df.repartition($codeholder_5amp;quot;random_index")
     .join(broadcast(campaign_df), Seq("random_bucket", "uuid", "time_range"), "left_outer")
     .drop("random_bucket", "random_index")
```



**4.优化后效果**

```shell
1.业务处理中存在复杂的多表关联和计算逻辑（原始数据达百亿数量级）
2.优化后，spark计算性能提升了约12倍(6h-->30min)
3.最终，业务的性能瓶颈存在于ES写入（计算结果，ES索引document数约为21亿 pri.store.size约 300gb）
```

### 优化处理思维导图

![图片](https://i0.hdslb.com/bfs/article/313d595eaaef57eda3e5cccfaea5b5d5feb1c858.png@1256w_!web-article-pic.webp)



**文末给大家分享一份经典的Spark性能优化文档，该文从开发、资源、数据倾斜和shuffle阶段四个角度分析了如何优化Spark程序，真实实用！**

```shell
链接：https://pan.baidu.com/s/1FsDx7foxo4tpQsrYx8dl5A
提取码：dsbg
```

![图片](https://i0.hdslb.com/bfs/article/6ff85b49ab4013de9714c1a69e6cacafe3cdb32d.png@1256w_!web-article-pic.webp)

![图片](https://i0.hdslb.com/bfs/article/c89e2480c370272197aa3a532663770475e6c295.png@1256w_!web-article-pic.webp)