# RocketMq  Interview

## 基础

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_1-为什么要使用消息队列呢)1.为什么要使用消息队列呢？

消息队列主要有三大用途，我们拿一个电商系统的下单举例：

- **解耦**：引入消息队列之前，下单完成之后，需要订单服务去调用库存服务减库存，调用营销服务加营销数据……引入消息队列之后，可以把订单完成的消息丢进队列里，下游服务自己去调用就行了，这样就完成了订单服务和其它服务的解耦合。

![消息队列解耦](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-dd332b3f-d5e5-41bc-813a-9f612e582255.jpg)消息队列解耦

- **异步**：订单支付之后，我们要扣减库存、增加积分、发送消息等等，这样一来这个链路就长了，链路一长，响应时间就变长了。引入消息队列，除了`更新订单状态`，其它的都可以**异步**去做，这样一来就来，就能降低响应时间。

![消息队列异步](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-40d8f782-08cf-48c4-98b2-cb125d287e93.jpg)消息队列异步

- **削峰**：消息队列合一用来削峰，例如秒杀系统，平时流量很低，但是要做秒杀活动，秒杀的时候流量疯狂怼进来，我们的服务器，Redis，MySQL各自的承受能力都不一样，直接全部流量照单全收肯定有问题啊，严重点可能直接打挂了。

我们可以把请求扔到队列里面，只放出我们服务能处理的流量，这样就能抗住短时间的大流量了。

![消息队列削峰](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-f028cb0c-b1a3-47ef-b290-f7d6f46512fb.jpg)消息队列削峰

解耦、异步、削峰，是消息队列最主要的三大作用。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_2-为什么要选择rocketmq)2.为什么要选择RocketMQ?

市场上几大消息队列对比如下：

![四大消息队列对比](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-c3493e70-67c7-4f0d-bb99-f0fe8074c807.jpg)四大消息队列对比

**总结一下**：

选择中间件的可以从这些维度来考虑：可靠性，性能，功能，可运维行，可拓展性，社区活跃度。目前常用的几个中间件，ActiveMQ作为“老古董”，市面上用的已经不多，其它几种：

- RabbitMQ：
- 优点：轻量，迅捷，容易部署和使用，拥有灵活的路由配置
- 缺点：性能和吞吐量不太理想，不易进行二次开发
- RocketMQ：
- 优点：性能好，高吞吐量，稳定可靠，有活跃的中文社区
- 缺点：兼容性上不是太好
- Kafka：
- 优点：拥有强大的性能及吞吐量，兼容性很好
- 缺点：由于“攒一波再处理”导致延迟比较高

我们的系统是面向用户的C端系统，具有一定的并发量，对性能也有比较高的要求，所以选择了低延迟、吞吐量比较高，可用性比较好的RocketMQ。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_3-rocketmq有什么优缺点)3.RocketMQ有什么优缺点？

RocketMQ优点：

- 单机吞吐量：十万级
- 可用性：非常高，分布式架构
- 消息可靠性：经过参数优化配置，消息可以做到0丢失
- 功能支持：MQ功能较为完善，还是分布式的，扩展性好
- 支持10亿级别的消息堆积，不会因为堆积导致性能下降
- 源码是Java，方便结合公司自己的业务二次开发
- 天生为金融互联网领域而生，对于可靠性要求很高的场景，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况
- **RoketMQ**在稳定性上可能更值得信赖，这些业务场景在阿里双11已经经历了多次考验，如果你的业务有上述并发场景，建议可以选择**RocketMQ**

RocketMQ缺点：

- 支持的客户端语言不多，目前是Java及c++，其中c++不成熟
- 没有在 MQ核心中去实现**JMS**等接口，有些系统要迁移需要修改大量代码

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_4-消息队列有哪些消息模型)4.消息队列有哪些消息模型？

消息队列有两种模型：**队列模型**和**发布/订阅模型**。

- **队列模型**

这是最初的一种消息队列模型，对应着消息队列“发-存-收”的模型。生产者往某个队列里面发送消息，一个队列可以存储多个生产者的消息，一个队列也可以有多个消费者，但是消费者之间是竞争关系，也就是说每条消息只能被一个消费者消费。

![队列模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-d94a9bf9-3fed-40a6-8aef-0d0395b6e409.jpg)队列模型

- **发布/订阅模型**

如果需要将一份消息数据分发给多个消费者，并且每个消费者都要求收到全量的消息。很显然，队列模型无法满足这个需求。解决的方式就是发布/订阅模型。

在发布 - 订阅模型中，消息的发送方称为发布者（Publisher），消息的接收方称为订阅者（Subscriber），服务端存放消息的容器称为主题（Topic）。发布者将消息发送到主题中，订阅者在接收消息之前需要先“订阅主题”。“订阅”在这里既是一个动作，同时还可以认为是主题在消费时的一个逻辑副本，每份订阅中，订阅者都可以接收到主题的所有消息。

![发布-订阅模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-692ec6a0-8499-4de2-be17-a577996bdaef.jpg)发布-订阅模型

它和 “队列模式” 的异同：生产者就是发布者，队列就是主题，消费者就是订阅者，无本质区别。唯一的不同点在于：一份消息数据是否可以被多次消费。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_5-那rocketmq的消息模型呢)5.那RocketMQ的消息模型呢？

RocketMQ使用的消息模型是标准的发布-订阅模型，在RocketMQ的术语表中，生产者、消费者和主题，与发布-订阅模型中的概念是完全一样的。

RocketMQ本身的消息是由下面几部分组成：

![RocketMQ消息的组成](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-4ab7f942-23d7-4462-8e36-e305cc0a045f.jpg)RocketMQ消息的组成

- **Message**

**Message**（消息）就是要传输的信息。

一条消息必须有一个主题（Topic），主题可以看做是你的信件要邮寄的地址。

一条消息也可以拥有一个可选的标签（Tag）和额处的键值对，它们可以用于设置一个业务 Key 并在 Broker 上查找此消息以便在开发期间查找问题。

- **Topic**

**Topic**（主题）可以看做消息的归类，它是消息的第一级类型。比如一个电商系统可以分为：交易消息、物流消息等，一条消息必须有一个 Topic 。

**Topic** 与生产者和消费者的关系非常松散，一个 Topic 可以有0个、1个、多个生产者向其发送消息，一个生产者也可以同时向不同的 Topic 发送消息。

一个 Topic 也可以被 0个、1个、多个消费者订阅。

- **Tag**

**Tag**（标签）可以看作子主题，它是消息的第二级类型，用于为用户提供额外的灵活性。使用标签，同一业务模块不同目的的消息就可以用相同 Topic 而不同的 **Tag** 来标识。比如交易消息又可以分为：交易创建消息、交易完成消息等，一条消息可以没有 **Tag** 。

标签有助于保持你的代码干净和连贯，并且还可以为 **RocketMQ** 提供的查询系统提供帮助。

- **Group**

RocketMQ中，订阅者的概念是通过消费组（Consumer Group）来体现的。每个消费组都消费主题中一份完整的消息，不同消费组之间消费进度彼此不受影响，也就是说，一条消息被Consumer Group1消费过，也会再给Consumer Group2消费。

消费组中包含多个消费者，同一个组内的消费者是竞争消费的关系，每个消费者负责消费组内的一部分消息。默认情况，如果一条消息被消费者Consumer1消费了，那同组的其他消费者就不会再收到这条消息。

- **Message Queue**

**Message Queue**（消息队列），一个 Topic 下可以设置多个消息队列，Topic 包括多个 Message Queue ，如果一个 Consumer 需要获取 Topic下所有的消息，就要遍历所有的 Message Queue。

RocketMQ还有一些其它的Queue——例如ConsumerQueue。

- **Offset**

在Topic的消费过程中，由于消息需要被不同的组进行多次消费，所以消费完的消息并不会立即被删除，这就需要RocketMQ为每个消费组在每个队列上维护一个消费位置（Consumer Offset），这个位置之前的消息都被消费过，之后的消息都没有被消费过，每成功消费一条消息，消费位置就加一。

也可以这么说，`Queue` 是一个长度无限的数组，**Offset** 就是下标。

RocketMQ的消息模型中，这些就是比较关键的概念了。画张图总结一下：

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-e470b972-f4ac-4b76-bcde-0df5d4765ca7.jpg)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_6-消息的消费模式了解吗)6.消息的消费模式了解吗？

消息消费模式有两种：**Clustering**（集群消费）和**Broadcasting**（广播消费）。

![两种消费模式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-2c574635-1eaa-4bdd-8aa8-1bdc3f10b274.jpg)两种消费模式

默认情况下就是集群消费，这种模式下`一个消费者组共同消费一个主题的多个队列，一个队列只会被一个消费者消费`，如果某个消费者挂掉，分组内其它消费者会接替挂掉的消费者继续消费。

而广播消费消息会发给消费者组中的每一个消费者进行消费。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_7-roctetmq基本架构了解吗)7.RoctetMQ基本架构了解吗？

先看图，RocketMQ的基本架构：

![RocketMQ架构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-d4c0e036-0f0e-466f-bd4b-7e6ee10daca4.jpg)RocketMQ架构

RocketMQ 一共有四个部分组成：NameServer，Broker，Producer 生产者，Consumer 消费者，它们对应了：发现、发、存、收，为了保证高可用，一般每一部分都是集群部署的。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_8-那能介绍一下这四部分吗)8.那能介绍一下这四部分吗？

类比一下我们生活的邮政系统——

邮政系统要正常运行，离不开下面这四个角色， 一是发信者，二 是收信者， 三是负责暂存传输的邮局， 四是负责协调各个地方邮局的管理机构。对应到 RocketMQ 中，这四个角色就是 Producer、 Consumer、 Broker 、NameServer。

![RocketMQ类比邮政体系](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-00175355-5532-4ee6-a48c-e3e3a9b87d64.jpg)RocketMQ类比邮政体系

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#nameserver)NameServer

NameServer 是一个无状态的服务器，角色类似于 Kafka使用的 Zookeeper，但比 Zookeeper 更轻量。

特点：

- 每个 NameServer 结点之间是相互独立，彼此没有任何信息交互。
- Nameserver 被设计成几乎是无状态的，通过部署多个结点来标识自己是一个伪集群，Producer 在发送消息前从 NameServer 中获取 Topic 的路由信息也就是发往哪个 Broker，Consumer 也会定时从 NameServer 获取 Topic 的路由信息，Broker 在启动时会向 NameServer 注册，并定时进行心跳连接，且定时同步维护的 Topic 到 NameServer。

功能主要有两个：

- 1、和Broker 结点保持长连接。
- 2、维护 Topic 的路由信息。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#broker)Broker

消息存储和中转角色，负责存储和转发消息。

- Broker 内部维护着一个个 Consumer Queue，用来存储消息的索引，真正存储消息的地方是 CommitLog（日志文件）。

![RocketMQ存储-图片来源官网](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-789379a9-4a0c-4992-9de1-e49283d089a4.jpg)RocketMQ存储-图片来源官网

- 单个 Broker 与所有的 Nameserver 保持着长连接和心跳，并会定时将 Topic 信息同步到 NameServer，和 NameServer 的通信底层是通过 Netty 实现的。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#producer)Producer

消息生产者，业务端负责发送消息，由用户自行实现和分布式部署。

- **Producer**由用户进行分布式部署，消息由**Producer**通过多种负载均衡模式发送到**Broker**集群，发送低延时，支持快速失败。
- **RocketMQ** 提供了三种方式发送消息：同步、异步和单向
- **同步发送**：同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。一般用于重要通知消息，例如重要通知邮件、营销短信。
- **异步发送**：异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景，例如用户视频上传后通知启动转码服务。
- **单向发送**：单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#consumer)Consumer

消息消费者，负责消费消息，一般是后台系统负责异步消费。

- **Consumer**也由用户部署，支持PUSH和PULL两种消费模式，支持**集群消费**和**广播消费**，提供**实时的消息订阅机制**。
- **Pull**：拉取型消费者（Pull Consumer）主动从消息服务器拉取信息，只要批量拉取到消息，用户应用就会启动消费过程，所以 Pull 称为主动消费型。
- **Push**：推送型消费者（Push Consumer）封装了消息的拉取、消费进度和其他的内部维护工作，将消息到达时执行的回调接口留给用户应用程序来实现。所以 Push 称为被动消费类型，但其实从实现上看还是从消息服务器中拉取消息，不同于 Pull 的是 Push 首先要注册消费监听器，当监听器处触发后才开始消费消息。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#进阶)进阶

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_9-如何保证消息的可用性-可靠性-不丢失呢)9.如何保证消息的可用性/可靠性/不丢失呢？

消息可能在哪些阶段丢失呢？可能会在这三个阶段发生丢失：生产阶段、存储阶段、消费阶段。

所以要从这三个阶段考虑：

![消息传递三阶段](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-692a920d-621f-4b36-87f8-edb53e7f5cd9.jpg)消息传递三阶段

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#生产)生产

在生产阶段，主要**通过请求确认机制，来保证消息的可靠传递**。

- 1、同步发送的时候，要注意处理响应结果和异常。如果返回响应OK，表示消息成功发送到了Broker，如果响应失败，或者发生其它异常，都应该重试。
- 2、异步发送的时候，应该在回调方法里检查，如果发送失败或者异常，都应该进行重试。
- 3、如果发生超时的情况，也可以通过查询日志的API，来检查是否在Broker存储成功。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#存储)存储

存储阶段，可以通过**配置可靠性优先的 Broker 参数来避免因为宕机丢消息**，简单说就是可靠性优先的场景都应该使用同步。

- 1、消息只要持久化到CommitLog（日志文件）中，即使Broker宕机，未消费的消息也能重新恢复再消费。
- 2、Broker的刷盘机制：同步刷盘和异步刷盘，不管哪种刷盘都可以保证消息一定存储在pagecache中（内存中），但是同步刷盘更可靠，它是Producer发送消息后等数据持久化到磁盘之后再返回响应给Producer。

![同步刷盘和异步刷盘-图片来源官网](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-61a3b092-1329-4738-bef9-6eb7ae429c26.jpg)同步刷盘和异步刷盘-图片来源官网

- 3、Broker通过主从模式来保证高可用，Broker支持Master和Slave同步复制、Master和Slave异步复制模式，生产者的消息都是发送给Master，但是消费既可以从Master消费，也可以从Slave消费。同步复制模式可以保证即使Master宕机，消息肯定在Slave中有备份，保证了消息不会丢失。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#消费)消费

从Consumer角度分析，如何保证消息被成功消费？

- Consumer保证消息成功消费的关键在于确认的时机，不要在收到消息后就立即发送消费确认，而是应该在执行完所有消费业务逻辑之后，再发送消费确认。因为消息队列维护了消费的位置，逻辑执行失败了，没有确认，再去队列拉取消息，就还是之前的一条。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_10-如何处理消息重复的问题呢)10.如何处理消息重复的问题呢？

对分布式消息队列来说，同时做到确保一定投递和不重复投递是很难的，就是所谓的“有且仅有一次” 。RocketMQ择了确保一定投递，保证消息不丢失，但有可能造成消息重复。

处理消息重复问题，主要有业务端自己保证，主要的方式有两种：**业务幂等**和**消息去重**。

![消息重复处理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-05c0538b-abfa-4bb0-973f-6b92555a6e5b.jpg)消息重复处理

**业务幂等**：第一种是保证消费逻辑的幂等性，也就是多次调用和一次调用的效果是一样的。这样一来，不管消息消费多少次，对业务都没有影响。

**消息去重**：第二种是业务端，对重复的消息就不再消费了。这种方法，需要保证每条消息都有一个惟一的编号，通常是业务相关的，比如订单号，消费的记录需要落库，而且需要保证和消息确认这一步的原子性。

具体做法是可以建立一个消费记录表，拿到这个消息做数据库的insert操作。给这个消息做一个唯一主键（primary key）或者唯一约束，那么就算出现重复消费的情况，就会导致主键冲突，那么就不再处理这条消息。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_11-怎么处理消息积压)11.怎么处理消息积压？

发生了消息积压，这时候就得想办法赶紧把积压的消息消费完，就得考虑提高消费能力，一般有两种办法：

![消息积压处理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-5d1ea064-1a37-4746-ad26-a18a1c8c344e.jpg)消息积压处理

- **消费者扩容**：如果当前Topic的Message Queue的数量大于消费者数量，就可以对消费者进行扩容，增加消费者，来提高消费能力，尽快把积压的消息消费玩。
- **消息迁移Queue扩容**：如果当前Topic的Message Queue的数量小于或者等于消费者数量，这种情况，再扩容消费者就没什么用，就得考虑扩容Message Queue。可以新建一个临时的Topic，临时的Topic多设置一些Message Queue，然后先用一些消费者把消费的数据丢到临时的Topic，因为不用业务处理，只是转发一下消息，还是很快的。接下来用扩容的消费者去消费新的Topic里的数据，消费完了之后，恢复原状。

![消息迁移扩容消费](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-69aa8004-45e9-4e41-b628-1d7ed6d94c92.jpg)消息迁移扩容消费

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_12-顺序消息如何实现)12.顺序消息如何实现？

顺序消息是指消息的消费顺序和产生顺序相同，在有些业务逻辑下，必须保证顺序，比如订单的生成、付款、发货，这个消息必须按顺序处理才行。

![顺序消息](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-e3de6bb5-b5db-47af-8ae3-73aedd269f32.jpg)顺序消息

顺序消息分为全局顺序消息和部分顺序消息，全局顺序消息指某个 Topic 下的所有消息都要保证顺序；

部分顺序消息只要保证每一组消息被顺序消费即可，比如订单消息，只要保证同一个订单 ID 个消息能按顺序消费即可。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#部分顺序消息)部分顺序消息

部分顺序消息相对比较好实现，生产端需要做到把同 ID 的消息发送到同一个 Message Queue ；在消费过程中，要做到从同一个Message Queue读取的消息顺序处理——消费端不能并发处理顺序消息，这样才能达到部分有序。

![部分顺序消息](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-14ab3700-8538-473e-bb66-8acfdd6a77a2.jpg)部分顺序消息

发送端使用 MessageQueueSelector 类来控制 把消息发往哪个 Message Queue 。

![顺序消息生产-例子来源官方](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-4aad853e-4dd7-4cde-b28d-b6e74ca9331f.jpg)顺序消息生产-例子来源官方

消费端通过使用 MessageListenerOrderly 来解决单 Message Queue 的消息被并发处理的问题。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-f233ab8e-87f0-4b64-91eb-d4d9ec71ab3e.jpg)

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#全局顺序消息)全局顺序消息

RocketMQ 默认情况下不保证顺序，比如创建一个 Topic ，默认八个写队列，八个读队列，这时候一条消息可能被写入任意一个队列里；在数据的读取过程中，可能有多个 Consumer ，每个 Consumer 也可能启动多个线程并行处理，所以消息被哪个 Consumer 消费，被消费的顺序和写人的顺序是否一致是不确定的。

要保证全局顺序消息， 需要先把 Topic 的读写队列数设置为 一，然后Producer Consumer 的并发设置，也要是一。简单来说，为了保证整个 Topic全局消息有序，只能消除所有的并发处理，各部分都设置成单线程处理 ，这时候就完全牺牲RocketMQ的高并发、高吞吐的特性了。

![全局顺序消息](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-8e98ac61-ad47-4ed4-aac6-223201f9aae2.jpg)全局顺序消息

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_13-如何实现消息过滤)13.如何实现消息过滤？

有两种方案：

- 一种是在 Broker 端按照 Consumer 的去重逻辑进行过滤，这样做的好处是避免了无用的消息传输到 Consumer 端，缺点是加重了 Broker 的负担，实现起来相对复杂。
- 另一种是在 Consumer 端过滤，比如按照消息设置的 tag 去重，这样的好处是实现起来简单，缺点是有大量无用的消息到达了 Consumer 端只能丢弃不处理。

一般采用Cosumer端过滤，如果希望提高吞吐量，可以采用Broker过滤。

对消息的过滤有三种方式：

![消息过滤](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-f2c8bf50-dc51-44c8-9d71-b8a22af199c4.jpg)消息过滤

- 根据Tag过滤：这是最常见的一种，用起来高效简单



```text
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_EXAMPLE");
consumer.subscribe("TOPIC", "TAGA || TAGB || TAGC");
```

- SQL 表达式过滤：SQL表达式过滤更加灵活



```text
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_4");
// 只有订阅的消息有这个属性a, a >=0 and a <= 3
consumer.subscribe("TopicTest", MessageSelector.bySql("a between 0 and 3");
consumer.registerMessageListener(new MessageListenerConcurrently() {
   @Override
   public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
       return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
   }
});
consumer.start();
```

- Filter Server 方式：最灵活，也是最复杂的一种方式，允许用户自定义函数进行过滤

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_14-延时消息了解吗)14.延时消息了解吗？

电商的订单超时自动取消，就是一个典型的利用延时消息的例子，用户提交了一个订单，就可以发送一个延时消息，1h后去检查这个订单的状态，如果还是未付款就取消订单释放库存。

RocketMQ是支持延时消息的，只需要在生产消息的时候设置消息的延时级别：



```text
// 实例化一个生产者来产生延时消息
DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
// 启动生产者
producer.start();
int totalMessagesToSend = 100;
for (int i = 0; i < totalMessagesToSend; i++) {
    Message message = new Message("TestTopic", ("Hello scheduled message " + i).getBytes());
    // 设置延时等级3,这个消息将在10s之后发送(现在只支持固定的几个时间,详看delayTimeLevel)
    message.setDelayTimeLevel(3);
    // 发送消息
    producer.send(message);
}
```

但是目前RocketMQ支持的延时级别是有限的：



```text
private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
```

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#rocketmq怎么实现延时消息的)RocketMQ怎么实现延时消息的？

简单，八个字：`临时存储`+`定时任务`。

Broker收到延时消息了，会先发送到主题（SCHEDULE_TOPIC_XXXX）的相应时间段的Message Queue中，然后通过一个定时任务轮询这些队列，到期后，把消息投递到目标Topic的队列中，然后消费者就可以正常消费这些消息。

![延迟消息处理流程-图片来源见水印](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-e3b68480-8006-4cd6-892a-1c72f8b0fbcb.jpg)延迟消息处理流程-图片来源见水印

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_15-怎么实现分布式消息事务的-半消息)15.怎么实现分布式消息事务的？半消息？

半消息：是指暂时还不能被 Consumer 消费的消息，Producer 成功发送到 Broker 端的消息，但是此消息被标记为 “暂不可投递” 状态，只有等 Producer 端执行完本地事务后经过二次确认了之后，Consumer 才能消费此条消息。

依赖半消息，可以实现分布式消息事务，其中的关键在于二次确认以及消息回查：

![RocketMQ实现消息事务](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-76df2fb9-0f3f-496d-88a8-aac79ad1102c.jpg)RocketMQ实现消息事务

- 1、Producer 向 broker 发送半消息
- 2、Producer 端收到响应，消息发送成功，此时消息是半消息，标记为 “不可投递” 状态，Consumer 消费不了。
- 3、Producer 端执行本地事务。
- 4、正常情况本地事务执行完成，Producer 向 Broker 发送 Commit/Rollback，如果是 Commit，Broker 端将半消息标记为正常消息，Consumer 可以消费，如果是 Rollback，Broker 丢弃此消息。
- 5、异常情况，Broker 端迟迟等不到二次确认。在一定时间后，会查询所有的半消息，然后到 Producer 端查询半消息的执行情况。
- 6、Producer 端查询本地事务的状态
- 7、根据事务的状态提交 commit/rollback 到 broker 端。（5，6，7 是消息回查）
- 8、消费者段消费到消息之后，执行本地事务，执行本地事务。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_16-死信队列知道吗)16.死信队列知道吗？

死信队列用于处理无法被正常消费的消息，即死信消息。

当一条消息初次消费失败，**消息队列 RocketMQ 会自动进行消息重试**；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列 RocketMQ 不会立刻将消息丢弃，而是将其发送到该**消费者对应的特殊队列中**，该特殊队列称为**死信队列**。

**死信消息的特点**：

- 不会再被消费者正常消费。
- 有效期与正常消息相同，均为 3 天，3 天后会被自动删除。因此，需要在死信消息产生后的 3 天内及时处理。

**死信队列的特点**：

- 一个死信队列对应一个 Group ID， 而不是对应单个消费者实例。
- 如果一个 Group ID 未产生死信消息，消息队列 RocketMQ 不会为其创建相应的死信队列。
- 一个死信队列包含了对应 Group ID 产生的所有死信消息，不论该消息属于哪个 Topic。

RocketMQ 控制台提供对死信消息的查询、导出和重发的功能。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_17-如何保证rocketmq的高可用)17.如何保证RocketMQ的高可用？

NameServer因为是无状态，且不相互通信的，所以只要集群部署就可以保证高可用。

![NameServer集群](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-0ce789f4-7a47-4c24-ac08-76f0490298f7.jpg)NameServer集群

RocketMQ的高可用主要是在体现在Broker的读和写的高可用，Broker的高可用是通过`集群`和`主从`实现的。

![Broker集群、主从示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-76c5eb61-9605-4620-84fb-dc960f01de85.jpg)Broker集群、主从示意图

Broker可以配置两种角色：Master和Slave，Master角色的Broker支持读和写，Slave角色的Broker只支持读，Master会向Slave同步消息。

也就是说Producer只能向Master角色的Broker写入消息，Cosumer可以从Master和Slave角色的Broker读取消息。

Consumer 的配置文件中，并不需要设置是从 Master 读还是从 Slave读，当 Master 不可用或者繁忙的时候， Consumer 的读请求会被自动切换到从 Slave。有了自动切换 Consumer 这种机制，当一个 Master 角色的机器出现故障后，Consumer 仍然可以从 Slave 读取消息，不影响 Consumer 读取消息，这就实现了读的高可用。

如何达到发送端写的高可用性呢？在创建 Topic 的时候，把 Topic 的多个Message Queue 创建在多个 Broker 组上（相同 Broker 名称，不同 brokerId机器组成 Broker 组），这样当 Broker 组的 Master 不可用后，其他组Master 仍然可用， Producer 仍然可以发送消息 RocketMQ 目前还不支持把Slave自动转成 Master ，如果机器资源不足，需要把 Slave 转成 Master ，则要手动停止 Slave 色的 Broker ，更改配置文件，用新的配置文件启动 Broker。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#原理)原理

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_18-说一下rocketmq的整体工作流程)18.说一下RocketMQ的整体工作流程？

简单来说，RocketMQ是一个分布式消息队列，也就是`消息队列`+`分布式系统`。

作为消息队列，它是`发`-`存`-`收`的一个模型，对应的就是Producer、Broker、Cosumer；作为分布式系统，它要有服务端、客户端、注册中心，对应的就是Broker、Producer/Consumer、NameServer

所以我们看一下它主要的工作流程：RocketMQ由NameServer注册中心集群、Producer生产者集群、Consumer消费者集群和若干Broker（RocketMQ进程）组成：

1. Broker在启动的时候去向所有的NameServer注册，并保持长连接，每30s发送一次心跳
2. Producer在发送消息的时候从NameServer获取Broker服务器地址，根据负载均衡算法选择一台服务器来发送消息
3. Conusmer消费消息的时候同样从NameServer获取Broker地址，然后主动拉取消息来消费

![RocketMQ整体工作流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-ec571bd4-fa24-4ada-87ab-f761a7dfdf3f.jpg)RocketMQ整体工作流程

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_19-为什么rocketmq不使用zookeeper作为注册中心呢)19.为什么RocketMQ不使用Zookeeper作为注册中心呢？

Kafka我们都知道采用Zookeeper作为注册中心——当然也开始逐渐去Zookeeper，RocketMQ不使用Zookeeper其实主要可能从这几方面来考虑：

1. 基于可用性的考虑，根据CAP理论，同时最多只能满足两个点，而Zookeeper满足的是CP，也就是说Zookeeper并不能保证服务的可用性，Zookeeper在进行选举的时候，整个选举的时间太长，期间整个集群都处于不可用的状态，而这对于一个注册中心来说肯定是不能接受的，作为服务发现来说就应该是为可用性而设计。
2. 基于性能的考虑，NameServer本身的实现非常轻量，而且可以通过增加机器的方式水平扩展，增加集群的抗压能力，而Zookeeper的写是不可扩展的，Zookeeper要解决这个问题只能通过划分领域，划分多个Zookeeper集群来解决，首先操作起来太复杂，其次这样还是又违反了CAP中的A的设计，导致服务之间是不连通的。
3. 持久化的机制来带的问题，ZooKeeper 的 ZAB 协议对每一个写请求，会在每个 ZooKeeper 节点上保持写一个事务日志，同时再加上定期的将内存数据镜像（Snapshot）到磁盘来保证数据的一致性和持久性，而对于一个简单的服务发现的场景来说，这其实没有太大的必要，这个实现方案太重了。而且本身存储的数据应该是高度定制化的。
4. 消息发送应该弱依赖注册中心，而RocketMQ的设计理念也正是基于此，生产者在第一次发送消息的时候从NameServer获取到Broker地址后缓存到本地，如果NameServer整个集群不可用，短时间内对于生产者和消费者并不会产生太大影响。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_20-broker是怎么保存数据的呢)20.Broker是怎么保存数据的呢？

RocketMQ主要的存储文件包括CommitLog文件、ConsumeQueue文件、Indexfile文件。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-b6d13d4d-c417-43b4-bfe1-12724777888c.jpg)

消息存储的整体的设计：

![消息存储整体设计-来源官网](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-ddbf8773-1d71-4d1a-a186-46f6985b621e.jpg)消息存储整体设计-来源官网

- **CommitLog**：消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容,消息内容不是定长的。单个文件大小默认1G, 文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；当第一个文件写满了，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推。消息主要是顺序写入日志文件，当文件满了，写入下一个文件。

CommitLog文件保存于${Rocket_Home}/store/commitlog目录中，从图中我们可以明显看出来文件名的偏移量，每个文件默认1G，写满后自动生成一个新的文件。

![CommitLog](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-a76f7365-8a8c-4b91-9505-9d427cf3bde4.jpg)CommitLog

- **ConsumeQueue**：消息消费队列，引入的目的主要是提高消息消费的性能，由于RocketMQ是基于主题topic的订阅模式，消息消费是针对主题进行的，如果要遍历commitlog文件中根据topic检索消息是非常低效的。

Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引，保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset，消息大小size和消息Tag的HashCode值。

ConsumeQueue文件可以看成是基于Topic的CommitLog索引文件，故ConsumeQueue文件夹的组织方式如下：topic/queue/file三层组织结构，具体存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}。同样ConsumeQueue文件采取定长设计，每一个条目共20个字节，分别为8字节的CommitLog物理偏移量、4字节的消息长度、8字节tag hashcode，单个文件由30W个条目组成，可以像数组一样随机访问每一个条目，每个ConsumeQueue文件大小约5.72M；

![Comsumer Queue](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-c8b22760-35c2-436b-81ed-be49a107357b.jpg)Comsumer Queue

- **IndexFile**：IndexFile（索引文件）提供了一种可以通过key或时间区间来查询消息的方法。Index文件的存储位置是： {fileName}，文件名fileName是以创建时的时间戳命名的，固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引，IndexFile的底层存储设计为在文件系统中实现HashMap结构，故RocketMQ的索引文件其底层实现为hash索引。

![IndexFile文件示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-f06f306b-fd87-48ff-b1cb-e39750d308e7.jpg)IndexFile文件示意图

总结一下：RocketMQ采用的是混合型的存储结构，即为Broker单个实例下所有的队列共用一个日志数据文件（即为CommitLog）来存储。

RocketMQ的混合型存储结构(多个Topic的消息实体内容都存储于一个CommitLog中)针对Producer和Consumer分别采用了数据和索引部分相分离的存储结构，Producer发送消息至Broker端，然后Broker端使用同步或者异步的方式对消息刷盘持久化，保存至CommitLog中。

只要消息被刷盘持久化至磁盘文件CommitLog中，那么Producer发送的消息就不会丢失。正因为如此，Consumer也就肯定有机会去消费这条消息。当无法拉取到消息后，可以等下一次消息拉取，同时服务端也支持长轮询模式，如果一个消息拉取请求未拉取到消息，Broker允许等待30s的时间，只要这段时间内有新消息到达，将直接返回给消费端。

这里，RocketMQ的具体做法是，使用Broker端的后台服务线程—ReputMessageService不停地分发请求并异步构建ConsumeQueue（逻辑消费队列）和IndexFile（索引文件）数据。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-80af9918-2d4b-4b43-b83a-d44bfc0f30bc.jpg)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_21-说说rocketmq怎么对文件进行读写的)21.说说RocketMQ怎么对文件进行读写的？

RocketMQ对文件的读写巧妙地利用了操作系统的一些高效文件读写方式——`PageCache`、`顺序读写`、`零拷贝`。

- PageCache、顺序读取

在RocketMQ中，ConsumeQueue逻辑消费队列存储的数据较少，并且是顺序读取，在page cache机制的预读取作用下，Consume Queue文件的读性能几乎接近读内存，即使在有消息堆积情况下也不会影响性能。而对于CommitLog消息存储的日志数据文件来说，读取消息内容时候会产生较多的随机访问读取，严重影响性能。如果选择合适的系统IO调度算法，比如设置调度算法为“Deadline”（此时块存储采用SSD的话），随机读的性能也会有所提升。

页缓存（PageCache)是OS对文件的缓存，用于加速对文件的读写。一般来说，程序对文件进行顺序读写的速度几乎接近于内存的读写速度，主要原因就是由于OS使用PageCache机制对读写访问操作进行了性能优化，将一部分的内存用作PageCache。对于数据的写入，OS会先写入至Cache内，随后通过异步的方式由pdflush内核线程将Cache内的数据刷盘至物理磁盘上。对于数据的读取，如果一次读取文件时出现未命中PageCache的情况，OS从物理磁盘上访问读取文件的同时，会顺序对其他相邻块的数据文件进行预读取。

- 零拷贝

另外，RocketMQ主要通过MappedByteBuffer对文件进行读写操作。其中，利用了NIO中的FileChannel模型将磁盘上的物理文件直接映射到用户态的内存地址中（这种Mmap的方式减少了传统IO，将磁盘文件数据在操作系统内核地址空间的缓冲区，和用户应用程序地址空间的缓冲区之间来回进行拷贝的性能开销），将对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率（正因为需要使用内存映射机制，故RocketMQ的文件存储都使用定长结构来存储，方便一次将整个文件映射至内存）。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#说说什么是零拷贝)说说什么是零拷贝?

在操作系统中，使用传统的方式，数据需要经历几次拷贝，还要经历用户态/内核态切换。

![传统文件传输示意图-来源《图解操作系统》](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-35fd884c-8d1b-4d04-8f09-23ed2f945b23.jpg)传统文件传输示意图-来源《图解操作系统》

1. 从磁盘复制数据到内核态内存；
2. 从内核态内存复制到用户态内存；
3. 然后从用户态内存复制到网络驱动的内核态内存；
4. 最后是从网络驱动的内核态内存复制到网卡中进行传输。

所以，可以通过零拷贝的方式，**减少用户态与内核态的上下文切换**和**内存拷贝的次数**，用来提升I/O的性能。零拷贝比较常见的实现方式是**mmap**，这种机制在Java中是通过MappedByteBuffer实现的。

![mmap示意图-来源《图解操作系统》](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-3ebab020-e411-4239-b91e-72147190c7b1.jpg)mmap示意图-来源《图解操作系统》

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_22-消息刷盘怎么实现的呢)22.消息刷盘怎么实现的呢？

RocketMQ提供了两种刷盘策略：同步刷盘和异步刷盘

- 同步刷盘：在消息达到Broker的内存之后，必须刷到commitLog日志文件中才算成功，然后返回Producer数据已经发送成功。
- 异步刷盘：异步刷盘是指消息达到Broker内存后就返回Producer数据已经发送成功，会唤醒一个线程去将数据持久化到CommitLog日志文件中。

**Broker** 在消息的存取时直接操作的是内存（内存映射文件），这可以提供系统的吞吐量，但是无法避免机器掉电时数据丢失，所以需要持久化到磁盘中。

刷盘的最终实现都是使用**NIO**中的 MappedByteBuffer.force() 将映射区的数据写入到磁盘，如果是同步刷盘的话，在**Broker**把消息写到**CommitLog**映射区后，就会等待写入完成。

异步而言，只是唤醒对应的线程，不保证执行的时机，流程如图所示。

![异步刷盘](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-10a2361b-5e23-462f-86bf-1a9bce2342e5.jpg)异步刷盘

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_22-能说下-rocketmq-的负载均衡是如何实现的)22.能说下 RocketMQ 的负载均衡是如何实现的？

RocketMQ中的负载均衡都在Client端完成，具体来说的话，主要可以分为Producer端发送消息时候的负载均衡和Consumer端订阅消息的负载均衡。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#producer的负载均衡)Producer的负载均衡

Producer端在发送消息的时候，会先根据Topic找到指定的TopicPublishInfo，在获取了TopicPublishInfo路由信息后，RocketMQ的客户端在默认方式下selectOneMessageQueue()方法会从TopicPublishInfo中的messageQueueList中选择一个队列（MessageQueue）进行发送消息。具这里有一个sendLatencyFaultEnable开关变量，如果开启，在随机递增取模的基础上，再过滤掉not available的Broker代理。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-5f254411-502b-4bd3-89b5-d3157964617b.jpg)

所谓的"latencyFaultTolerance"，是指对之前失败的，按一定的时间做退避。例如，如果上次请求的latency超过550Lms，就退避3000Lms；超过1000L，就退避60000L；如果关闭，采用随机递增取模的方式选择一个队列（MessageQueue）来发送消息，latencyFaultTolerance机制是实现消息发送高可用的核心关键所在。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#consumer的负载均衡)Consumer的负载均衡

在RocketMQ中，Consumer端的两种消费模式（Push/Pull）都是基于拉模式来获取消息的，而在Push模式只是对pull模式的一种封装，其本质实现为消息拉取线程在从服务器拉取到一批消息后，然后提交到消息消费线程池后，又“马不停蹄”的继续向服务器再次尝试拉取消息。如果未拉取到消息，则延迟一下又继续拉取。在两种基于拉模式的消费方式（Push/Pull）中，均需要Consumer端知道从Broker端的哪一个消息队列中去获取消息。因此，有必要在Consumer端来做负载均衡，即Broker端中多个MessageQueue分配给同一个ConsumerGroup中的哪些Consumer消费。

1. Consumer端的心跳包发送

在Consumer启动后，它就会通过定时任务不断地向RocketMQ集群中的所有Broker实例发送心跳包（其中包含了，消息消费分组名称、订阅关系集合、消息通信模式和客户端id的值等信息）。Broker端在收到Consumer的心跳消息后，会将它维护在ConsumerManager的本地缓存变量—consumerTable，同时并将封装后的客户端网络通道信息保存在本地缓存变量—channelInfoTable中，为之后做Consumer端的负载均衡提供可以依据的元数据信息。

1. Consumer端实现负载均衡的核心类—RebalanceImpl

在Consumer实例的启动流程中的启动MQClientInstance实例部分，会完成负载均衡服务线程—RebalanceService的启动（每隔20s执行一次）。

通过查看源码可以发现，RebalanceService线程的run()方法最终调用的是RebalanceImpl类的rebalanceByTopic()方法，这个方法是实现Consumer端负载均衡的核心。

rebalanceByTopic()方法会根据消费者通信类型为“广播模式”还是“集群模式”做不同的逻辑处理。这里主要来看下集群模式下的主要处理流程：

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-7cbfd097-5186-47ba-9641-687bc9381d0b.jpg)

(1) 从rebalanceImpl实例的本地缓存变量—topicSubscribeInfoTable中，获取该Topic主题下的消息消费队列集合（mqSet）；

(2) 根据topic和consumerGroup为参数调用mQClientFactory.findConsumerIdList()方法向Broker端发送通信请求，获取该消费组下消费者Id列表；

(3) 先对Topic下的消息消费队列、消费者Id排序，然后用消息队列分配策略算法（默认为：消息队列的平均分配算法），计算出待拉取的消息队列。这里的平均分配算法，类似于分页的算法，将所有MessageQueue排好序类似于记录，将所有消费端Consumer排好序类似页数，并求出每一页需要包含的平均size和每个页面记录的范围range，最后遍历整个range而计算出当前Consumer端应该分配到的的MessageQueue。

![Cosumer分配](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-6d1c69e7-5245-495f-8f8d-b5e48162df6f.jpg)Cosumer分配

(4) 然后，调用updateProcessQueueTableInRebalance()方法，具体的做法是，先将分配到的消息队列集合（mqSet）与processQueueTable做一个过滤比对。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-c84236ed-77a6-45e9-b086-4927d72ce21a.jpg)

- 上图中processQueueTable标注的红色部分，表示与分配到的消息队列集合mqSet互不包含。将这些队列设置Dropped属性为true，然后查看这些队列是否可以移除出processQueueTable缓存变量，这里具体执行removeUnnecessaryMessageQueue()方法，即每隔1s 查看是否可以获取当前消费处理队列的锁，拿到的话返回true。如果等待1s后，仍然拿不到当前消费处理队列的锁则返回false。如果返回true，则从processQueueTable缓存变量中移除对应的Entry；
- 上图中processQueueTable的绿色部分，表示与分配到的消息队列集合mqSet的交集。判断该ProcessQueue是否已经过期了，在Pull模式的不用管，如果是Push模式的，设置Dropped属性为true，并且调用removeUnnecessaryMessageQueue()方法，像上面一样尝试移除Entry；
- 最后，为过滤后的消息队列集合（mqSet）中的每个MessageQueue创建一个ProcessQueue对象并存入RebalanceImpl的processQueueTable队列中（其中调用RebalanceImpl实例的computePullFromWhere(MessageQueue mq)方法获取该MessageQueue对象的下一个进度消费值offset，随后填充至接下来要创建的pullRequest对象属性中），并创建拉取请求对象—pullRequest添加到拉取列表—pullRequestList中，最后执行dispatchPullRequest()方法，将Pull消息的请求对象PullRequest依次放入PullMessageService服务线程的阻塞队列pullRequestQueue中，待该服务线程取出后向Broker端发起Pull消息的请求。其中，可以重点对比下，RebalancePushImpl和RebalancePullImpl两个实现类的dispatchPullRequest()方法不同，RebalancePullImpl类里面的该方法为空。

消息消费队列在同一消费组不同消费者之间的负载均衡，其核心设计理念是在一个消息消费队列在同一时间只允许被同一消费组内的一个消费者消费，一个消息消费者能同时消费多个消息队列。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/rocketmq.html#_23-rocketmq消息长轮询了解吗)23.RocketMQ消息长轮询了解吗？

所谓的长轮询，就是Consumer 拉取消息，如果对应的 Queue 如果没有数据，Broker 不会立即返回，而是把 PullReuqest hold起来，等待 queue 有了消息后，或者长轮询阻塞时间到了，再重新处理该 queue 上的所有 PullRequest。

![长轮询简单示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/nice-article/weixin-mianznxrocketmqessw-5a715dea-8e18-471e-9a74-e97299901658.jpg)长轮询简单示意图

- PullMessageProcessor#processRequest



```text
//如果没有拉到数据
case ResponseCode.PULL_NOT_FOUND:
// broker 和 consumer 都允许 suspend，默认开启
if (brokerAllowSuspend && hasSuspendFlag) {
    long pollingTimeMills = suspendTimeoutMillisLong;
    if (!this.brokerController.getBrokerConfig().isLongPollingEnable()) {
        pollingTimeMills = this.brokerController.getBrokerConfig().getShortPollingTimeMills();
    }

    String topic = requestHeader.getTopic();
    long offset = requestHeader.getQueueOffset();
    int queueId = requestHeader.getQueueId();
    //封装一个PullRequest
    PullRequest pullRequest = new PullRequest(request, channel, pollingTimeMills,
            this.brokerController.getMessageStore().now(), offset, subscriptionData, messageFilter);
    //把PullRequest挂起来
    this.brokerController.getPullRequestHoldService().suspendPullRequest(topic, queueId, pullRequest);
    response = null;
    break;
}
```

挂起的请求，有一个服务线程会不停地检查，看queue中是否有数据，或者超时。

- PullRequestHoldService#run()



```text
@Override
public void run() {
    log.info("{} service started", this.getServiceName());
    while (!this.isStopped()) {
        try {
            if (this.brokerController.getBrokerConfig().isLongPollingEnable()) {
                this.waitForRunning(5 * 1000);
            } else {
                this.waitForRunning(this.brokerController.getBrokerConfig().getShortPollingTimeMills());
            }

            long beginLockTimestamp = this.systemClock.now();
            //检查hold住的请求
            this.checkHoldRequest();
            long costTime = this.systemClock.now() - beginLockTimestamp;
            if (costTime > 5 * 1000) {
                log.info("[NOTIFYME] check hold request cost {} ms.", costTime);
            }
        } catch (Throwable e) {
            log.warn(this.getServiceName() + " service has exception. ", e);
        }
    }

    log.info("{} service end", this.getServiceName());
}
```

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。







# Rabbitmq  Interview

## 什么是MQ

- MQ就是消息队列。是软件和软件进行通信的中间件产品

## MQ的优点

- 简答
  - 异步处理 - 相比于传统的串行、并行方式，提高了系统吞吐量。
  - 应用解耦 - 系统间通过消息通信，不用关心其他系统的处理。
  - 流量削锋 - 可以通过消息队列长度控制请求量；可以缓解短时间内的高并发请求。
  - 日志处理 - 解决大量日志传输。
  - 消息通讯 - 消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。
- 详答

## 解耦、异步、削峰是什么？。

- **解耦**：A 系统发送数据到 BCD 三个系统，通过接口调用发送。如果 E 系统也要这个数据呢？那如果 C 系统现在不需要了呢？A 系统负责人几乎崩溃…A 系统跟其它各种乱七八糟的系统严重耦合，A 系统产生一条比较关键的数据，很多系统都需要 A 系统将这个数据发送过来。如果使用 MQ，A 系统产生一条数据，发送到 MQ 里面去，哪个系统需要数据自己去 MQ 里面消费。如果新系统需要数据，直接从 MQ 里消费即可；如果某个系统不需要这条数据了，就取消对 MQ 消息的消费即可。这样下来，A 系统压根儿不需要去考虑要给谁发送数据，不需要维护这个代码，也不需要考虑人家是否调用成功、失败超时等情况。

  `就是一个系统或者一个模块，调用了多个系统或者模块，互相之间的调用很复杂，维护起来很麻烦。但是其实这个调用是不需要直接同步调用接口的，如果用 MQ 给它异步化解耦。`

- **异步**：A 系统接收一个请求，需要在自己本地写库，还需要在 BCD 三个系统写库，自己本地写库要 3ms，BCD 三个系统分别写库要 300ms、450ms、200ms。最终请求总延时是 3 + 300 + 450 + 200 = 953ms，接近 1s，用户感觉搞个什么东西，慢死了慢死了。用户通过浏览器发起请求。如果使用 MQ，那么 A 系统连续发送 3 条消息到 MQ 队列中，假如耗时 5ms，A 系统从接受一个请求到返回响应给用户，总时长是 3 + 5 = 8ms。

- **削峰**：减少高峰时期对服务器压力。

## 消息队列有什么缺点

- 缺点有以下几个：

  1. **系统可用性降低**

     本来系统运行好好的，现在你非要加入个消息队列进去，那消息队列挂了，你的系统不是呵呵了。因此，系统可用性会降低；

  2. **系统复杂度提高**

     加入了消息队列，要多考虑很多方面的问题，比如：一致性问题、如何保证消息不被重复消费、如何保证消息可靠性传输等。因此，需要考虑的东西更多，复杂性增大。

  3. **一致性问题**

     A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，咋整？你这数据就不一致了。

```
所以消息队列实际是一种非常复杂的架构，你引入它有很多好处，但是也得针对它带来的坏处做各种额外的技术方案和架构来规避掉，做好之后，你会发现，妈呀，系统复杂度提升了一个数量级，也许是复杂了 10 倍。但是关键时刻，用，还是得用的。
```

## 你们公司生产环境用的是什么消息中间件？

- 这个首先你可以说下你们公司选用的是什么消息中间件，比如用的是RabbitMQ，然后可以初步给一些你对不同MQ中间件技术的选型分析。
- 举个例子：比如说ActiveMQ是老牌的消息中间件，国内很多公司过去运用的还是非常广泛的，功能很强大。
- 但是问题在于没法确认ActiveMQ可以支撑互联网公司的高并发、高负载以及高吞吐的复杂场景，在国内互联网公司落地较少。而且使用较多的是一些传统企业，用ActiveMQ做异步调用和系统解耦。
- 然后你可以说说RabbitMQ，他的好处在于可以支撑高并发、高吞吐、性能很高，同时有非常完善便捷的后台管理界面可以使用。
- 另外，他还支持集群化、高可用部署架构、消息高可靠支持，功能较为完善。
- 而且经过调研，国内各大互联网公司落地大规模RabbitMQ集群支撑自身业务的case较多，国内各种中小型互联网公司使用RabbitMQ的实践也比较多。
- 除此之外，RabbitMQ的开源社区很活跃，较高频率的迭代版本，来修复发现的bug以及进行各种优化，因此综合考虑过后，公司采取了RabbitMQ。
- 但是RabbitMQ也有一点缺陷，就是他自身是基于erlang语言开发的，所以导致较为难以分析里面的源码，也较难进行深层次的源码定制和改造，毕竟需要较为扎实的erlang语言功底才可以。
- 然后可以聊聊RocketMQ，是阿里开源的，经过阿里的生产环境的超高并发、高吞吐的考验，性能卓越，同时还支持分布式事务等特殊场景。
- 而且RocketMQ是基于Java语言开发的，适合深入阅读源码，有需要可以站在源码层面解决线上生产问题，包括源码的二次开发和改造。
- 另外就是Kafka。Kafka提供的消息中间件的功能明显较少一些，相对上述几款MQ中间件要少很多。
- 但是Kafka的优势在于专为超高吞吐量的实时日志采集、实时数据同步、实时数据计算等场景来设计。
- 因此Kafka在大数据领域中配合实时计算技术（比如Spark Streaming、Storm、Flink）使用的较多。但是在传统的MQ中间件使用场景中较少采用。

## Kafka、ActiveMQ、RabbitMQ、RocketMQ 有什么优缺点？



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d49883657~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



- 综上，各种对比之后，有如下建议：
- 一般的业务系统要引入 MQ，最早大家都用 ActiveMQ，但是现在确实大家用的不多了，没经过大规模吞吐量场景的验证，社区也不是很活跃，所以大家还是算了吧，我个人不推荐用这个了；
- 后来大家开始用 RabbitMQ，但是确实 erlang 语言阻止了大量的 Java 工程师去深入研究和掌控它，对公司而言，几乎处于不可控的状态，但是确实人家是开源的，比较稳定的支持，活跃度也高；
- 不过现在确实越来越多的公司会去用 RocketMQ，确实很不错，毕竟是阿里出品，但社区可能有突然黄掉的风险（目前 RocketMQ 已捐给 [Apache](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq)，但 GitHub 上的活跃度其实不算高）对自己公司技术实力有绝对自信的，推荐用 RocketMQ，否则回去老老实实用 RabbitMQ 吧，人家有活跃的开源社区，绝对不会黄。
- 所以**中小型公司**，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择；**大型公司**，基础架构研发实力较强，用 RocketMQ 是很好的选择。
- 如果是**大数据领域**的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。

## MQ 有哪些常见问题？如何解决这些问题？

- MQ 的常见问题有：
  - 消息的顺序问题
  - 消息的重复问题

**消息的顺序问题**

- 消息有序指的是可以按照消息的发送顺序来消费。
- 假如生产者产生了 2 条消息：M1、M2，假定 M1 发送到 S1，M2 发送到 S2，如果要保证 M1 先于 M2 被消费，怎么做？



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d4996c042~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



- 解决方案：
  1. 保证生产者 - MQServer - 消费者是一对一对一的关系



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d49baaae7~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



- 缺陷：
  - 并行度就会成为消息系统的瓶颈（吞吐量不够）
  - 更多的异常处理，比如：只要消费端出现问题，就会导致整个处理流程阻塞，我们不得不花费更多的精力来解决阻塞的问题。 （2）通过合理的设计或者将问题分解来规避。
  - 不关注乱序的应用实际大量存在
  - 队列无序并不意味着消息无序 所以从业务层面来保证消息的顺序而不仅仅是依赖于消息系统，是一种更合理的方式。

**消息的重复问题**

- 造成消息重复的根本原因是：网络不可达。
- 所以解决这个问题的办法就是绕过这个问题。那么问题就变成了：如果消费端收到两条一样的消息，应该怎样处理？
- 消费端处理消息的业务逻辑保持幂等性。只要保持幂等性，不管来多少条重复消息，最后处理的结果都一样。保证每条消息都有唯一编号且保证消息处理成功与去重表的日志同时出现。利用一张日志表来记录已经处理成功的消息的 ID，如果新到的消息 ID 已经在日志表中，那么就不再处理这条消息。

## 什么是RabbitMQ？

- RabbitMQ是一款开源的，Erlang编写的，消息中间件； 最大的特点就是消费并不需要确保提供方存在,实现了服务之间的高度解耦 可以用它来：解耦、异步、削峰。

## rabbitmq 的使用场景

（1）服务间异步通信

（2）顺序消费

（3）定时任务

（4）请求削峰

## RabbitMQ基本概念

- Broker： 简单来说就是消息队列服务器实体
- Exchange： 消息交换机，它指定消息按什么规则，路由到哪个队列
- Queue： 消息队列载体，每个消息都会被投入到一个或多个队列
- Binding： 绑定，它的作用就是把exchange和queue按照路由规则绑定起来
- Routing Key： 路由关键字，exchange根据这个关键字进行消息投递
- VHost： vhost 可以理解为虚拟 broker ，即 mini-RabbitMQ server。其内部均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段（一个典型的例子就是不同的应用可以跑在不同的 vhost 中）。
- Producer： 消息生产者，就是投递消息的程序
- Consumer： 消息消费者，就是接受消息的程序
- Channel： 消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务

```
由Exchange、Queue、RoutingKey三个才能决定一个从Exchange到Queue的唯一的线路。
```

## RabbitMQ的工作模式

**一.simple模式（即最简单的收发模式）**



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d49b7ccb7~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



1. 消息产生消息，将消息放入队列
2. 消息的消费者(consumer) 监听 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失，这里可以设置成手动的ack,但如果设置成手动ack，处理完后要及时发送ack消息给队列，否则会造成内存溢出)。

**二.work工作模式(资源的竞争)**



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d4a6f6be6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



1. 消息产生者将消息放入队列消费者可以有多个,消费者1,消费者2同时监听同一个队列,消息被消费。C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize) 保证一条消息只能被一个消费者使用)。

**三.publish/subscribe发布订阅(共享资源)**



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d4a57e741~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



1. 每个消费者监听自己的队列；
2. 生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息。

**四.routing路由模式**



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d7605829e~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



1. 消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息;
2. 根据业务功能定义路由字符串
3. 从系统的代码逻辑中获取对应的功能字符串,将消息任务扔到对应的队列中。
4. 业务场景:error 通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误;

**五.topic 主题模式(路由模式的一种)**



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717348d779d8b8d~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



1. 星号井号代表通配符
2. 星号代表多个单词,井号代表一个单词
3. 路由功能添加模糊匹配
4. 消息产生者产生消息,把消息交给交换机
5. 交换机根据key的规则模糊匹配到对应的队列,由队列的监听消费者接收消息消费

```
（在我的理解看来就是routing查询的一种模糊匹配，就类似sql的模糊查询方式）
```

## 如何保证RabbitMQ消息的顺序性？

- 拆分多个 queue(消息队列)，每个 queue(消息队列) 一个 consumer(消费者)，就是多一些 queue (消息队列)而已，确实是麻烦点；
- 或者就一个 queue (消息队列)但是对应一个 consumer(消费者)，然后这个 consumer(消费者)内部用内存队列做排队，然后分发给底层不同的 worker 来处理。

## 消息如何分发？

- 若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。通过路由可实现多消费的功能

## 消息怎么路由？

- 消息提供方->路由->一至多个队列消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。通过队列路由键，可以把队列绑定到交换器上。消息到达交换器后，RabbitMQ 会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）；
- 常用的交换器主要分为一下三种：
  1. fanout：如果交换器收到消息，将会广播到所有绑定的队列上
  2. direct：如果路由键完全匹配，消息就被投递到相应的队列
  3. topic：可以使来自不同源头的消息能够到达同一个队列。 使用 topic 交换器时，可以使用通配符

## 消息基于什么传输？

- 由于 TCP 连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ 使用信道的方式来传输数据。信道是建立在真实的 TCP 连接内的虚拟连接，且每条 TCP 连接上的信道数量没有限制。

## 如何保证消息不被重复消费？或者说，如何保证消息消费时的幂等性？

- 先说为什么会重复消费：正常情况下，消费者在消费消息的时候，消费完毕后，会发送一个确认消息给消息队列，消息队列就知道该消息被消费了，就会将该消息从消息队列中删除；
- 但是因为网络传输等等故障，确认信息没有传送到消息队列，导致消息队列不知道自己已经消费过该消息了，再次将消息分发给其他的消费者。
- 针对以上问题，一个解决思路是：保证消息的唯一性，就算是多次传输，不要让消息的多次消费带来影响；保证消息等幂性；
  - 比如：在写入消息队列的数据做唯一标示，消费消息时，根据唯一标识判断是否消费过；
  - 假设你有个系统，消费一条消息就往数据库里插入一条数据，要是你一个消息重复两次，你不就插入了两条，这数据不就错了？但是你要是消费到第二次的时候，自己判断一下是否已经消费过了，若是就直接扔了，这样不就保留了一条数据，从而保证了数据的正确性。

## 如何确保消息正确地发送至 RabbitMQ？ 如何确保消息接收方消费了消息？

**发送方确认模式**

- 将信道设置成 confirm 模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的 ID。
- 一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一 ID）。
- 如果 RabbitMQ 发生内部错误从而导致消息丢失，会发送一条 nack（notacknowledged，未确认）消息。
- 发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

**接收方确认机制**

- 消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除。
- 这里并没有用到超时机制，RabbitMQ 仅通过 Consumer 的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ 给了 Consumer 足够长的时间来处理消息。保证数据的最终一致性；

**下面罗列几种特殊情况**

- 如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）
- 如果消费者接收到消息却没有确认消息，连接也未断开，则 RabbitMQ 认为该消费者繁忙，将不会给该消费者分发更多的消息。

## 如何保证RabbitMQ消息的可靠传输？

- 消息不可靠的情况可能是消息丢失，劫持等原因；
- 丢失又分为：生产者丢失消息、消息列表丢失消息、消费者丢失消息；

1. **生产者丢失消息**：从生产者弄丢数据这个角度来看，RabbitMQ提供transaction和confirm模式来确保生产者不丢消息；

   transaction机制就是说：发送消息前，开启事务（channel.txSelect()）,然后发送消息，如果发送过程中出现什么异常，事务就会回滚（channel.txRollback()）,如果发送成功则提交事务（channel.txCommit()）。然而，这种方式有个缺点：吞吐量下降；

   confirm模式用的居多：一旦channel进入confirm模式，所有在该信道上发布的消息都将会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后；

   rabbitMQ就会发送一个ACK给生产者（包含消息的唯一ID），这就使得生产者知道消息已经正确到达目的队列了；

   如果rabbitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

2. **消息队列丢数据**：消息持久化。

   处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。

   这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。

   这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

   那么如何持久化呢？

   这里顺便说一下吧，其实也很容易，就下面两步

   ​	1. 将queue的持久化标识durable设置为true,则代表是一个持久的队列

   ​	2. 发送消息的时候将deliveryMode=2

   这样设置以后，即使rabbitMQ挂了，重启后也能恢复数据

3. **消费者丢失消息**：消费者丢数据一般是因为采用了自动确认消息模式，改为手动确认消息即可！

   消费者在收到消息之后，处理消息之前，会自动回复RabbitMQ已收到消息；

   如果这时处理消息失败，就会丢失该消息；

   解决方案：处理消息成功后，手动回复确认消息。

## 为什么不应该对所有的 message 都使用持久化机制？

- 首先，必然导致性能的下降，因为写磁盘比写 RAM 慢的多，message 的吞吐量可能有 10 倍的差距。
- 其次，message 的持久化机制用在 RabbitMQ 的内置 cluster 方案时会出现“坑爹”问题。矛盾点在于，若 message 设置了 persistent 属性，但 queue 未设置 durable 属性，那么当该 queue 的 owner node 出现异常后，在未重建该 queue 前，发往该 queue 的 message 将被 blackholed ；若 message 设置了 persistent 属性，同时 queue 也设置了 durable 属性，那么当 queue 的 owner node 异常且无法重启的情况下，则该 queue 无法在其他 node 上重建，只能等待其 owner node 重启后，才能恢复该 queue 的使用，而在这段时间内发送给该 queue 的 message 将被 blackholed 。
- 所以，是否要对 message 进行持久化，需要综合考虑性能需要，以及可能遇到的问题。若想达到 100,000 条/秒以上的消息吞吐量（单 RabbitMQ 服务器），则要么使用其他的方式来确保 message 的可靠 delivery ，要么使用非常快速的存储系统以支持全持久化（例如使用 SSD）。另外一种处理原则是：仅对关键消息作持久化处理（根据业务重要程度），且应该保证关键消息的量不会导致性能瓶颈。

## 如何保证高可用的？RabbitMQ 的集群

- RabbitMQ 是比较有代表性的，因为是基于主从（非分布式）做高可用性的，我们就以 RabbitMQ 为例子讲解第一种 MQ 的高可用性怎么实现。RabbitMQ 有三种模式：单机模式、普通集群模式、镜像集群模式。

1. **单机模式**，就是 Demo 级别的，一般就是你本地启动了玩玩儿的?，没人生产用单机模式
2. **普通集群模式**：
   - 意思就是在多台机器上启动多个 RabbitMQ 实例，每个机器启动一个。
   - 你创建的 queue，只会放在一个 RabbitMQ 实例上，但是每个实例都同步 queue 的元数据（元数据可以认为是 queue 的一些配置信息，通过元数据，可以找到 queue 所在实例）。你消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从 queue 所在实例上拉取数据过来。这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个 queue 的读写操作。
3. **镜像集群模式**：
   - 这种模式，才是所谓的 RabbitMQ 的高可用模式。跟普通集群模式不一样的是，在镜像集群模式下，你创建的 queue，无论元数据还是 queue 里的消息都会存在于多个实例上，就是说，每个 RabbitMQ 节点都有这个 queue 的一个完整镜像，包含 queue 的全部数据的意思。然后每次你写消息到 queue 的时候，都会自动把消息同步到多个实例的 queue 上。RabbitMQ 有很好的管理控制台，就是在后台新增一个策略，这个策略是镜像集群模式的策略，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建 queue 的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。
   - 这样的好处在于，你任何一个机器宕机了，没事儿，其它机器（节点）还包含了这个 queue 的完整数据，别的 consumer 都可以到其它节点上去消费数据。坏处在于，第一，这个性能开销也太大了吧，消息需要同步到所有机器上，导致网络带宽压力和消耗很重！RabbitMQ 一个 queue 的数据都是放在一个节点里的，镜像集群下，也是每个节点都放这个 queue 的完整数据。

## 如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，怎么办？

- 消息积压处理办法：临时紧急扩容：
- 先修复 consumer 的问题，确保其恢复消费速度，然后将现有 cnosumer 都停掉。
   新建一个 topic，partition 是原来的 10 倍，临时建立好原先 10 倍的 queue 数量。
   然后写一个临时的分发数据的 consumer 程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的 10 倍数量的 queue。
   接着临时征用 10 倍的机器来部署 consumer，每一批 consumer 消费一个临时 queue 的数据。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。
   等快速消费完积压数据之后，得恢复原先部署的架构，重新用原先的 consumer 机器来消费消息。
   MQ中消息失效：假设你用的是 RabbitMQ，RabbtiMQ 是可以设置过期时间的，也就是 TTL。如果消息在 queue 中积压超过一定的时间就会被 RabbitMQ 给清理掉，这个数据就没了。那这就是第二个坑了。这就不是说数据会大量积压在 mq 里，而是大量的数据会直接搞丢。我们可以采取一个方案，就是批量重导，这个我们之前线上也有类似的场景干过。就是大量积压的时候，我们当时就直接丢弃数据了，然后等过了高峰期以后，比如大家一起喝咖啡熬夜到晚上12点以后，用户都睡觉了。这个时候我们就开始写程序，将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入 mq 里面去，把白天丢的数据给他补回来。也只能是这样了。假设 1 万个订单积压在 mq 里面，没有处理，其中 1000 个订单都丢了，你只能手动写程序把那 1000 个订单给查出来，手动发到 mq 里去再补一次。
- mq消息队列块满了：如果消息积压在 mq 里，你很长时间都没有处理掉，此时导致 mq 都快写满了，咋办？这个还有别的办法吗？没有，谁让你第一个方案执行的太慢了，你临时写程序，接入数据来消费，消费一个丢弃一个，都不要了，快速消费掉所有的消息。然后走第二个方案，到了晚上再补数据吧。

## 设计MQ思路

- 比如说这个消息队列系统，我们从以下几个角度来考虑一下：
- 首先这个 mq 得支持可伸缩性吧，就是需要的时候快速扩容，就可以增加吞吐量和容量，那怎么搞？设计个分布式的系统呗，参照一下 kafka 的设计理念，broker -> topic -> partition，每个 partition 放一个机器，就存一部分数据。如果现在资源不够了，简单啊，给 topic 增加 partition，然后做数据迁移，增加机器，不就可以存放更多数据，提供更高的吞吐量了？
- 其次你得考虑一下这个 mq 的数据要不要落地磁盘吧？那肯定要了，落磁盘才能保证别进程挂了数据就丢了。那落磁盘的时候怎么落啊？顺序写，这样就没有磁盘随机读写的寻址开销，磁盘顺序读写的性能是很高的，这就是 kafka 的思路。
- 其次你考虑一下你的 mq 的可用性啊？这个事儿，具体参考之前可用性那个环节讲解的 kafka 的高可用保障机制。多副本 -> leader & follower -> broker 挂了重新选举 leader 即可对外服务。
- 能不能支持数据 0 丢失啊？可以呀，有点复杂的。



作者：小杰要吃蛋
链接：https://juejin.cn/post/6844904125935665160
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





# Rabbitmq  Interview2

## 1.RabbitMQ特点

1. 可靠性: RabbitMQ使用一些机制来保证可靠性， 如持久化、传输确认及发布确认等。
2. 扩展性: 多个RabbitMQ节点可以组成一个集群，也可以根据实际业务情况动态地扩展 集群中节点。
3. 管理界面 : RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息、集 群中的节点等。
4. 插件机制 : RabbitMQ 提供了许多插件 ， 以实现从多方面进行扩展，当然也可以编写自己的插件。

## 2.AMQP是什么?

RabbitMQ就是 `AMQP` 协议的 `Erlang` 的实现(当然 RabbitMQ 还支持 STOMP2、 MQTT3 等协议 ) AMQP 的模型架构 和 RabbitMQ 的模型架构是一样的，生产者将消息发送给交换机，交换机和队列进行绑定 。

## 3.AMQP模型的几大组件？

1. 交换器 (Exchange)：消息代理服务器中用于把消息路由到队列的组件。
2. 队列 (Queue)：用来存储消息的数据结构，位于硬盘或内存中。
3. 绑定 (Binding)：一套规则，告知交换器消息应该将消息投递给哪个队列。

## 4.说说生产者Producer和消费者Consumer?

1. 生产者

- 消息生产者，就是投递消息的一方。
- 消息一般包含两个部分：消息体（payload)和标签(Label)。

1. 消费者

- 消费消息，也就是接收消息的一方。
- 消费者连接到RabbitMQ服务器，并订阅到队列上。消费消息时只消费消息体，丢弃标签。

## 5.为什么使用MQ？MQ的优点

1. 异步处理 - 相比于传统的串行、并行方式，提高了系统的吞吐量。
2. 应用解耦 - 系统间通过消息通信，不用关心其他系统的处理。
3. 流量削锋 - 可以通过消息队列长度控制请求量，可以缓解短时间内的高并发请求。
4. 消息通讯 - 消息队列一般都内置了高效的通信机制，因此也可以用在纯消息通讯上。比如实现点对点消息队列，或者聊天室等。
5. 日志处理 - 解决大量日志传输。

## 6.说说Broker服务节点、Queue队列、Exchange交换器？

1. Broker可以看做RabbitMQ的服务节点。一般请下一个Broker可以看做一个RabbitMQ服务器。
2. Queue:RabbitMQ的内部对象，用于存储消息。多个消费者可以订阅同一队列，这时队列中的消息会被平摊（轮询）给多个消费者进行处理。
3. Exchange:生产者将消息发送到交换器，由交换器将消息路由到一个或者多个队列中。当路由不到时，或返回给生产者或直接丢弃。

## 7.你们公司生产环境用的是什么消息中间件？

> 对不同MQ中间件技术的选型分析。

1. 比如说ActiveMQ是老牌的消息中间件，国内很多公司过去运用的还是非常广泛的，功能很强大。

但是问题在于ActiveMQ没法支撑互联网公司的高并发、高负载以及高吞吐的复杂场景，现在在国内互联网公司落地较少。

1. RabbitMQ，

- 他的好处在于可以支撑高并发、高吞吐量、性能很高，同时有非常完善便捷的后台管理界面可以使用。
- 另外，他还支持集群化、高可用部署架构、消息高可靠支持，功能较为完善。
- 而且经过调研，国内各大互联网公司落地RabbitMQ集群支撑自身业务的case较多，国内各种中小型互联网公司使用RabbitMQ的实践也比较多。
- 除此之外，RabbitMQ的开源社区很活跃，较高频率的版本迭代，来修复发现的bug以及进行各种优化，因此综合考虑过后，公司采取了RabbitMQ。

1. 然后可以聊聊RocketMQ

- 是阿里开源的，经过阿里生产环境的超高并发、高吞吐的考验，性能卓越，同时还支持分布式事务等特殊场景。
- 而且RocketMQ是基于Java语言开发的，适合深入阅读源码，有需要可以站在源码层面解决线上问题，包括源码的二次开发和改造。

1. 另外就是Kafka。

- Kafka提供的消息中间件的功能明显较少一些，相对上述几款MQ中间件要少很多。
- 但是Kafka的优势在于专为超高吞吐量的实时日志采集、实时数据同步、实时数据计算等场景。

## 8.Kafka、ActiveMQ、RabbitMQ、RocketMQ 有什么优缺点？

| 项目       | ActiveMQ                                                | RabbitMQ                                                     | RocketMQ                                                     | Kafka                                                        |      |
| :--------- | :------------------------------------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--- |
| 单机吞吐量 | 比RabbitMQ低                                            | 2.6w/s（消息做持久化）                                       | 11.6w/s                                                      | 17.3w/s                                                      |      |
| 开发语言   | Java                                                    | Erlang                                                       | Java                                                         | Scala/Java                                                   |      |
| 主要维护者 | Apache                                                  | Mozilla/Spring                                               | Alibaba                                                      | Apache                                                       |      |
| 成熟度     | 成熟                                                    | 成熟                                                         | 开源版本不够成熟                                             | 比较成熟                                                     |      |
| 订阅形式   | 点对点(p2p)、广播（发布-订阅）                          | 提供了4种：direct, topic ,Headers和fanout。fanout就是广播模式 | 基于topic/messageTag以及按照消息类型、属性进行正则匹配的发布订阅模式 | 基于topic以及按照topic进行正则匹配的发布订阅模式             |      |
| 持久化     | 支持少量堆积                                            | 支持少量堆积                                                 | 支持大量堆积                                                 | 支持大量堆积                                                 |      |
| 顺序消息   | 不支持                                                  | 不支持                                                       | 支持                                                         | 支持                                                         |      |
| 性能稳定性 | 好                                                      | 好                                                           | 一般                                                         | 较差                                                         |      |
| 集群方式   | 支持简单集群模式，比如’主-备’，对高级集群模式支持不好。 | 支持简单集群，'复制’模式，对高级集群模式支持不好。           | 常用 多对’Master-Slave’ 模式，开源版本需手动切换Slave变成Maste | 天然的‘Leader-Slave’无状态集群，每台服务器既是Master也是Slave |      |
| 管理界面   | 一般                                                    | 较好                                                         | 一般                                                         | 无                                                           |      |

## 9.消息积压怎么处理

> 消息积压的直接原因，一定是系统中某个部分出现了性能问题，来不及处理上游发送的消息，才会导致消息积压。

1. 从两个方面开始考虑：

- 一个是生产者生产的消息过快，导致消费者消费不及时
- 二是消费者出现异常，导致一直无法接收新的消息

1. 设置消息过期时间，过期后转入死信队列，写一个程序处理死信消息（重新如队列或者即使处理或记录到数据库延后处理）
2. 考虑使用队列最大长度限制

## 10.如何保证RabbitMQ消息的可靠传输？消息丢失怎么办？

> 丢失又分为：生产者丢失消息、消息列表丢失消息、消费者丢失消息；

1. 生产者的解决策略

- `开启事务`（同步操作，影响性能，吞吐量下降），发送消息前，开启事务（channel.txSelect()），然后发送消息，如果发送过程中出现什么异常，事务就会回滚（channel.txRollback()），如果发送成功则提交事务（channel.txCommit()）
- `生产者确认模式`，所有在该信道上发布的消息都将会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个`ACK`给生产者（包含消息的唯一ID），这就代表成功将消息发送到队列中了；如果rabbitMQ没能处理该消息，则会发送一个`Nack`消息给你，生产者可以进行重试操作。

1. 队列的解决策略

处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。
这个`持久化`配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。
这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

1. 消费者的解决策略

消费者在收到消息之后，处理消息之前，会自动回复RabbitMQ已收到消息；
如果这时处理消息失败，就会丢失该消息；
解决方案：处理消息成功后，`手动回复确认消息`。

1. 最终

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/e6acf7c100fb429a8d38912f52588c6a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ZOS5ZOS6K-0SmF2YQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 11.RabbitMQ 常见工作模式和应用场景

### 11.1 简单模式

> 一个生产者，一个消费者。生产者将消息发送到队列，消费者监听消息队列，如果队列中有消息，就进行消费，消费后消息从队列中删除

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/5ae164cb25aa4e31a9ccc3b2c6ffe5b3.png)

### 11.2 工作模式

> 原理：一个生产者，多个消费者，一条消息只能被一个消费者消费。生产者将消息发送到消息队列，多个消费者同时监听一个队列，谁先抢到消息谁负责消费。这样就形成了资源竞争，谁的资源空闲大，争抢到的可能性就大。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/74eba76efed34fa0a5914dbbfe545add.png)

### 11.3 发布订阅模式

> 一个生产者，多个消费者，每个消费者都可以收到相同的消息。生产者将消息发送到交换机，交换机类型是fanout，不同的队列注册到交换机上，不同的消费者监听不同的队列，所有消费者都会收到消息。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/f76972a9a4634835b81fb68b4803d9d0.png)

### 11.4 路由模式

> 生产者将消息发送给交换机，消息携带具体的routingkey。交换机类型是direct，交换机匹配与之绑定的队列的routingkey，分发到不同的队列上。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/9713691cb51a4376a48c00b0a030421e.png)

### 11.5 主题模式

> 路由模式的一种，交换机类型是topic，路由功能添加了模糊匹配。星号（*）代表1个单词，#号（#）代表一个或多个单词。

![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/25af43d9a9cd41ed90ec6a163f5868d7.png)