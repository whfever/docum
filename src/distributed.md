# Distributed
#分布式

##iD
###ID
UUID的优点就是他的性能比较高，不依赖网络，本地就可以生成，使用起来也比较简单。
但是他也有两个比较明显的缺点，那就是*长度过长和没有任何含义*
 11
### SnowFlake
所以，基于时间戳+数据中心标识+机器标识+序列号，就保证了在不同进程中主键的不重复，在相同进程中主键的有序性。

雪花算法之所以被广泛使用，主要是因为他有以下优点：
1
高性能高可用：生成时不依赖于数据库，完全在内存中生成
2
高吞吐：每秒钟能生成数百万的自增 ID
3
ID 自增：在单个进程中，生成的ID是自增的，可以用作数据库主键做范围查询。但是需要注意的是，在集群中是没办法保证一定顺序递增的。

SnowFlake 算法的缺点或者限制：

1、在Snowflake算法中，每个节点的机器ID和数据中心ID都是硬编码在代码中的，而且这些ID是全局唯一的。当某个节点出现故障或者需要扩容时，就需要更改其对应的机器ID或数据中心ID，但是这个过程比较麻烦，需要重新编译代码，重新部署系统。还有就是，如果某个节点的机器ID或数据中心ID被设置成了已经被分配的ID，那么就会出现重复的ID，这样会导致系统的错误和异常。

2、Snowflake算法中，需要使用zookeeper来协调各个节点的ID生成，但是ZK的部署其实是有挺大的成本的，并且zookeeper本身也可能成为系统的瓶颈。

3、依赖于系统时间的一致性，如果系统*时间被回拨*，或者不一致，可能会造成 ID 重复。


### 事务
首先，二者的实现机制不同，2PC使用*协调者和参与者*的方式来实现分布式事务，而TCC采用分阶段提交的方式。


处理方式不同，2PC采用预写式日志的方式，在提交和回滚阶段需要协调者和参与者之间进行多次网络通信，整个事务处理过程较为复杂。TCC则不需要网络通信，只需要在Try、Confirm和Cancel阶段执行相应的业务逻辑。

异常处理不同，2PC需要处理网络、节点故障等异常情况，可能会导致整个事务无法提交或回滚，处理异常情况的复杂度较高。而TCC只需要处理业务异常情况，异常处理相对简单。

适用场景不同，2PC适用于对事务一致性要求较高的场景，例如银行转账等，需要保证数据一致性和完整性。而TCC适用于对事务一致性要求不那么高的场景，例如电商库存扣减等，需要保证数据最终一致性即可。

最初TCC的设计是强一致性，基本上一次事务执行完之后，数据是一致的，要么都commit，要么都cancel。

但是其实在实际使用过程中，可能会采用最终一致性的思想，比如commit失败之后，进行异步重试让他尝试成功，而不是立刻cancel
详解

## ♥分布式系统 - 知识体系详解♥

> TBD

- 分布式系统 - 理论基础,理论及一致性算法
  - 一个分布式系统是一些独立的计算机集合，但是对这个系统的用户来说，系统就像一台计算机一样。本文主要介绍分布式的基础知识，以及分布式的理论基础和一致性算法。
- 分布式系统 - 全局唯一ID实现方案
  - 本文主要介绍常见的分布式ID生成方式，大致分类的话可以分为两类：一种是类DB型的，根据设置不同起始值和步长来实现趋势递增，需要考虑服务的容错性和可用性; 另一种是类snowflake型，这种就是将64位划分为不同的段，每段代表不同的涵义，基本就是时间戳、机器ID和序列数。这种方案就是需要考虑时钟回拨的问题以及做一些 buffer的缓冲设计提高性能。
- 分布式系统 - 分布式锁及实现方案
  - 本文主要介绍分布式锁的概念和分布式锁的设计原则，以及常见的分布式锁的实现方式。
- 分布式系统 - 分布式事务及实现方案
  - **事务**是一个程序执行单元，里面的所有操作要么全部执行成功，要么全部执行失败。而**分布式事务**是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。
- 分布式系统 - 分布式任务及实现方案
  - 本文主要介绍定时任务的基础和单体方式下定时任务方案的演化，以及常见的分布式任务方案(包括Quartz Cluster，ElasticJob，xxl-job等）和技术实现要点。
- 分布式系统 - 分布式会话及实现方案
  - 无状态的token或者有状态的Session集中管理是目前最为常用的方案，本节主要讨论的有状态的分布式Session会话, 包括Session Stick, Session Replication, Session 数据集中存储, Cookie Based 以及Token方式等。
- 分布式系统 - 分布式系统的8个谬误
  - 20多年前，Peter Deutsch和James Gosling定义了分布式计算的8个谬误。这些是许多开发人员对分布式系统做出的错误假设。从长远来看，这些通常被证明是错误的，导致难以修复错误。这篇文章从反面警示我们分布式系统需要注意的事项。

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/arch/arch-z-overview.html



## 分布式系统 - 理论基础,理论及一致性算法

> 一个分布式系统是一些独立的计算机集合，但是对这个系统的用户来说，系统就像一台计算机一样。本文主要介绍分布式的基础知识，以及分布式的理论基础和一致性算法。@pdai

[什么是分布式系统？]()[分布式系统的主要特征]()[分布式系统面临的问题]()[衡量分布式系统的指标]()[分布式理论基础]()[CAP理论]()[BASE理论]()[分布式一致性算法]()[一致性Hash算法]()[Paxos算法]()[Raft算法]()[ZAB算法]()[参考文章]()

### [#](#什么是分布式系统) 什么是分布式系统？

> 一个分布式系统是一些独立的计算机集合，但是对这个系统的用户来说，系统就像一台计算机一样。

分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统。简单来说就是**一群独立计算机集合共同对外提供服务，但是对于系统的用户来说，就像是一台计算机在提供服务一样**。分布式意味着可以采用更多的普通计算机（相对于昂贵的大型机）组成分布式集群对外提供服务。计算机越多，CPU、内存、存储资源等也就越多，能够处理的并发访问量也就越大。

从分布式系统的概念中我们知道，各个主机之间通信和协调主要通过网络进行，所以分布式系统中的计算机在空间上几乎没有任何限制，这些计算机可能被放在不同的机柜上，也可能被部署在不同的机房中，还可能在不同的城市中，对于大型的网站甚至可能分布在不同的国家和地区。

#### [#](#分布式系统的主要特征) 分布式系统的主要特征

> 无论空间上如何分布，一个标准的分布式系统应该具有以下几个主要特征

- **分布性**

分布式系统中的多台计算机之间在空间位置上可以随意分布，同时，机器的分布情况也会随时变动。

- **对等性**

分布式系统中的计算机没有主／从之分，即没有控制整个系统的主机，也没有被控制的从机，组成分布式系统的所有计算机节点都是对等的。副本（Replica）是分布式系统最常见的概念之一，指的是分布式系统对数据和服务提供的一种冗余方式。在常见的分布式系统中，为了对外提供高可用的服务，我们往往会对数据和服务进行副本处理。数据副本是指在不同节点上持久化同一份数据，当某一个节点上存储的数据丢失时，可以从副本上读取该数据，这是解决分布式系统数据丢失问题最为有效的手段。另一类副本是服务副本，指多个节点提供同样的服务，每个节点都有能力接收来自外部的请求并进行相应的处理。

- **自治性**

分布式系统中的各个节点都包含自己的处理机和内存，各自具有独立的处理数据的功能。通常，彼此在地位上是平等的，无主次之分，既能自治地进行工作，又能利用共享的通信线路来传送信息，协调任务处理。

- **并发性**

在一个计算机网络中，程序运行过程的并发性操作是非常常见的行为。例如同一个分布式系统中的多个节点，可能会并发地操作一些共享的资源，如何准确并高效地协调分布式并发操作也成为了分布式系统架构与设计中最大的挑战之一。

#### [#](#分布式系统面临的问题) 分布式系统面临的问题

> 分布式系统面临的问题有哪些？

- **缺乏全局时钟**

在分布式系统中，很难定义两个事件究竟谁先谁后，原因就是因为分布式系统缺乏一个全局的时钟序列控制。

- **机器宕机**

机器宕机是最常见的异常之一。在大型集群中每日宕机发生的概率为千分之一左右，在实践中，一台宕机的机器恢复的时间通常认为是24 小时，一般需要人工介入重启机器。

- **网络异常**

消息丢失，两片节点之间彼此完全无法通信，即出现了“网络分化”；消息乱序，有一定的概率不是按照发送时的顺序依次到达目的节点，考虑使用序列号等机制处理网络消息的乱序问题，使得无效的、过期的网络消息不影响系统的正确性；数据错误；不可靠的TCP，TCP 协议为应用层提供了可靠的、面向连接的传输服务，但在分布式系统的协议设计中不能认为所有网络通信都基于TCP 协议则通信就是可靠的。TCP协议只能保证同一个TCP 链接内的网络消息不乱序，TCP 链接之间的网络消息顺序则无法保证。

- **分布式三态**

如果某个节点向另一个节点发起RPC(Remote procedure call)调用，即某个节点A 向另一个节点B 发送一个消息，节点B 根据收到的消息内容完成某些操作，并将操作的结果通过另一个消息返回给节点A，那么这个RPC 执行的结果有三种状态：“成功”、“失败”、“超时（未知）”，称之为分布式系统的三态。

- **存储数据丢失**

对于有状态节点来说，数据丢失意味着状态丢失，通常只能从其他节点读取、恢复存储的状态。 *异常处理原则*：被大量工程实践所检验过的异常处理黄金原则是：任何在设计阶段考虑到的异常情况一定会在系统实际运行中发生，但在系统实际运行遇到的异常却很有可能在设计时未能考虑，所以，除非需求指标允许，在系统设计时不能放过任何异常情况。

#### [#](#衡量分布式系统的指标) 衡量分布式系统的指标

> 衡量分布式系统的指标有哪些？

- **性能**

系统的吞吐能力，指系统在某一时间可以处理的数据总量，通常可以用系统每秒处理的总的数据量来衡量；系统的响应延迟，指系统完成某一功能需要使用的时间；系统的并发能力，指系统可以同时完成某一功能的能力，通常也用QPS(query per second)来衡量。上述三个性能指标往往会相互制约，追求高吞吐的系统，往往很难做到低延迟；系统平均响应时间较长时，也很难提高QPS。

- **可用性**

系统的可用性(availability)指系统在面对各种异常时可以正确提供服务的能力。系统的可用性可以用系统停服务的时间与正常服务的时间的比例来衡量，也可以用某功能的失败次数与成功次数的比例来衡量。可用性是分布式的重要指标，衡量了系统的鲁棒性，是系统容错能力的体现。

- **可扩展性**

系统的可扩展性(scalability)指分布式系统通过扩展集群机器规模提高系统性能（吞吐、延迟、并发）、存储容量、计算能力的特性。好的分布式系统总在追求“线性扩展性”，也就是使得系统的某一指标可以随着集群中的机器数量线性增长。

- **一致性**

分布式系统为了提高可用性，总是不可避免的使用副本的机制，从而引发副本一致性的问题。越是强的一致的性模型，对于用户使用来说使用起来越简单。

### [#](#分布式理论基础) 分布式理论基础

> 主要包含CAP理论和BASE理论。

#### [#](#cap理论) CAP理论

> CAP理论是分布式系统、特别是分布式存储领域中被讨论的最多的理论。其中C代表一致性 (Consistency)，A代表可用性 (Availability)，P代表分区容错性 (Partition tolerance)。CAP理论告诉我们C、A、P三者不能同时满足，最多只能满足其中两个。

![CAP](https://pdai.tech/images/arch/arch-cap-1.png)

- [分布式理论 - CAP]()

#### [#](#base理论) BASE理论

> BASE是“Basically Available, Soft state, Eventually consistent(基本可用、软状态、最终一致性)”的首字母缩写。其中的软状态和最终一致性这两种技巧擅于对付存在分区的场合，并因此提高了可用性。

>  Base  是对AP的补充  关注*<u>最终一致性</u>*

![](https://pdai.tech/images/arch/arch-cap-2.png)

- [分布式理论 - BASE]()

#### [#](#分布式一致性算法) 分布式一致性算法

> 一致性算法的目的是保证在分布式系统中，多数据副本节点数据一致性。主要包含一致性Hash算法，Paxos算法，Raft算法，ZAB算法等。

##### [#](#一致性hash算法) 一致性Hash算法

- 分布式算法 - 一致性Hash算法
  - 一致性Hash算法是个经典算法，Hash环的引入是为解决`单调性(Monotonicity)`的问题；虚拟节点的引入是为了解决`平衡性(Balance)`问题

##### [#](#paxos算法) Paxos算法

- 分布式算法 - Paxos算法
  - Paxos算法是Lamport宗师提出的一种基于消息传递的分布式一致性算法，使其获得2013年图灵奖。自Paxos问世以来就持续垄断了分布式一致性算法，Paxos这个名词几乎等同于分布式一致性, 很多分布式一致性算法都由Paxos演变而来
  
  ```
  //Paxos 算法是一种用于在分布式系统中达成一致的算法，通过协调不同节点之间的提案，确保多数节点的一致性。它的基本思想是，节点间相互通信，提出提案、接受提案，并最终达成一致意见。
  
  //ZooKeeper 算法原理：
  
  ZooKeeper 使用了类似 Paxos 的原则来实现分布式协调服务。ZooKeeper 中的核心概念是ZAB（ZooKeeper Atomic Broadcast） 协议，它是一种基于 Paxos 的协议，用于实现数据的原子广播。
  
  ZooKeeper 将所有的写操作转换为一个有序的事务日志，并将这些事务日志按照 ZAB 协议进行广播，确保多数节点收到并写入。这样，ZooKeeper 能够保持数据的一致性，并通过选举机制选出领导者（Leader）节点，以处理客户端请求。当领导者发生故障时，通过选举算法选择新的领导者。
  
  ZooKeeper 还提供了分层的命名空间、临时节点、观察者模式等功能，用于协调分布式系统中的数据管理、配置管理和状态协调等任务。
  
  总之，ZooKeeper 使用类似 Paxos 的原则来实现分布式系统的数据管理和协调，确保多个节点之间的一致性和可靠性。
  ```
  
  

##### [#](#raft算法) Raft算法

- 分布式算法 - Raft算法
  - Paxos是出了名的难懂，而Raft正是为了探索一种更易于理解的一致性算法而产生的。它的首要设计目的就是易于理解，所以在选主的冲突处理等方式上它都选择了非常简单明了的解决方案

##### [#](#zab算法) ZAB算法

- 分布式算法 - ZAB算法
  - ZAB 协议全称：Zookeeper Atomic Broadcast（Zookeeper 原子广播协议）, 它应该是所有一致性协议中生产环境中应用最多的了。为什么呢？因为他是为 Zookeeper 设计的分布式一致性协议！

[#](#参考文章) 参考文章

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/arch/arch-z-theory.html



## 分布式系统 - 全局唯一ID实现方案

> 本文主要介绍常见的分布式ID生成方式，大致分类的话可以分为两类：**一种是类DB型的**，根据设置不同起始值和步长来实现趋势递增，需要考虑服务的容错性和可用性; **另一种是类snowflake型**，这种就是将64位划分为不同的段，每段代表不同的涵义，基本就是时间戳、机器ID和序列数。这种方案就是需要考虑时钟回拨的问题以及做一些 buffer的缓冲设计提高性能。@pdai

[为什么需要全局唯一ID]()[UUID]()[数据库生成]()[使用redis实现]()[雪花算法-Snowflake]()[百度-UidGenerator]()[DefaultUidGenerator 实现]()[CachedUidGenerator 实现]()[美团Leaf]()[Leaf-segment 数据库方案]()[Leaf-snowflake方案]()[Mist 薄雾算法]()[考量了什么业务场景和要求呢？]()[薄雾算法的设计思路是怎么样的？]()[薄雾算法生成的数值是什么样的？]()[薄雾算法 mist 和雪花算法 snowflake 有何区别？]()[为什么薄雾算法不受时间回拨影响？]()[为什么说薄雾算法的结果值不可预测？]()[当程序重启，薄雾算法的值会重复吗？]()[薄雾算法的值会重复，那我要它干嘛？]()[是否提供薄雾算法的工程实践或者架构实践？]()[薄雾算法的分布式架构，推荐 CP 还是 AP？]()[总结]()[参考文章]()

### [#](#为什么需要全局唯一id) 为什么需要全局唯一ID

传统的单体架构的时候，我们基本是单库然后业务单表的结构。每个业务表的ID一般我们都是从1增，通过AUTO_INCREMENT=1设置自增起始值，但是在分布式服务架构模式下分库分表的设计，使得多个库或多个表存储相同的业务数据。这种情况根据数据库的自增ID就会产生相同ID的情况，不能保证主键的唯一性。

![img](https://pdai.tech/images/arch/arch-z-id-1.png)

如上图，如果第一个订单存储在 DB1 上则订单 ID 为1，当一个新订单又入库了存储在 DB2 上订单 ID 也为1。我们系统的架构虽然是分布式的，但是在用户层应是无感知的，重复的订单主键显而易见是不被允许的。那么针对分布式系统如何做到主键唯一性呢？

### [#](#uuid) UUID

`UUID （Universally Unique Identifier）`，通用唯一识别码的缩写。UUID是由一组32位数的16进制数字所构成，所以UUID理论上的总数为 `16^32=2^128`，约等于 `3.4 x 10^38`。也就是说若每纳秒产生1兆个UUID，要花100亿年才会将所有UUID用完。

生成的UUID是由 8-4-4-4-12格式的数据组成，其中32个字符和4个连字符' - '，一般我们使用的时候会将连字符删除 uuid.`toString().replaceAll("-","")`。

目前UUID的产生方式有5种版本，每个版本的算法不同，应用范围也不同。

- `基于时间的UUID` - 版本1： 这个一般是通过当前时间，随机数，和本地Mac地址来计算出来，可以通过 org.apache.logging.log4j.core.util包中的 UuidUtil.getTimeBasedUuid()来使用或者其他包中工具。由于使用了MAC地址，因此能够确保唯一性，但是同时也暴露了MAC地址，私密性不够好。
- `DCE安全的UUID` - 版本2 DCE（Distributed Computing Environment）安全的UUID和基于时间的UUID算法相同，但会把时间戳的前4位置换为POSIX的UID或GID。这个版本的UUID在实际中较少用到。
- `基于名字的UUID（MD5）`- 版本3 基于名字的UUID通过计算名字和名字空间的MD5散列值得到。这个版本的UUID保证了：相同名字空间中不同名字生成的UUID的唯一性；不同名字空间中的UUID的唯一性；相同名字空间中相同名字的UUID重复生成是相同的。
- `随机UUID` - 版本4 根据随机数，或者伪随机数生成UUID。这种UUID产生重复的概率是可以计算出来的，但是重复的可能性可以忽略不计，因此该版本也是被经常使用的版本。JDK中使用的就是这个版本。
- `基于名字的UUID（SHA1）` - 版本5 和基于名字的UUID算法类似，只是散列值计算使用SHA1（Secure Hash Algorithm 1）算法。

我们 Java中 JDK自带的 UUID产生方式就是版本4根据随机数生成的 UUID 和版本3基于名字的 UUID，有兴趣的可以去看看它的源码。

```java
public static void main(String[] args) {

    //获取一个版本4根据随机字节数组的UUID。
    UUID uuid = UUID.randomUUID();
    System.out.println(uuid.toString().replaceAll("-",""));

    //获取一个版本3(基于名称)根据指定的字节数组的UUID。
    byte[] nbyte = {10, 20, 30};
    UUID uuidFromBytes = UUID.nameUUIDFromBytes(nbyte);
    System.out.println(uuidFromBytes.toString().replaceAll("-",""));
}
```

得到的UUID结果，

```java
59f51e7ea5ca453bbfaf2c1579f09f1d
7f49b84d0bbc38e9a493718013baace6
```

虽然 UUID 生成方便，本地生成没有网络消耗，但是使用起来也有一些缺点，

- **不易于存储**：UUID太长，16字节128位，通常以36长度的字符串表示，很多场景不适用。
- **信息不安全**：基于MAC地址生成UUID的算法可能会造成MAC地址泄露，暴露使用者的位置。
- **对MySQL索引不利**：如果作为数据库主键，在InnoDB引擎下，UUID的无序性可能会引起数据位置频繁变动，严重影响性能，可以查阅 Mysql 索引原理 B+树的知识。

### [#](#数据库生成) 数据库生成

是不是一定要基于外界的条件才能满足分布式唯一ID的需求呢，我们能不能在我们分布式数据库的基础上获取我们需要的ID？

由于分布式数据库的起始自增值一样所以才会有冲突的情况发生，那么我们将分布式系统中数据库的同一个业务表的自增ID设计成不一样的起始值，然后设置固定的步长，步长的值即为分库的数量或分表的数量。

以MySQL举例，利用给字段设置`auto_increment_increment`和`auto_increment_offset`来保证ID自增。

- `auto_increment_offset`：表示自增长字段从那个数开始，他的取值范围是1 .. 65535。
- `auto_increment_increment`：表示自增长字段每次递增的量，其默认值是1，取值范围是1 .. 65535。

假设有三台机器，则DB1中order表的起始ID值为1，DB2中order表的起始值为2，DB3中order表的起始值为3，它们自增的步长都为3，则它们的ID生成范围如下图所示：

![img](https://pdai.tech/images/arch/arch-z-id-2.png)

通过这种方式明显的优势就是依赖于数据库自身不需要其他资源，并且ID号单调自增，可以实现一些对ID有特殊要求的业务。

但是缺点也很明显，首先它**强依赖DB**，当DB异常时整个系统不可用。虽然配置主从复制可以尽可能的增加可用性，但是**数据一致性在特殊情况下难以保证**。主从切换时的不一致可能会导致重复发号。还有就是**ID发号性能瓶颈限制在单台MySQL的读写性能**。

### [#](#使用redis实现) 使用redis实现

Redis实现分布式唯一ID主要是通过提供像 `INCR` 和 `INCRBY` 这样的自增原子命令，由于Redis自身的单线程的特点所以能保证生成的 ID 肯定是唯一有序的。

但是单机存在性能瓶颈，无法满足高并发的业务需求，所以可以采用集群的方式来实现。集群的方式又会涉及到和数据库集群同样的问题，所以也需要设置分段和步长来实现。

为了避免长期自增后数字过大可以通过与当前时间戳组合起来使用，另外为了保证并发和业务多线程的问题可以采用 Redis + Lua的方式进行编码，保证安全。

Redis 实现分布式全局唯一ID，它的性能比较高，生成的数据是有序的，对排序业务有利，但是同样它依赖于redis，**需要系统引进redis组件，增加了系统的配置复杂性**。

当然现在Redis的使用性很普遍，所以如果其他业务已经引进了Redis集群，则可以资源利用考虑使用Redis来实现。

### [#](#雪花算法-snowflake) 雪花算法-Snowflake

Snowflake，雪花算法是由Twitter开源的分布式ID生成算法，以划分命名空间的方式将 64-bit位分割成多个部分，每个部分代表不同的含义。而 Java中64bit的整数是Long类型，所以在 Java 中 SnowFlake 算法生成的 ID 就是 long 来存储的。

- **第1位**占用1bit，其值始终是0，可看做是符号位不使用。
- **第2位**开始的41位是时间戳，41-bit位可表示2^41个数，每个数代表毫秒，那么雪花算法可用的时间年限是`(1L<<41)/(1000L360024*365)`=69 年的时间。
- **中间的10-bit位**可表示机器数，即2^10 = 1024台机器，但是一般情况下我们不会部署这么台机器。如果我们对IDC（互联网数据中心）有需求，还可以将 10-bit 分 5-bit 给 IDC，分5-bit给工作机器。这样就可以表示32个IDC，每个IDC下可以有32台机器，具体的划分可以根据自身需求定义。
- **最后12-bit位**是自增序列，可表示2^12 = 4096个数。

这样的划分之后相当于**在一毫秒一个数据中心的一台机器上可产生4096个有序的不重复的ID**。但是我们 IDC 和机器数肯定不止一个，所以毫秒内能生成的有序ID数是翻倍的。

![img](https://pdai.tech/images/arch/arch-z-id-3.png)

Snowflake 的Twitter官方原版是用Scala写的，对Scala语言有研究的同学可以去阅读下，以下是 Java 版本的写法。

```java
package com.jajian.demo.distribute;

/**
 * Twitter_Snowflake<br>
 * SnowFlake的结构如下(每部分用-分开):<br>
 * 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000 <br>
 * 1位标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负数是1，所以id一般是正数，最高位是0<br>
 * 41位时间截(毫秒级)，注意，41位时间截不是存储当前时间的时间截，而是存储时间截的差值（当前时间截 - 开始时间截)
 * 得到的值），这里的的开始时间截，一般是我们的id生成器开始使用的时间，由我们程序来指定的（如下下面程序IdWorker类的startTime属性）。41位的时间截，可以使用69年，年T = (1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69<br>
 * 10位的数据机器位，可以部署在1024个节点，包括5位datacenterId和5位workerId<br>
 * 12位序列，毫秒内的计数，12位的计数顺序号支持每个节点每毫秒(同一机器，同一时间截)产生4096个ID序号<br>
 * 加起来刚好64位，为一个Long型。<br>
 * SnowFlake的优点是，整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞(由数据中心ID和机器ID作区分)，并且效率较高，经测试，SnowFlake每秒能够产生26万ID左右。
 */
public class SnowflakeDistributeId {


    // ==============================Fields===========================================
    /**
     * 开始时间截 (2015-01-01)
     */
    private final long twepoch = 1420041600000L;

    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 5L;

    /**
     * 数据标识id所占的位数
     */
    private final long datacenterIdBits = 5L;

    /**
     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
     */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);

    /**
     * 支持的最大数据标识id，结果是31
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    /**
     * 序列在id中占的位数
     */
    private final long sequenceBits = 12L;

    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;

    /**
     * 数据标识id向左移17位(12+5)
     */
    private final long datacenterIdShift = sequenceBits + workerIdBits;

    /**
     * 时间截向左移22位(5+5+12)
     */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);

    /**
     * 工作机器ID(0~31)
     */
    private long workerId;

    /**
     * 数据中心ID(0~31)
     */
    private long datacenterId;

    /**
     * 毫秒内序列(0~4095)
     */
    private long sequence = 0L;

    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;

    //==============================Constructors=====================================

    /**
     * 构造函数
     *
     * @param workerId     工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public SnowflakeDistributeId(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }

    // ==============================Methods==========================================

    /**
     * 获得下一个ID (该方法是线程安全的)
     *
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        //如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }

        //上次生成ID的时间截
        lastTimestamp = timestamp;

        //移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - twepoch) << timestampLeftShift) //
                | (datacenterId << datacenterIdShift) //
                | (workerId << workerIdShift) //
                | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     *
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    /**
     * 返回以毫秒为单位的当前时间
     *
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }
}
```

测试的代码如下

```java
public static void main(String[] args) {
    SnowflakeDistributeId idWorker = new SnowflakeDistributeId(0, 0);
    for (int i = 0; i < 1000; i++) {
        long id = idWorker.nextId();
//      System.out.println(Long.toBinaryString(id));
        System.out.println(id);
    }
}
```

**雪花算法提供了一个很好的设计思想，雪花算法生成的ID是趋势递增，不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的，而且可以根据自身业务特性分配bit位，非常灵活**。

但是雪花算法强**依赖机器时钟**，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态。如果恰巧回退前生成过一些ID，而时间回退后，生成的ID就有可能重复。官方对于此并没有给出解决方案，而是简单的抛错处理，这样会造成在时间被追回之前的这段时间服务不可用。

很多其他类雪花算法也是在此思想上的设计然后改进规避它的缺陷，后面介绍的`百度 UidGenerator` 和 `美团分布式ID生成系统 Leaf` 中snowflake模式都是在 snowflake 的基础上演进出来的。

### [#](#百度-uidgenerator) 百度-UidGenerator

> 百度的 `UidGenerator` 是百度开源基于Java语言实现的唯一ID生成器，是在雪花算法 snowflake 的基础上做了一些改进。`UidGenerator`以组件形式工作在应用项目中, 支持自定义workerId位数和初始化策略，适用于docker等虚拟化环境下实例自动重启、漂移等场景。

在实现上，UidGenerator 提供了两种生成唯一ID方式，分别是 `DefaultUidGenerator` 和 `CachedUidGenerator`，官方建议如果有**性能考虑**的话使用 `CachedUidGenerator` 方式实现。

`UidGenerator` 依然是以划分命名空间的方式将 64-bit位分割成多个部分，只不过它的默认划分方式有别于雪花算法 snowflake。它默认是由 `1-28-22-13` 的格式进行划分。可根据你的业务的情况和特点，自己调整各个字段占用的位数。

- **第1位**仍然占用1bit，其值始终是0。
- **第2位**开始的28位是时间戳，28-bit位可表示2^28个数，这里不再是以毫秒而是以秒为单位，每个数代表秒则可用`（1L<<28）/ (360024365) ≈ 8.51` 年的时间。
- 中间的 workId （数据中心+工作机器，可以其他组成方式）则由 **22-bit位**组成，可表示 2^22 = 4194304个工作ID。
- 最后由**13-bit位**构成自增序列，可表示2^13 = 8192个数。

![img](https://pdai.tech/images/arch/arch-z-id-4.png)

其中 workId （机器 id），最多可支持约420w次机器启动。**内置实现为在启动时由数据库分配（表名为 WORKER_NODE），默认分配策略为用后即弃，后续可提供复用策略**。

```sql
DROP TABLE IF EXISTS WORKER_NODE;
CREATE TABLE WORKER_NODE
(
ID BIGINT NOT NULL AUTO_INCREMENT COMMENT 'auto increment id',
HOST_NAME VARCHAR(64) NOT NULL COMMENT 'host name',
PORT VARCHAR(64) NOT NULL COMMENT 'port',
TYPE INT NOT NULL COMMENT 'node type: ACTUAL or CONTAINER',
LAUNCH_DATE DATE NOT NULL COMMENT 'launch date',
MODIFIED TIMESTAMP NOT NULL COMMENT 'modified time',
CREATED TIMESTAMP NOT NULL COMMENT 'created time',
PRIMARY KEY(ID)
)
 COMMENT='DB WorkerID Assigner for UID Generator',ENGINE = INNODB;
```

#### [#](#defaultuidgenerator-实现) DefaultUidGenerator 实现

`DefaultUidGenerator` 就是正常的根据时间戳和机器位还有序列号的生成方式，和雪花算法很相似，对于时钟回拨也只是抛异常处理。仅有一些不同，如**以秒为为单位**而不再是毫秒和支持Docker等虚拟化环境。

```java
protected synchronized long nextId() {
    long currentSecond = getCurrentSecond();

    // Clock moved backwards, refuse to generate uid
    if (currentSecond < lastSecond) {
        long refusedSeconds = lastSecond - currentSecond;
        throw new UidGenerateException("Clock moved backwards. Refusing for %d seconds", refusedSeconds);
    }

    // At the same second, increase sequence
    if (currentSecond == lastSecond) {
        sequence = (sequence + 1) & bitsAllocator.getMaxSequence();
        // Exceed the max sequence, we wait the next second to generate uid
        if (sequence == 0) {
            currentSecond = getNextSecond(lastSecond);
        }

    // At the different second, sequence restart from zero
    } else {
        sequence = 0L;
    }

    lastSecond = currentSecond;

    // Allocate bits for UID
    return bitsAllocator.allocate(currentSecond - epochSeconds, workerId, sequence);
}
```

如果你要使用 DefaultUidGenerator 的实现方式的话，以上划分的占用位数可通过 spring 进行参数配置。

```xml
<bean id="defaultUidGenerator" class="com.baidu.fsg.uid.impl.DefaultUidGenerator" lazy-init="false">
    <property name="workerIdAssigner" ref="disposableWorkerIdAssigner"/>

    <!-- Specified bits & epoch as your demand. No specified the default value will be used -->
    <property name="timeBits" value="29"/>
    <property name="workerBits" value="21"/>
    <property name="seqBits" value="13"/>
    <property name="epochStr" value="2016-09-20"/>
</bean>
```

#### [#](#cacheduidgenerator-实现) CachedUidGenerator 实现

而官方建议的性能较高的 `CachedUidGenerator` 生成方式，是使用 RingBuffer 缓存生成的id。数组每个元素成为一个slot。RingBuffer容量，默认为Snowflake算法中sequence最大值（2^13 = 8192）。可通过 boostPower 配置进行扩容，以提高 RingBuffer 读写吞吐量。

Tail指针、Cursor指针用于环形数组上读写slot：

- **Tail指针** 表示Producer生产的最大序号(此序号从0开始，持续递增)。Tail不能超过Cursor，即生产者不能覆盖未消费的slot。当Tail已赶上curosr，此时可通过rejectedPutBufferHandler指定PutRejectPolicy
- **Cursor指针** 表示Consumer消费到的最小序号(序号序列与Producer序列相同)。Cursor不能超过Tail，即不能消费未生产的slot。当Cursor已赶上tail，此时可通过rejectedTakeBufferHandler指定TakeRejectPolicy

![img](/images/arch/arch-z-id-5.png)

CachedUidGenerator采用了双RingBuffer，Uid-RingBuffer用于存储Uid、Flag-RingBuffer用于存储Uid状态(是否可填充、是否可消费)。

由于数组元素在内存中是连续分配的，可最大程度利用CPU cache以提升性能。但同时会带来「伪共享」FalseSharing问题，为此在Tail、Cursor指针、Flag-RingBuffer中采用了CacheLine 补齐方式。

![img](/images/arch/arch-z-id-6.png)

**RingBuffer填充时机**

- **初始化预填充** RingBuffer初始化时，预先填充满整个RingBuffer。
- **即时填充** Take消费时，即时检查剩余可用slot量(tail - cursor)，如小于设定阈值，则补全空闲slots。阈值可通过paddingFactor来进行配置，请参考Quick Start中CachedUidGenerator配置。
- **周期填充** 通过Schedule线程，定时补全空闲slots。可通过scheduleInterval配置，以应用定时填充功能，并指定Schedule时间间隔。

### [#](#美团leaf) 美团Leaf

> Leaf是美团基础研发平台推出的一个分布式ID生成服务，名字取自德国哲学家、数学家莱布尼茨的著名的一句话：“There are no two identical leaves in the world”，世间不可能存在两片相同的叶子。

Leaf 也提供了两种ID生成的方式，分别是 `Leaf-segment 数据库方案`和 `Leaf-snowflake 方案`。

#### [#](#leaf-segment-数据库方案) Leaf-segment 数据库方案

Leaf-segment 数据库方案，是在上文描述的在使用数据库的方案上，做了如下改变：

- 原方案每次获取ID都得读写一次数据库，造成数据库压力大。改为利用proxy server批量获取，每次获取一个segment(step决定大小)号段的值。用完之后再去数据库获取新的号段，可以大大的减轻数据库的压力。
- 各个业务不同的发号需求用 `biz_tag`字段来区分，每个biz-tag的ID获取相互隔离，互不影响。如果以后有性能需求需要对数据库扩容，不需要上述描述的复杂的扩容操作，只需要对biz_tag分库分表就行。

数据库表设计如下：

```sql
CREATE TABLE `leaf_alloc` (
  `biz_tag` varchar(128)  NOT NULL DEFAULT '' COMMENT '业务key',
  `max_id` bigint(20) NOT NULL DEFAULT '1' COMMENT '当前已经分配了的最大id',
  `step` int(11) NOT NULL COMMENT '初始步长，也是动态调整的最小步长',
  `description` varchar(256)  DEFAULT NULL COMMENT '业务key的描述',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`biz_tag`)
) ENGINE=InnoDB;
```

原来获取ID每次都需要写数据库，现在只需要把step设置得足够大，比如1000。那么只有当1000个号被消耗完了之后才会去重新读写一次数据库。读写数据库的频率从1减小到了1/step，大致架构如下图所示：

![img](/images/arch/arch-z-id-7.png)

同时Leaf-segment 为了解决 TP999（满足千分之九百九十九的网络请求所需要的最低耗时）数据波动大，当号段使用完之后还是会hang在更新数据库的I/O上，TP999 数据会出现偶尔的尖刺的问题，提供了双buffer优化。

简单的说就是，Leaf 取号段的时机是在号段消耗完的时候进行的，也就意味着号段临界点的ID下发时间取决于下一次从DB取回号段的时间，并且在这期间进来的请求也会因为DB号段没有取回来，导致线程阻塞。如果请求DB的网络和DB的性能稳定，这种情况对系统的影响是不大的，但是假如取DB的时候网络发生抖动，或者DB发生慢查询就会导致整个系统的响应时间变慢。

为了DB取号段的过程能够做到无阻塞，不需要在DB取号段的时候阻塞请求线程，即当号段消费到某个点时就异步的把下一个号段加载到内存中，而不需要等到号段用尽的时候才去更新号段。这样做就可以很大程度上的降低系统的 TP999 指标。详细实现如下图所示：

![img](/images/arch/arch-z-id-8.png)

采用双buffer的方式，Leaf服务内部有两个号段缓存区segment。当前号段已下发10%时，如果下一个号段未更新，则另启一个更新线程去更新下一个号段。当前号段全部下发完后，如果下个号段准备好了则切换到下个号段为当前segment接着下发，循环往复。

- 每个biz-tag都有消费速度监控，通常推荐segment长度设置为服务高峰期发号QPS的600倍（10分钟），这样即使DB宕机，Leaf仍能持续发号10-20分钟不受影响。
- 每次请求来临时都会判断下个号段的状态，从而更新此号段，所以偶尔的网络抖动不会影响下个号段的更新。

对于这种方案依然存在一些问题，它**仍然依赖 DB的稳定性，需要采用主从备份的方式提高 DB的可用性**，还有 Leaf-segment方案生成的ID是趋势递增的，这样ID号是可被计算的，例如订单ID生成场景，**通过订单id号相减就能大致计算出公司一天的订单量，这个是不能忍受的**。

#### [#](#leaf-snowflake方案) Leaf-snowflake方案

Leaf-snowflake方案完全沿用 snowflake 方案的bit位设计，对于workerID的分配引入了Zookeeper持久顺序节点的特性自动对snowflake节点配置 wokerID。避免了服务规模较大时，动手配置成本太高的问题。

Leaf-snowflake是按照下面几个步骤启动的：

- 启动Leaf-snowflake服务，连接Zookeeper，在leaf_forever父节点下检查自己是否已经注册过（是否有该顺序子节点）。
- 如果有注册过直接取回自己的workerID（zk顺序节点生成的int类型ID号），启动服务。
- 如果没有注册过，就在该父节点下面创建一个持久顺序节点，创建成功后取回顺序号当做自己的workerID号，启动服务。

![img](/images/arch/arch-z-id-9.png)

为了减少对 Zookeeper的依赖性，会在本机文件系统上缓存一个workerID文件。当ZooKeeper出现问题，恰好机器出现问题需要重启时，能保证服务能够正常启动。

上文阐述过在类 snowflake算法上都存在时钟回拨的问题，Leaf-snowflake在解决时钟回拨的问题上是通过校验自身系统时间与 `leaf_forever/${self}`节点记录时间做比较然后启动报警的措施。

![img](/images/arch/arch-z-id-10.png)

美团官方建议是由于强依赖时钟，对时间的要求比较敏感，**在机器工作时NTP同步也会造成秒级别的回退，建议可以直接关闭NTP同步。要么在时钟回拨的时候直接不提供服务直接返回ERROR_CODE，等时钟追上即可。或者做一层重试，然后上报报警系统，更或者是发现有时钟回拨之后自动摘除本身节点并报警。**

在性能上官方提供的数据目前 Leaf 的性能在4C8G 的机器上QPS能压测到近5w/s，TP999 1ms。

### [#](#mist-薄雾算法) Mist 薄雾算法

> 最近有个号称超过snowflake 587倍的ID生成算法，可以参看[这里在新窗口打开](https://juejin.im/post/6846687584324681735#heading-5), 如下内容摘自 [GitHub中项目README在新窗口打开](https://github.com/asyncins/mist/tree/master)

薄雾算法是不同于 snowflake 的全局唯一 ID 生成算法。相比 snowflake ，薄雾算法具有更高的数值上限和更长的使用期限。

现在薄雾算法拥有比雪花算法更高的性能！

#### [#](#考量了什么业务场景和要求呢) 考量了什么业务场景和要求呢？

用到全局唯一 ID 的场景不少，这里引用美团 Leaf 的场景介绍:

> 在复杂分布式系统中，往往需要对大量的数据和消息进行唯一标识。如在美团点评的金融、支付、餐饮、酒店、猫眼电影等产品的系统中，数据日渐增长，对数据分库分表后需要有一个唯一 ID 来标识一条数据或消息，数据库的自增 ID 显然不能满足需求；特别一点的如订单、骑手、优惠券也都需要有唯一 ID 做标识。此时一个能够生成全局唯一ID 的系统是非常必要的。

引用微信 seqsvr 的场景介绍：

> 微信在立项之初，就已确立了利用数据版本号实现终端与后台的数据增量同步机制，确保发消息时消息可靠送达对方手机。

爬虫数据服务的场景介绍：

> 数据来源各不相同，且并发极大的情况下难以生成统一的数据编号，同时数据编号又将作为爬虫下游整个链路的溯源依据，在爬虫业务链路中十分重要。

这里参考美团 [Leaf在新窗口打开](https://tech.meituan.com/2017/04/21/mt-leaf.html) 的要求：

1、全局唯一性：不能出现重复的 ID 号，既然是唯一标识，这是最基本的要求；

2、趋势递增：在 MySQL InnoDB 引擎中使用的是聚集索引，由于多数 RDBMS 使用 B-tree 的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键保证写入性能；

3、单调递增：保证下一个 ID 一定大于上一个 ID，例如事务版本号、IM 增量消息、排序等特殊需求；

4、信息安全：如果 ID 是连续的，恶意用户的爬取工作就非常容易做了，直接按照顺序下载指定 URL 即可；如果是订单号就更危险了，竞对可以直接知道我们一天的单量。所以在一些应用场景下，会需要 ID 无规则、不规则；

可以用“全局不重复，不可猜测且呈递增态势”这句话来概括描述要求。

#### [#](#薄雾算法的设计思路是怎么样的) 薄雾算法的设计思路是怎么样的？

薄雾算法采用了与 snowflake 相同的位数——64，在考量业务场景和要求后并没有沿用 1-41-10-12 的占位，而是采用了 1-47-8-8 的占位。即：

```text
* 1      2                                                     48         56       64
* +------+-----------------------------------------------------+----------+----------+
* retain | increas                                             | salt     | salt |
* +------+-----------------------------------------------------+----------+----------+
* 0      | 0000000000 0000000000 0000000000 0000000000 0000000 | 00000000 | 00000000 |
* +------+-----------------------------------------------------+------------+--------+
```

- 第一段为最高位，占 1 位，保持为 0，使得值永远为正数；
- 第二段放置自增数，占 47 位，自增数在高位能保证结果值呈递增态势，遂低位可以为所欲为；
- 第三段放置随机因子一，占 8 位，上限数值 255，使结果值不可预测；
- 第四段放置随机因子二，占 8 位，上限数值 255，使结果值不可预测；

#### [#](#薄雾算法生成的数值是什么样的) 薄雾算法生成的数值是什么样的？

薄雾自增数为 1～10 的运行结果类似如下：

```text
171671
250611
263582
355598
427749
482010
581550
644278
698636
762474
```

根据运行结果可知，薄雾算法能够满足“全局不重复，不可猜测且呈递增态势”的场景要求。

#### [#](#薄雾算法-mist-和雪花算法-snowflake-有何区别) 薄雾算法 mist 和雪花算法 snowflake 有何区别？

snowflake 是由 Twitter 公司提出的一种全局唯一 ID 生成算法，它具有“递增态势、不依赖数据库、高性能”等特点，自 snowflake 推出以来备受欢迎，算法被应用于大大小小公司的服务中。snowflake 高位为时间戳的二进制，遂完全受到时间戳的影响，倘若时间回拨（当前服务器时间回到之前的某一时刻），那么 snowflake 有极大概率生成与之前同一时刻的重复 ID，这直接影响整个业务。

snowflake 受时间戳影响，使用上限不超过 70 年。

薄雾算法 Mist 由书籍《Python3 反爬虫原理与绕过实战》的作者韦世东综合 [百度 UidGenerator在新窗口打开](https://github.com/baidu/uid-generator)、 [美团 Leaf在新窗口打开](https://tech.meituan.com/2017/04/21/mt-leaf.html) 和 [微信序列号生成器 seqsvr在新窗口打开](https://www.infoq.cn/article/wechat-serial-number-generator-architecture) 中介绍的技术点，同时考虑高性能分布式序列号生成器架构后设计的一款“递增态势、不依赖数据库、高性能且不受时间回拨影响”的全局唯一序列号生成算法。

![mistSturct](http://can.sfhfpc.com/7798.png)

薄雾算法不受时间戳影响，受到数值大小影响。薄雾算法高位数值上限计算方式为`int64(1<<47 - 1)`，上限数值`140737488355327` 百万亿级，假设每天消耗 10 亿，薄雾算法能使用 385+ 年。

#### [#](#为什么薄雾算法不受时间回拨影响) 为什么薄雾算法不受时间回拨影响？

snowflake 受时间回拨影响的根本原因是高位采用时间戳的二进制值，而薄雾算法的高位是按序递增的数值。结果值的大小由高位决定，遂薄雾算法不受时间回拨影响。

[#](#为什么说薄雾算法的结果值不可预测) 为什么说薄雾算法的结果值不可预测？

考虑到“不可预测”的要求，薄雾算法的中间位是 8 位随机值，且末 8 位是也是随机值，两组随机值大大增加了预测难度，因此称为结果值不可预测。

中间位和末位随机值的开闭区间都是 [0, 255]，理论上随机值可以出现 `256 * 256` 种组合。

#### [#](#当程序重启-薄雾算法的值会重复吗) 当程序重启，薄雾算法的值会重复吗？

snowflake 受时间回拨影响，一旦时间回拨就有极大概率生成重复的 ID。薄雾算法中的高位是按序递增的数值，程序重启会造成按序递增数值回到初始值，但由于中间位和末尾随机值的影响，因此不是必定生成（有大概率生成）重复 ID，但递增态势必定受到影响。

#### [#](#薄雾算法的值会重复-那我要它干嘛) 薄雾算法的值会重复，那我要它干嘛？

1、无论是什么样的全局唯一 ID 生成算法，都会有优点和缺点。在实际的应用当中，没有人会将全局唯一 ID 生成算法完全托付给程序，而是会用数据库存储关键值或者所有生成的值。全局唯一 ID 生成算法大多都采用分布式架构或者主备架构提供发号服务，这时候就不用担心它的重复问题；

2、生成性能比雪花算法高太多倍；

3、代码少且简单，在大型应用中，单功能越简单越好；

### [#](#是否提供薄雾算法的工程实践或者架构实践) 是否提供薄雾算法的工程实践或者架构实践？

是的，作者的另一个项目 [Medis在新窗口打开](https://github.com/asyncins/medis) 是薄雾算法与 Redis 的结合，实现了“全局不重复”，你再也不用担心程序重启带来的问题。

#### [#](#薄雾算法的分布式架构-推荐-cp-还是-ap) 薄雾算法的分布式架构，推荐 CP 还是 AP？

CAP 是分布式架构中最重要的理论，C 指的是一致性、A 指的是可用性、P 指的是分区容错性。CAP 当中，C 和 A 是互相冲突的，且 P 一定存在，遂我们必须在 CP 和 AP 中选择。**实际上这跟具体的业务需求有关**，但是对于全局唯一 ID 发号服务来说，大多数时候可用性比一致性更重要，也就是选择 AP 会多过选择 CP。至于你怎么选，还是得结合具体的业务场景考虑。

[#](#薄雾算法的性能测试) 薄雾算法的性能测试

采用 Golnag（1.14） 自带的 Benchmark 进行测试，测试机硬件环境如下：

```text
内存 16 GB 2133 MHz LPDDR3
处理器 2.3 GHz 双核Intel Core i5
操作系统 macOS Catalina
机器 MacBook Pro (13-inch, 2017, Two Thunderbolt 3 ports)
```

进行了多轮测试，随机取 3 轮测试结果。以此计算平均值，得 `单次执行时间 346 ns/op`。以下是随机 3 轮测试的结果：

```text
goos: darwin
goarch: amd64
pkg: mist
BenchmarkMain-4          3507442               339 ns/op
PASS
ok      mist    1.345s
goos: darwin
goarch: amd64
pkg: mist
BenchmarkMain-4          3488708               338 ns/op
PASS
ok      mist    1.382s
goos: darwin
goarch: amd64
pkg: mist
BenchmarkMain-4          3434936               360 ns/op
PASS
ok      mist    1.394s
```

### [#](#总结) 总结

以上基本列出了所有常用的分布式ID生成方式，其实大致分类的话可以分为两类：

- **一种是类DB型的**，根据设置不同起始值和步长来实现趋势递增，需要考虑服务的容错性和可用性。
- **另一种是类snowflake型**，这种就是将64位划分为不同的段，每段代表不同的涵义，基本就是时间戳、机器ID和序列数。这种方案就是需要考虑时钟回拨的问题以及做一些 buffer的缓冲设计提高性能。

而且可通过将三者（时间戳，机器ID，序列数）划分不同的位数来改变使用寿命和并发数。

例如对于并发数要求不高、期望长期使用的应用，可增加时间戳位数，减少序列数的位数. 例如配置成`{"workerBits":23,"timeBits":31,"seqBits":9}`时, 可支持28个节点以整体并发量14400 UID/s的速度持续运行68年。

对于节点重启频率频繁、期望长期使用的应用, 可增加工作机器位数和时间戳位数, 减少序列数位数. 例如配置成`{"workerBits":27,"timeBits":30,"seqBits":6}`时, 可支持37个节点以整体并发量2400 UID/s的速度持续运行34年。



------

著作权归@pdai所有 原文链接：https://pdai.tech/md/arch/arch-z-id.html





## 分布式系统 - 分布式锁及实现方案

> 本文主要介绍分布式锁的概念和分布式锁的设计原则，以及常见的分布式锁的实现方式。@pdai

[什么是分布式锁]()[分布式锁的设计原则]()[分布式锁的实现方案]()[基于数据库如何实现分布式锁？有什么缺陷？]()[基于数据库表（锁表，很少使用）]()[基于悲观锁]()[基于乐观锁]()[基于redis如何实现分布式锁？有什么缺陷？]()[set NX PX + Lua]()[基于RedLock实现分布式锁]()[基于Redis的客户端]()[进一步理解]()[基于zookeeper如何实现分布式锁？]()[参考文章]()

### [#](#什么是分布式锁) 什么是分布式锁

> 要介绍分布式锁，首先要提到与分布式锁相对应的是线程锁、进程锁。

- **线程锁**：主要用来给方法、代码块加锁。当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或该代码段。线程锁只在同一JVM中有效果，因为线程锁的实现在根本上是依靠线程之间共享内存实现的，比如synchronized是共享对象头，显示锁Lock是共享某个变量（state）。
- **进程锁**：为了控制同一操作系统中多个进程访问某个共享资源，因为进程具有独立性，各个进程无法访问其他进程的资源，因此无法通过synchronized等线程锁实现进程锁。
- **分布式锁**：当多个进程不在同一个系统中(比如分布式系统中控制共享资源访问)，用分布式锁控制多个进程对资源的访问。

#### [#](#分布式锁的设计原则) 分布式锁的设计原则

> 分布式锁的最小设计原则：**安全性**和**有效性**

[Redis的官网在新窗口打开](https://redis.io/docs/reference/patterns/distributed-locks/)上对使用分布式锁提出至少需要满足如下三个要求：

1. **互斥**（属于安全性）：在任何给定时刻，只有一个客户端可以持有锁。
2. **无死锁**（属于有效性）：即使锁定资源的客户端崩溃或被分区，也总是可以获得锁；通常通过超时机制实现。
3. **容错性**（属于有效性）：只要大多数 Redis 节点都启动，客户端就可以获取和释放锁。

除此之外，分布式锁的设计中还可以/需要考虑：

1. 加锁解锁的**同源性**：A加的锁，不能被B解锁
2. 获取锁是**非阻塞**的：如果获取不到锁，不能无限期等待；
3. **高性能**：加锁解锁是高性能的

#### [#](#分布式锁的实现方案) 分布式锁的实现方案

> 就体系的角度而言，谈谈常见的分布式锁的实现方案。

- 基于数据库实现分布式锁

  - 基于数据库表（锁表，很少使用）
  - 乐观锁(基于版本号)
  - 悲观锁(基于排它锁)

- 基于 redis 实现分布式锁

  : 

  - 单个Redis实例：setnx(key,当前时间+过期时间) + Lua
  - Redis集群模式：Redlock

- 基于 zookeeper实现分布式锁

  - 临时有序节点来实现的分布式锁,Curator

- **基于 Consul 实现分布式锁**

### [#](#基于数据库如何实现分布式锁-有什么缺陷) 基于数据库如何实现分布式锁？有什么缺陷？

> 基于数据库如何实现分布式锁？有什么缺陷？

#### [#](#基于数据库表-锁表-很少使用) 基于数据库表（锁表，很少使用）

最简单的方式可能就是直接创建一张锁表，然后通过操作该表中的数据来实现了。当我们想要获得锁的时候，就可以在该表中增加一条记录，想要释放锁的时候就删除这条记录。

为了更好的演示，我们先创建一张数据库表，参考如下：

```sql
CREATE TABLE database_lock (
	`id` BIGINT NOT NULL AUTO_INCREMENT,
	`resource` int NOT NULL COMMENT '锁定的资源',
	`description` varchar(1024) NOT NULL DEFAULT "" COMMENT '描述',
	PRIMARY KEY (id),
	UNIQUE KEY uiq_idx_resource (resource)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='数据库分布式锁表';
```

当我们想要获得锁时，可以插入一条数据：

```sql
INSERT INTO database_lock(resource, description) VALUES (1, 'lock');
```

当需要释放锁的时，可以删除这条数据：

```sql
DELETE FROM database_lock WHERE resource=1;
```

#### [#](#基于悲观锁) 基于悲观锁

**悲观锁实现思路**？

1. 在对任意记录进行修改前，先尝试为该记录加上排他锁（exclusive locking）。
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。 具体响应方式由开发者根据实际需要决定。
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
4. 其间如果有其他对该记录做修改或加排他锁的操作，都会等待我们解锁或直接抛出异常。

**以MySQL InnoDB中使用悲观锁为例**？

要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。set autocommit=0;

```sql
//0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
//1.查询出商品信息
select status from t_goods where id=1 for update;
//2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_goods set status=2;
//4.提交事务
commit;/commit work;
```

上面的查询语句中，我们使用了`select…for update`的方式，这样就通过开启排他锁的方式实现了悲观锁。此时在t_goods表中，id为1的 那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

上面我们提到，使用`select…for update`会把数据给锁住，不过我们需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。

#### [#](#基于乐观锁) 基于乐观锁

乐观并发控制（又名“乐观锁”，Optimistic Concurrency Control，缩写“OCC”）是一种并发控制的方法。它假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。

**以使用版本号实现乐观锁为例？**

使用版本号时，可以在数据初始化时指定一个版本号，每次对数据的更新操作都对版本号执行+1操作。并判断当前版本号是不是该数据的最新的版本号。

```sql
1.查询出商品信息
select (status,status,version) from t_goods where id=#{id}
2.根据商品信息生成订单
3.修改商品status为2
update t_goods 
set status=2,version=version+1
where id=#{id} and version=#{version};
```

需要注意的是，乐观锁机制往往基于系统中数据存储逻辑，因此也具备一定的局限性。由于乐观锁机制是在我们的系统中实现的，对于来自外部系统的用户数据更新操作不受我们系统的控制，因此可能会造成脏数据被更新到数据库中。在系统设计阶段，我们应该充分考虑到这些情况，并进行相应的调整（如将乐观锁策略在数据库存储过程中实现，对外只开放基于此存储过程的数据更新途径，而不是将数据库表直接对外公开）。

- **缺陷**

对数据库依赖，开销问题，行锁变表锁问题，无法解决数据库单点和可重入的问题。

### [#](#基于redis如何实现分布式锁-有什么缺陷) 基于redis如何实现分布式锁？有什么缺陷？

> 基于redis如何实现分布式锁？这里一定要看[Redis的官网在新窗口打开](https://redis.io/docs/reference/patterns/distributed-locks/)的分布式锁的实现这篇文章。

#### [#](#set-nx-px-lua) set NX PX + Lua

**加锁**： set NX PX + 重试 + 重试间隔

向Redis发起如下命令: `SET productId:lock 0xx9p03001 NX PX 30000` 其中，"productId"由自己定义，可以是与本次业务有关的id，"0xx9p03001"是一串随机值，必须保证全局唯一(原因在后文中会提到)，“NX"指的是当且仅当key(也就是案例中的"productId:lock”)在Redis中不存在时，返回执行成功，否则执行失败。"PX 30000"指的是在30秒后，key将被自动删除。执行命令后返回成功，表明服务成功的获得了锁。

```java
@Override
public boolean lock(String key, long expire, int retryTimes, long retryDuration) {
    // use JedisCommands instead of setIfAbsense
    boolean result = setRedis(key, expire);

    // retry if needed
    while ((!result) && retryTimes-- > 0) {
        try {
            log.debug("lock failed, retrying..." + retryTimes);
            Thread.sleep(retryDuration);
        } catch (Exception e) {
            return false;
        }

        // use JedisCommands instead of setIfAbsense
        result = setRedis(key, expire);
    }
    return result;
}

private boolean setRedis(String key, long expire) {
    try {
        RedisCallback<String> redisCallback = connection -> {
            JedisCommands commands = (JedisCommands) connection.getNativeConnection();
            String uuid = SnowIDUtil.uniqueStr();
            lockFlag.set(uuid);
            return commands.set(key, uuid, NX, PX, expire); // 看这里
        };
        String result = redisTemplate.execute(redisCallback);
        return !StringUtil.isEmpty(result);
    } catch (Exception e) {
        log.error("set redis occurred an exception", e);
    }
    return false;
}
```

**解锁**：采用lua脚本

在删除key之前，一定要判断服务A持有的value与Redis内存储的value是否一致。如果贸然使用服务A持有的key来删除锁，则会误将服务B的锁释放掉。

```lua
if redis.call("get", KEYS[1])==ARGV[1] then
	return redis.call("del", KEYS[1])
else
	return 0
end
```

#### [#](#基于redlock实现分布式锁) 基于RedLock实现分布式锁

> 这是Redis作者推荐的分布式集群情况下的方式，请看这篇文章[Is Redlock safe?在新窗口打开](http://antirez.com/news/101)

假设有两个服务A、B都希望获得锁，有一个包含了5个redis master的Redis Cluster，执行过程大致如下:

1. 客户端获取当前时间戳，单位: 毫秒
2. 服务A轮寻每个master节点，尝试创建锁。(这里锁的过期时间比较短，一般就几十毫秒) RedLock算法会尝试在大多数节点上分别创建锁，假如节点总数为n，那么大多数节点指的是n/2+1。
3. 客户端计算成功建立完锁的时间，如果建锁时间小于超时时间，就可以判定锁创建成功。如果锁创建失败，则依次(遍历master节点)删除锁。
4. 只要有其它服务创建过分布式锁，那么当前服务就必须轮寻尝试获取锁。

[#](#基于redis的客户端) 基于Redis的客户端

> 这里Redis的客户端（Jedis, Redisson, Lettuce等）都是基于上述两类形式来实现分布式锁的，只是两类形式的封装以及一些优化（比如Redisson的watch dog)。

以基于Redisson实现分布式锁为例（支持了 单实例、Redis哨兵、redis cluster、redis master-slave等各种部署架构）：

**特色**？

1. redisson所有指令都通过lua脚本执行，保证了操作的原子性
2. redisson设置了watchdog看门狗，“看门狗”的逻辑保证了没有死锁发生
3. redisson支持Redlock的实现方式。

**过程**？

1. 线程去获取锁，获取成功: 执行lua脚本，保存数据到redis数据库。
2. 线程去获取锁，获取失败: 订阅了解锁消息，然后再尝试获取锁，获取成功后，执行lua脚本，保存数据到redis数据库。

**互斥**？

如果这个时候客户端B来尝试加锁，执行了同样的一段lua脚本。第一个if判断会执行“exists myLock”，发现myLock这个锁key已经存在。接着第二个if判断，判断myLock锁key的hash数据结构中，是否包含客户端B的ID，但明显没有，那么客户端B会获取到pttl myLock返回的一个数字，代表myLock这个锁key的剩余生存时间。此时客户端B会进入一个while循环，不听的尝试加锁。

**watch dog自动延时机制**？

客户端A加锁的锁key默认生存时间只有30秒，如果超过了30秒，客户端A还想一直持有这把锁，怎么办？其实只要客户端A一旦加锁成功，就会启动一个watch dog看门狗，它是一个后台线程，会每隔10秒检查一下，如果客户端A还持有锁key，那么就会不断的延长锁key的生存时间。

**可重入**？

每次lock会调用incrby，每次unlock会减一。

#### [#](#进一步理解) 进一步理解

1. 借助Redis实现分布式锁时，有一个共同的缺陷: 当获取锁被拒绝后，需要不断的循环，重新发送获取锁(创建key)的请求，直到请求成功。这就造成空转，浪费宝贵的CPU资源。
2. RedLock算法本身有争议，具体看这篇文章[How to do distributed locking在新窗口打开](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html) 以及作者的回复[Is Redlock safe?在新窗口打开](http://antirez.com/news/101)

### [#](#基于zookeeper如何实现分布式锁) 基于zookeeper如何实现分布式锁？

说几个核心点：

- **顺序节点**

创建一个用于发号的节点“/test/lock”，然后以它为父亲节点的前缀为“/test/lock/seq-”依次发号：

![img](https://pdai.tech/images/zk/zk-1.png)

- **获得最小号得锁**

由于序号的递增性，可以规定排号最小的那个获得锁。所以，每个线程在尝试占用锁之前，首先判断自己是排号是不是当前最小，如果是，则获取锁。

- **节点监听机制**

每个线程抢占锁之前，先抢号创建自己的ZNode。同样，释放锁的时候，就需要删除抢号的Znode。抢号成功后，如果不是排号最小的节点，就处于等待通知的状态。等谁的通知呢？不需要其他人，只需要等前一个Znode 的通知就可以了。当前一个Znode 删除的时候，就是轮到了自己占有锁的时候。第一个通知第二个、第二个通知第三个，击鼓传花似的依次向后。

## [#](#参考文章) 参考文章

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/arch/arch-z-lock.html