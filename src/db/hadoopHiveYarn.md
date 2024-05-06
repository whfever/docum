## Hadoop

### HDFS 读写流程

```

写入数据：
（1）客户端向NameNode发送文件写入请求，NameNode返回可写的DataNode列表。

（2）客户端选择一个DataNode作为写入目标，并向其发送写入请求。

（3）DataNode向客户端确认并创建一个临时文件存储数据。

（4）客户端将数据分成一个个的数据块，并将它们按照顺序写入DataNode。

（5）DataNode接收数据并存储在本地磁盘上，同时将数据块的副本复制到其他DataNode上。

（6）客户端在最后一个数据块写入完成后，向NameNode发送一个关闭文件的请求。

（7）NameNode更新文件的元数据，并向客户端返回写入完成的确认信息。

读取数据：
（1）客户端向NameNode发送文件读取请求。

（2）NameNode返回包含文件块信息的DataNode列表。

（3）客户端选择一个DataNode，并向其发送读取请求。

（4）DataNode向客户端确认请求并返回所请求数据块的数据。

（5）如果客户端需要读取的数据块不在所选择的DataNode上，则重复（3）和（4）步骤，直到获取所需数据块。

（6）客户端接收数据并将其合并，形成完整的文件数据。

需要注意的是，HDFS采用了多副本机制来提高数据的容错性和可靠性。每个数据块默认会存储三个副本，分别存储在不同的DataNode上，以防止单个节点故障导致数据丢失。同时，在读取数据时，客户端会选择离其最近的DataNode进行读取，以减少网络带宽的使用和数据传输的延迟。
```



### HDFS 在读取文件的时候，如果其中一个块突然损坏了怎么办

```
在HDFS中，当某个数据块损坏或不可用时，HDFS会自动从副本中选择一个可用的副本作为替代。如果发生数据块损坏，HDFS会检测块校验和并与NameNode联系，以确定哪个副本应该用于替换损坏的副本。然后，HDFS会将新的副本数据传输到该节点，以保证文件完整性和可用性。
 
如果所有副本都损坏或不可用，则该文件将无法完全恢复，此时应考虑使用备份系统或其他故障恢复措施。为了避免出现这种情况，HDFS默认在不同的节点上保存多个副本，并使用周期性检查和自我修复来检测和修复数据损坏或丢失的情况。
```

 HDFS 在上传文件的时候，如果其中一个 DataNode 突然挂掉了怎么办

```
客户端上传文件时与 DataNode 建立 pipeline 管道，管道的正方向是客户端向 DataNode 发送的数据包，管道反向是 DataNode 向客户端发送 ack 确认，也就是正确接收到数据包之后发送一个已确认接收到的应答。



当 DataNode 突然挂掉了，客户端接收不到这个 DataNode 发送的 ack 确认，客户端会通知 NameNode，NameNode 检查该块的副本与规定的不符，NameNode 会通知 DataNode 去复制副本，并将挂掉的 DataNode 作下线处理，不再让它参与文件上传与下载。
```

当NameNode启动时，它会执行以下操作：

```
当NameNode启动时，它会执行以下操作：

读取存储在本地磁盘上的文件系统元数据信息。元数据包括文件系统命名空间中所有目录和文件的名称、权限、时间戳、数据块分配和数据块副本的位置信息等。

将元数据信息加载到内存中，形成内存元数据镜像。这个过程称为“fsimage加载”，在加载完成后，NameNode将拥有完整的内存元数据镜像，可以对文件系统进行操作。

读取最近一次的编辑日志信息，该日志文件包含自上一次checkpoint之后所有文件系统的修改操作。将编辑日志信息与内存元数据镜像进行合并，构建出最新的文件系统状态。这个过程称为“编辑日志应用”。

启动各种后台服务和线程，如数据块报告接收、心跳检测、块扫描、安全性检查等。这些服务和线程负责管理和监视数据节点，并保证文件系统的正常运行。

响应客户端请求，提供文件系统访问服务。

定期执行checkpoint操作，将当前的内存元数据镜像和编辑日志信息合并成一个新的fsimage文件，以及清理过时的编辑日志文件。这个过程会把元数据写回到磁盘上，保证文件系统的一致性和可靠性。

总之，NameNode在启动时主要负责加载元数据信息，构建出文件系统的内存镜像，启动后台服务和线程，以及响应客户端请求。这些操作都是为了保证文件系统的可靠性、高可用性和高性能。
```

### Secondary NameNode 了解吗，它的工作机制是怎样的

```
Secondary NameNode 是 Hadoop 的一个组件，它的作用是帮助 NameNode 处理一些工作，包括日志合并、编辑日志压缩、内存中镜像的持久化等。在 Hadoop 2.0 之后，Secondary NameNode 已经被官方废弃，而是用 Checkpoint Node 来替代。

Secondary NameNode 的工作机制主要分为两个步骤：

定期从 NameNode 获取文件系统的镜像和编辑日志，通常是每小时或每天执行一次。
将镜像和编辑日志合并，并将它们写入一个新的镜像文件中，这个镜像文件包含了最新的文件系统状态。然后，Secondary NameNode 将这个新的镜像文件发送回 NameNode，供其使用。
这样，NameNode 就可以通过这个新的镜像文件来恢复文件系统的状态，而不是从一堆编辑日志中一步一步地恢复。这样的好处是可以加快恢复速度，并且减少编辑日志的数量，从而降低 NameNode 的负担。
```

### Secondary NameNode 不能恢复 NameNode 的全部数据，那如何保证 NameNode 数据存储安全

```
这个问题就要说 NameNode 的高可用了，即 NameNode HA。



一个 NameNode 有单点故障的问题，那就配置双 NameNode，配置有两个关键点，一是必须要保证这两个 NameNode 的元数据信息必须要同步的，二是一个 NameNode 挂掉之后另一个要立马补上。



元数据信息同步在 HA 方案中采用的是“共享存储”。每次写文件时，需要将日志同步写入共享存储，这个步骤成功才能认定写文件成功。然后备份节点定期从共享存储同步日志，以便进行主备切换。

监控 NameNode 状态采用 zookeeper，两个 NameNode 节点的状态存放在 zookeeper 中，另外两个 NameNode 节点分别有一个进程监控程序，实施读取 zookeeper 中有 NameNode 的状态，来判断当前的 NameNode 是不是已经 down 机。如果 Standby 的 NameNode 节点的 ZKFC 发现主节点已经挂掉，那么就会强制给原本的 Active NameNode 节点发送强制关闭请求，之后将备用的 NameNode 设置为 Active。



如果面试官再问 HA 中的 共享存储 是怎么实现的知道吗？

可以进行解释下：NameNode 共享存储方案有很多，比如 Linux HA, VMware FT, QJM 等，目前社区已经把由 Clouderea 公司实现的基于 QJM（Quorum Journal Manager）的方案合并到 HDFS 的 trunk 之中并且作为默认的共享存储实现。

基于 QJM 的共享存储系统主要用于保存 EditLog，并不保存 FSImage 文件。FSImage 文件还是在 NameNode 的本地磁盘上。

QJM 共享存储的基本思想来自于 Paxos 算法，采用多个称为 JournalNode 的节点组成的 JournalNode 集群来存储 EditLog。每个 JournalNode 保存同样的 EditLog 副本。每次 NameNode 写 EditLog 的时候，除了向本地磁盘写入 EditLog 之外，也会并行地向 JournalNode 集群之中的每一个 JournalNode 发送写请求，只要大多数的 JournalNode 节点返回成功就认为向 JournalNode 集群写入 EditLog 成功。如果有 2N+1 台 JournalNode，那么根据大多数的原则，最多可以容忍有 N 台 JournalNode 节点挂掉。
```



###  在 NameNode HA 中，会出现脑裂问题吗？怎么解决脑裂

> 在 NameNode HA 中，的确可能会出现脑裂问题。脑裂是指由于网络分区或其他原因导致 NameNode 集群内的节点无法通信，从而导致集群分裂成两个或多个部分，每个部分都有自己的活动 NameNode。这会导致数据一致性问题，甚至可能导致数据的损失。
>
> 为了解决脑裂问题，Hadoop 提供了一些机制：
>
> 使用 ZooKeeper 作为 NameNode HA 的协调器，通过选举机制选择一个活动的 NameNode，其他的 NameNode 都作为备用节点。ZooKeeper 通过心跳机制来监控活动 NameNode 的状态，如果活动 NameNode 失去响应，ZooKeeper 会将备用节点中的一个晋升为新的活动 NameNode，从而避免脑裂的发生。
>
> 在使用 ZooKeeper 时，可以采用“集中式锁”或“分布式锁”的方式来避免脑裂。即通过对某些资源进行锁定，只有获得锁的节点才能访问资源，从而避免数据的冲突。
>
> 在 NameNode HA 中还可以使用 fencing 机制，即在选举新的活动 NameNode 之前，需要使用某种方式（例如通过 IPMI、WoL 等）将之前的 NameNode “断电”，以确保其不会再次加入集群。
>
> ​	- 在进行 fencing 的时候，会执行以下的操作：
>
> 
>
> 1. 首先尝试调用这个旧 Active NameNode 的 HAServiceProtocol RPC 接口的 transitionToStandby 方法，看能不能把它转换为 Standby 状态。
> 2. 如果 transitionToStandby 方法调用失败，那么就执行 Hadoop 配置文件之中预定义的隔离措施，Hadoop 目前主要提供两种隔离措施，通常会选择 sshfence：
> 3. sshfence：通过 SSH 登录到目标机器上，执行命令 fuser 将对应的进程杀死；
> 4. shellfence：执行一个用户自定义的 shell 脚本来将对应的进程隔离。
>
> 通过上述机制，可以有效地避免 NameNode HA 集群中的脑裂问题。

### 小文件过多会有什么危害，如何避免

1. ```
   小文件过多会导致以下危害：
   
   1. 占用 NameNode 的内存：每个文件和目录都要占用一定的内存空间，小文件过多会导致 NameNode 的内存被耗尽，影响整个集群的性能。
   2. 影响文件系统的性能：小文件的数量较多，会导致文件系统中存在大量的元数据，导致访问文件系统时需要大量的时间来处理这些元数据，从而影响文件系统的性能。
   3. 影响数据块的利用率：HDFS的默认块大小为128MB，小文件会浪费数据块的存储空间，影响数据块的利用率。
   
   为了避免小文件过多的问题，可以采取以下措施：
   
   1. 合并小文件：通过将多个小文件合并成一个大文件，可以减少元数据的数量，节省存储空间，提高系统性能。
   2. 使用 SequenceFile：SequenceFile是一种二进制文件格式，可以将多个小文件合并成一个SequenceFile文件，提高存储和读取的效率。
   3. 使用分区：将数据按照一定的规则进行分区，可以将小文件归类到同一个分区中，减少元数据的数量，提高系统性能。
   4. 采用压缩技术：可以使用压缩技术将小文件压缩成一个文件，从而减少存储空间的占用，提高系统性能。
   ```

### 请说下 HDFS 的组织架构

```
HDFS（Hadoop Distributed File System）是一个分布式文件系统，由以下几个组件构成：

NameNode：维护文件系统的命名空间，存储文件元数据信息（文件名、目录结构、权限等），记录文件块的存储位置以及数据块的校验和信息等。它是HDFS的主节点，每个HDFS集群只有一个NameNode。

DataNode：负责实际存储数据块，向客户端提供数据块的读写服务。DataNode以本地文件系统的形式存储数据块，并定期向NameNode报告本地数据块的信息。

客户端：向NameNode请求读写文件，与DataNode交换数据。客户端可以是Hadoop MapReduce的任务，也可以是用户使用Hadoop提供的命令行工具，如hdfs dfs、hdfs fsck等。

Secondary NameNode：辅助NameNode进行元数据的备份和检查点的创建，减轻NameNode的压力。

这些组件一起协作，形成了HDFS的分布式存储和处理能力。
```

### 请说下 MR 中 Map Task 的工作机制

```
在Hadoop MapReduce（MR）框架中，Map任务的工作机制如下：

读取输入数据：Map任务从HDFS中读取指定的输入文件，根据输入文件的格式将其解析为<key,value>键值对。

处理输入数据：对于每个<key,value>键值对，Map任务会执行Map函数对其进行处理。Map函数是开发人员自己编写的，其主要任务是将输入数据转换成中间键值对。

中间键值对输出：Map函数将处理后的中间结果输出为<key,value>键值对形式，这些中间结果会被缓存到内存中。

分区操作：中间结果会根据它们的键进行分区操作，每个分区内的键值对会被写入到一个独立的输出文件中。

排序操作：每个分区内的键值对会根据键进行排序操作，排序后的结果会被写入到独立的输出文件中。

输出结果：Map任务的输出结果会被写入到HDFS中指定的输出目录中。

需要注意的是，Map任务的输出结果会被传递给Reduce任务进行最终的处理。同时，Map任务的数量会根据输入数据的大小和集群的资源进行自动调整，以实现更好的性能和吞吐量。
```

### 请说下 MR 中 Reduce Task 的工作机制

```
拷贝（Copy）：Reduce Task需要从多个Map Task所在的节点上获取它们的输出结果。通过MapReduce框架提供的机制，Reduce Task会向每个Map Task所在节点发送请求，要求拷贝该节点上的分组文件。

排序（Sort）：Reduce Task接收到来自各个Map Task的分组文件后，需要将它们合并并按照键进行排序。这个排序过程在Map Task中已经完成，因此Reduce Task只需要进行归并操作即可。

聚合（Reduce）：按照键进行分组、排序、归并后，Reduce Task会将分组结果传递给reduce函数进行聚合操作，最终输出每个键对应的结果。

在Reduce Task的工作过程中，主要涉及到数据的传输、排序和聚合等操作。因此，MapReduce框架需要通过网络传输大量数据，并对数据进行排序和归并等操作。这些操作的效率直接影响到整个MapReduce作业的性能。为了提高Reduce Task的执行效率，可以采用一些优化策略，例如调整Map Task的并行度、优化数据压缩算法、使用本地磁盘缓存等。
```

### 请说下 MR 中 Shuffle 阶段

> 在MapReduce中，Shuffle是指将Map输出的数据按照key进行排序和分区，以便于Reduce节点接收到的数据能够进行合并和处理。
>
> Shuffle阶段分为三个步骤：
>
> 1. Partitioning（分区）：Map Task根据key值和分区函数，将数据分到不同的分区中，确保相同的key值会被分到同一个分区中。默认情况下，MapReduce使用HashPartitioner进行分区。
> 2. Sorting（排序）：Reduce Task需要对分区中的数据进行排序，以便于相同key的值被合并到一起。MapReduce使用ExternalSort进行排序。
> 3. Grouping（分组）：将分区中相同key的数据合并在一起，组成一个<key, values>对，其中key为相同的key值，values为该key对应的value的集合。
>
> Shuffle是MapReduce中的一个非常重要的阶段，也是整个MapReduce过程中最耗费网络带宽和IO资源的阶段，因此，Shuffle的优化对于MapReduce的性能和吞吐量非常重要。常见的Shuffle优化手段包括使用本地化优化、压缩、合并等。

### 在写 MR 时，什么情况下可以使用规约

> 在写 MR 时，可以在Map和Reduce阶段之间添加一个Combiner（规约器）来进行规约，以减少数据在网络中传输的量。Combiner是Reduce的本地化版本，在Map任务结束时执行，将Map输出的结果进行合并，以减少Map产生的数据量，从而减轻网络传输负载，提高作业执行效率。Combiner只适用于可以满足交换律和结合律的函数，如求和、求最大/最小值等。



```java
public class WordCount {
  public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    
    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }
  
  public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    private IntWritable result = new IntWritable();
    
    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
  
  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class); // 使用Combiner
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}

```





## Yarn

### YARN 集群的架构和工作原理知道多少

```
YARN 的基本设计思想是将 MapReduce V1 中的 JobTracker 拆分为两个独立的服务：ResourceManager 和 ApplicationMaster。



ResourceManager 负责整个系统的资源管理和分配，ApplicationMaster 负责单个应用程序的的管理。



ResourceManager：RM 是一个全局的资源管理器，负责整个系统的资源管理和分配，它主要由两个部分组成：调度器（Scheduler）和应用程序管理器（Application Manager）。



调度器根据容量、队列等限制条件，将系统中的资源分配给正在运行的应用程序，在保证容量、公平性和服务等级的前提下，优化集群资源利用率，让所有的资源都被充分利用应用程序管理器负责管理整个系统中的所有的应用程序，包括应用程序的提交、与调度器协商资源以启动 ApplicationMaster、监控 ApplicationMaster 运行状态并在失败时重启它。



ApplicationMaster：用户提交的一个应用程序会对应于一个 ApplicationMaster，它的主要功能有：

与 RM 调度器协商以获得资源，资源以 Container 表示。

将得到的任务进一步分配给内部的任务。

与 NM 通信以启动/停止任务。

监控所有的内部任务状态，并在任务运行失败的时候重新为任务申请资源以重启任务。

NodeManager：NodeManager 是每个节点上的资源和任务管理器，一方面，它会定期地向 RM 汇报本节点上的资源使用情况和各个 Container 的运行状态；另一方面，他接收并处理来自 AM 的 Container 启动和停止请求。

Container：Container 是 YARN 中的资源抽象，封装了各种资源。一个应用程序会分配一个 Container，这个应用程序只能使用这个 Container 中描述的资源。不同于 MapReduceV1 中槽位 slot 的资源封装，Container 是一个动态资源的划分单位，更能充分利用资源。
```

### YARN 的任务提交流程是怎样的

 jobclient 向 YARN 提交一个应用程序后，YARN 将分两个阶段运行这个应用程序：一是启动 ApplicationMaster;第二个阶段是由 ApplicationMaster 创建应用程序，为它申请资源，监控运行直到结束。具体步骤如下:



1. 用户向 YARN 提交一个应用程序，并指定 ApplicationMaster 程序、启动 ApplicationMaster 的命令、用户程序。
2. RM 为这个应用程序分配第一个 Container，并与之对应的 NM 通讯，要求它在这个 Container 中启动应用程序 ApplicationMaster。
3. ApplicationMaster 向 RM 注册，然后拆分为内部各个子任务，为各个内部任务申请资源，并监控这些任务的运行，直到结束。
4. AM 采用轮询的方式向 RM 申请和领取资源。
5. RM 为 AM 分配资源，以 Container 形式返回。
6. AM 申请到资源后，便与之对应的 NM 通讯，要求 NM 启动任务。
7. NodeManager 为任务设置好运行环境，将任务启动命令写到一个脚本中，并通过运行这个脚本启动任务。
8. 各个任务向 AM 汇报自己的状态和进度，以便当任务失败时可以重启任务。
9. 应用程序完成后，ApplicationMaster 向 ResourceManager 注销并关闭自己。



###  YARN 的资源调度三种模型了解吗

```
在 Yarn 中有三种调度器可以选择：FIFO Scheduler ，Capacity Scheduler，Fair Scheduler。

Apache 版本的 hadoop 默认使用的是 Capacity Scheduler 调度方式。CDH 版本的默认使用的是 Fair Scheduler 调度方式

FIFO Scheduler（先来先服务）：

FIFO Scheduler 把应用按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。

FIFO Scheduler 是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。大的应用可能会占用所有集群资源，这就导致其它应用被阻塞，比如有个大任务在执行，占用了全部的资源，再提交一个小任务，则此小任务会一直被阻塞。

Capacity Scheduler（能力调度器）：

对于 Capacity 调度器，有一个专门的队列用来运行小任务，但是为小任务专门设置一个队列会预先占用一定的集群资源，这就导致大任务的执行时间会落后于使用 FIFO 调度器时的时间。

Fair Scheduler（公平调度器）：

在 Fair 调度器中，我们不需要预先占用一定的系统资源，Fair 调度器会为所有运行的 job 动态的调整系统资源。

比如：当第一个大 job 提交时，只有这一个 job 在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair 调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。

需要注意的是，在 Fair 调度器中，从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的 Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终的效果就是 Fair 调度器即得到了高的资源利用率又能保证小任务及时完成。
```





## HIVE

### Hive 内部表和外部表的区别

未被 external 修饰的是内部表，被 external 修饰的为外部表。



**区别**：



1. 内部表数据由 Hive 自身管理，外部表数据由 HDFS 管理；
2. 内部表数据存储的位置是`hive.metastore.warehouse.dir`（默认：`/user/hive/warehouse`），外部表数据的存储位置由自己制定（如果没有 LOCATION，Hive 将在 HDFS 上的`/user/hive/warehouse`文件夹下以外部表的表名创建一个文件夹，并将属于这个表的数据存放在这里）；
3. **删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS 上的文件并不会被删除**。

### Hive 索引

Hive 支持索引（3.0 版本之前），但是 Hive 的索引与关系型数据库中的索引并不相同，比如，Hive 不支持主键或者外键。并且 Hive 索引提供的功能很有限，效率也并不高，因此 Hive 索引很少使用。



- 索引适用的场景：



适用于不更新的静态字段。以免总是重建索引数据。每次建立、更新数据后，都要重建索引以构建索引表。



- Hive 索引的机制如下：



hive 在指定列上建立索引，会产生一张索引表（Hive 的一张物理表），里面的字段包括：索引列的值、该值对应的 HDFS 文件路径、该值在文件中的偏移量。



Hive 0.8 版本后引入 bitmap 索引处理器，这个处理器适用于去重后，值较少的列（例如，某字段的取值只可能是几个枚举值）因为索引是用空间换时间，索引列的取值过多会导致建立 bitmap 索引表过大。



**注意**：Hive 中每次有数据时需要及时更新索引，相当于重建一个新表，否则会影响数据查询的效率和准确性，**Hive 官方文档已经明确表示 Hive 的索引不推荐被使用，在新版本的 Hive 中已经被废弃了**。



**扩展**：Hive 是在 0.7 版本之后支持索引的，在 0.8 版本后引入 bitmap 索引处理器，在 3.0 版本开始移除索引的功能，取而代之的是 2.3 版本开始的物化视图，自动重写的物化视图替代了索引的功能



### ORC、Parquet 等列式存储的优点

```

列式存储（如ORC和Parquet）相比于行式存储（如CSV和JSON）有以下几个优点：

更高的压缩率：列式存储更容易压缩相似的数据，因为相似数据在同一列中并且通常是连续的，这有助于减少存储成本和 I/O 开销。

更快的查询速度：由于列式存储可以仅读取与查询相关的列，因此可以减少 I/O 开销，从而提高查询速度。

更快的数据分析：列式存储支持更高级的分析查询，如聚合操作和窗口函数，因为这些查询通常只涉及一小部分列。

更好的数据压缩率：列式存储压缩率高，可以减少存储空间和 I/O 开销，而且在进行压缩时可以利用列中数据的统计信息。

更快的数据加载速度：列式存储可以提高数据加载速度，因为只需要读取要加载的列，而不是整行数据。

总的来说，列式存储更适合用于分析型应用，因为这些应用通常需要查询大量的数据，而且对数据加载和查询速度的要求比较高。而行式存储更适合用于事务型应用，因为这些应用通常需要对一些列进行更新，而且对数据的随机访问的要求比较高。
```



## 数仓

### 数据建模模型

星座模式是星型模式延伸而来，星型模式是基于一张事实表的，而**星座模式是基于多张事实表的，而且共享维度信息**。前面介绍的两种维度建模方法都是多维表对应单事实表，但在很多时候维度空间内的事实表不止一个，而一个维表也可能被多个事实表用到。在业务发展后期，绝大部分维度建模都采用的是星座模式。

### 为什么要对数据仓库分层

对数据仓库进行分层有以下几个主要原因：

1. 提高可维护性和可扩展性：数据仓库通常包含大量的数据，数据量不断增长。如果不进行分层，数据仓库的管理和维护将变得异常复杂，甚至无法维护。通过分层，可以将数据仓库划分为不同的层级，每个层级都有特定的职责和功能，方便管理和维护。
2. 提高查询效率：在数据仓库中，查询是最常见的操作。而数据量大的情况下，查询速度可能会受到很大的影响。通过将数据仓库分层，可以将热点数据和频繁查询的数据放在更容易访问和查询的层级，从而提高查询效率。
3. 支持数据共享：企业内部通常有多个部门或团队需要访问同一份数据。如果没有对数据仓库进行分层，可能会导致访问冲突和数据不一致的问题。通过分层，可以将数据仓库划分为不同的层级，并对每个层级进行访问控制，从而确保数据共享的安全性和一致性。
4. 提供更好的数据质量：在数据仓库中，数据的质量是至关重要的。通过分层，可以将数据仓库划分为不同的层级，并在每个层级中实施不同的数据质量措施，从而提高数据的准确性和一致性。

### 数据仓库是什么

### sort by 和 order by 的区别

在Hive中，SORT BY和ORDER BY都是对查询结果进行排序的关键字。它们的区别在于：

- ORDER BY：对查询结果进行全局排序，即将所有数据都拉取到一个节点上进行排序。在数据量较大的情况下，可能会导致单节点内存不足或者任务失败。
- SORT BY：对查询结果进行局部排序，即在每个Reducer节点上进行排序。这种方式在数据量较大的情况下可以有效地避免单节点内存不足的问题，但是可能会影响排序的精度。

因此，如果需要对查询结果进行全局排序，应该使用ORDER BY。如果需要在数据量较大的情况下进行排序，可以考虑使用SORT BY，并将mapreduce.task.io.sort.mb参数调大以提高排序的精度。

###  Hive 小文件过多怎么解决

##### 1. 使用 hive 自带的 concatenate 命令，自动合并小文件

使用方法：



```
#对于非分区表alter table A concatenate;#对于分区表alter table B partition(day=20201224) concatenate;
```

复制代码

复制代码



> 注意：
>
> 1、concatenate 命令只支持 RCFILE 和 ORC 文件类型。
>
> 2、使用 concatenate 命令合并小文件时不能指定合并后的文件数量，但可以多次执行该命令。
>
> 3、当多次使用 concatenate 后文件数量不在变化，这个跟参数 mapreduce.input.fileinputformat.split.minsize=256mb 的设置有关，可设定每个文件的最小 size。

##### 2. 调整参数减少 Map 数量

设置 map 输入合并小文件的相关参数（执行 Map 前进行小文件合并）：



在 mapper 中将多个文件合成一个 split 作为输入（`CombineHiveInputFormat`底层是 Hadoop 的`CombineFileInputFormat`方法）：



```
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; -- 默认
```

复制代码

复制代码



每个 Map 最大输入大小（这个值决定了合并后文件的数量）：



```
set mapred.max.split.size=256000000;   -- 256M
```

复制代码

复制代码



一个节点上 split 的至少大小（这个值决定了多个 DataNode 上的文件是否需要合并）：



```
set mapred.min.split.size.per.node=100000000;  -- 100M
```

复制代码

复制代码



一个交换机下 split 的至少大小(这个值决定了多个交换机上的文件是否需要合并)：



```
set mapred.min.split.size.per.rack=100000000;  -- 100M
```

复制代码

复制代码

##### 3. 减少 Reduce 的数量

reduce 的个数决定了输出的文件的个数，所以可以调整 reduce 的个数控制 hive 表的文件数量。



hive 中的分区函数 distribute by 正好是控制 MR 中 partition 分区的，可以通过设置 reduce 的数量，结合分区函数让数据均衡的进入每个 reduce 即可：



```
#设置reduce的数量有两种方式，第一种是直接设置reduce个数set mapreduce.job.reduces=10;#第二种是设置每个reduce的大小，Hive会根据数据总大小猜测确定一个reduce个数set hive.exec.reducers.bytes.per.reducer=5120000000; -- 默认是1G，设置为5G#执行以下语句，将数据均衡的分配到reduce中set mapreduce.job.reduces=10;insert overwrite table A partition(dt)select * from Bdistribute by rand();
```

复制代码

复制代码



对于上述语句解释：如设置 reduce 数量为 10，使用 rand()， 随机生成一个数 `x % 10` ，这样数据就会随机进入 reduce 中，防止出现有的文件过大或过小。

##### 4. 使用 hadoop 的 archive 将小文件归档

Hadoop Archive 简称 HAR，是一个高效地将小文件放入 HDFS 块中的文件存档工具，它能够将多个小文件打包成一个 HAR 文件，这样在减少 namenode 内存使用的同时，仍然允许对文件进行透明的访问。



```
#用来控制归档是否可用set hive.archive.enabled=true;#通知Hive在创建归档时是否可以设置父目录set hive.archive.har.parentdir.settable=true;#控制需要归档文件的大小set har.partfile.size=1099511627776;使用以下命令进行归档：ALTER TABLE A ARCHIVE PARTITION(dt='2021-05-07', hr='12');对已归档的分区恢复为原文件：ALTER TABLE A UNARCHIVE PARTITION(dt='2021-05-07', hr='12');
```

复制代码

复制代码



> 注意:
>
> **归档的分区可以查看不能 insert overwrite，必须先 unarchive**



Hive 小文件问题具体可查看：[解决hive小文件过多问题](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg2MzU2MDYzOA%3D%3D%26mid%3D2247483683%26idx%3D1%26sn%3D14b25010032bdf0d375080e48de36d7f%26chksm%3Dce77f7f2f9007ee49d773367f2d7abbf708a2e0e18a55794fcd797944ac73ccc88a41d253d6b%26token%3D1679639512%26lang%3Dzh_CN%23rd) 

### Hive 优化有哪些🐳

##### 1. 数据存储及压缩：

针对 hive 中表的存储格式通常有 orc 和 parquet，压缩格式一般使用 snappy。相比与 textfile 格式表，orc 占有更少的存储。因为 hive 底层使用 MR 计算架构，数据流是 hdfs 到磁盘再到 hdfs，而且会有很多次，所以使用 orc 数据格式和 snappy 压缩策略可以降低 IO 读写，还能降低网络传输量，这样在一定程度上可以节省存储，还能提升 hql 任务执行效率；

##### 2. 通过调参优化：

并行执行，调节 parallel 参数；



调节 jvm 参数，重用 jvm；



设置 map、reduce 的参数；开启 strict mode 模式；



关闭推测执行设置。

##### 3. 有效地减小数据集将大表拆分成子表；结合使用外部表和分区表。

##### 4. SQL 优化

- 大表对大表：尽量减少数据集，可以通过分区表，避免扫描全表或者全字段；
- 大表对小表：设置自动识别小表，将小表放入内存中去执行。

### 数据倾斜怎么解决

数据倾斜问题主要有以下几种：



1. 空值引发的数据倾斜
2. 不同数据类型引发的数据倾斜
3. 不可拆分大文件引发的数据倾斜
4. 数据膨胀引发的数据倾斜
5. 表连接时引发的数据倾斜
6. 确实无法减少数据量引发的数据倾斜

### Tez

```
Tez 可以将多个有依赖的作业转换为一个作业，这样只需写一次 HDFS，且中间节点较少，从而大大提升作业的计算性能。



Mr/tez/spark 区别：



Mr 引擎：多 job 串联，基于磁盘，落盘的地方比较多。虽然慢，但一定能跑出结果。一般处理，周、月、年指标。



Spark 引擎：虽然在 Shuffle 过程中也落盘，但是并不是所有算子都需要 Shuffle，尤其是多算子过程，中间过程不落盘 DAG 有向无环图。 兼顾了可靠性和效率。一般处理天指标。



Tez 引擎：完全基于内存。 注意：如果数据量特别大，慎重使用。容易 OOM。一般用于快速出结果，数据量比较小的场景。
```



## Spark

![image-20230531154924006](https://article.biliimg.com/bfs/article/38cd1b5f5cb79d8fd56464aa9baccfcb64e8ac11.png)





![image-20230606212847671](https://article.biliimg.com/bfs/article/bdb953020c407a041315a39449d55ea72c016916.png)
