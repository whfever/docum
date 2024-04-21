# Redis

## **1.Redis是单线程的还是多线程的？**

当我们在学习和使用redis的时候，大家口耳相传的，**说的都是redis是单线程的**。其实这种说法**并不严谨**，Redis的版本有非常多，3.x、4.x、6.x，版本不同架构也是不同的，**不限定版本问是否单线程**不太严谨。
1）版本3.x，最早版本，也就是大家口口相传的redis是**单线程。**
2）版本4.x，严格意义上来说也不是单线程，而是负责客户端处理请求的线程是单线程，但是开始加了点多线程的东西（**异步删除**）
3）最新版本的6.0.x后，告别了刻板印象中的单线程，**而采用了一种全新的多线程来解决问题。**

![image.png](https://segmentfault.com/img/bVcYdBZ)

所以我们可以得出，Redis 6.0版本以后，对于整个redis来说，是多线程的。

**Redis是单线程到多线程的演变：**

在以前，Redis的网络IO和键值对的读写**都是由一个线程来完成的**，Redis的请求时包括获**取（Socket读），解析，执行，内容返回（socket写）等都是由一个顺序串行的主线程处理**，这就是所谓的**单线程**。

![image.png](https://segmentfault.com/img/bVcYdCd)

但此时，Redis的**瓶颈就出现了**：
**I/O的读写本身是阻塞的**，比如当socket中有数据的时候，Redis会通过调用先将数据**从内核态空间拷贝到用户态空间**，再交给Redis调用，**而这个拷贝的过程就是阻塞的**，当**数据量越大时拷贝所需要的时间就越多**，而这些操作都是基于**单线程**完成的。

在Redis6.0中新增加了多线程的功能来**提高I/O读写性能**，他的主要实现思路是将**主线程的IO读写任务拆分给一组独立的线程去执行**，这样就可以使多个socket的读写并行化了，**采用I/O多路复用技术可以让单个线程高效处理多个连接请求**（尽量减少网络IO的时间消耗），将最耗时的socket读取，请求解析，写入单独外包出去，剩下的命令执行任然是由主线程串行执行和内存的数据交互。

![image.png](https://segmentfault.com/img/bVcYdEM)

结合上图可知，**网络IO操作就变成多线程化了**，其他核心部分仍然是线程安全的，是个不错的折中办法。

流程图：

![image.png](https://segmentfault.com/img/bVcYd1c)

流程简述：
1）主线程负责接收建立链接请求，获取socket放入全局等待读取队列
2）主线程处理完读事件后，**将这些连接分配给这些IO线程**
3）**主线程阻塞等待IO线程读取socket完毕**
4）**IO线程组收集各个socket输入进来的命令，收集完毕**
5）主线解除阻塞，程执行命令，**执行完毕后输出结果数据**
6）主线程阻塞等待 **IO 线程将数据回写** socket 完毕
7）解除绑定，清空等待队列

## 2.Redis性能很快的原因

1）**基于内存操作**，Redis的所有数据都存在内存中，因此所有的运算都是内存级别的，所以他的性能比较高。
2）**数据结构简单**：Redis的数据结构都是专门设计的，而这些简单的数据结构的查找和操作的时间大部分复杂度都是O（1），因此性能比较强，可以参考我之前写过的这篇文章。[深入理解redis——redis经典五种数据类型及底层实现](https://segmentfault.com/a/1190000041459224)
3）**多路复用和非阻塞I/O**，Redis使用I/O多路复用来监听多个socket链接客户端，这样就可以使用一个线程链接来处理多个请求，减少线程切换带来的开销，同时也避免了I/O阻塞操作。
4）**主线程为单线程，避免上下文切换**，因为是单线程模型，因此避免了不必要的上下文切换和多线程竞争（比如锁），这就省去了多线程切换带来的时间和性能上的消耗，而且单线程不会导致死锁问题的发生。

## **3.Redis的瓶颈在哪里？**

说了这么多redis的优点，**redis的性能瓶颈到底在哪里**？

**从cpu上看：**
1）redis是基于内存的，因此减少了cpu将数据从磁盘复制到内存的时间
2）redis是单线程的，因此减少了多线程切换和恢复上下文的时间
3）redis是单线程的，因此多核cpu和单核cpu对于redis来说没有太大影响，单个线程的执行使用一个cpu即可

综上所述，**redis并没有太多受到cpu的限制**，所以cpu大概率不会成为redis的瓶颈。

而**内存大小和网络IO才有可能是redis的瓶颈。**

所以Redis**使用了I/O多路复用模型**，来优化redis。

## **4.I/O多路复用模型理论**

在学习IO多路复用之前，我们先明白下面的几个词语的概念：

1）**同步**：调用者要一直等待调用结果的通知后，才能继续执行，类似于串行执行程序，执行完毕后返回。

2）**异步**：指被调用者先返回应答让调用者先回去，然后计算调用结果，计算完成后，再以回调的方式通知给调用方。

**同步，异步的侧重点**，在于**被调用者**，重点在于获得**调用结果的通知方式上。**

3）**阻塞**：调用方一直在等，且其它别的事情都不做，当前线程会被挂起，啥都不干

4）**非阻塞**：调用在发出去之后，调用方先去干别的事情，不会阻塞当前线程，而会立刻返回。

**阻塞，非阻塞的侧重点**，在于**调用者在等待消息的时候的行为**，**调用者能否干其它的事。**

以上的IO可以组合成4种组合方式：**同步阻塞，同步非阻塞，异步阻塞，异步非阻塞**

同时也就衍生出了unix网络编程中的五种模型：

![image.png](https://segmentfault.com/img/bVcYd7l)

接下来我们用代码进行验证

## **5.I/O多路复用模型JAVA验证**

BIO(阻塞IO)：

我们来看这样一段代码：

假设我们现在有一个服务器，监听6379端口，让别人来链接，我们用这样的方式来写代码：

```csharp
public class RedisServer
{
    public static void main(String[] args) throws IOException
    {
        byte[] bytes = new byte[1024];
        //监听6379端口
        ServerSocket serverSocket = new ServerSocket(6379);
        while(true)
        {
            System.out.println("模拟RedisServer启动-----111 等待连接");
            Socket socket = serverSocket.accept();
            System.out.println("-----222 成功连接");
            System.out.println();
        }
    }
}
```

客户端：

```java
public class RedisClient01
{
    public static void main(String[] args) throws IOException
    {
        System.out.println("------RedisClient01 start");
        Socket socket = new Socket("127.0.0.1", 6379);

    }
}
```

上面这个模型存在很大的问题，如果客户端与服务端建立了链接，**如果上面这个链接的客户端迟迟不发数据，线程就会一直阻塞在read()方法上**，这样其它客户端也不能进行链接，**也就是一个服务器只能处理一个客户端，对客户很不友好**，我们需要升级：

利用多线程：
**只要连接了一个socket，操作系统就会分配一个线程来处理**，这样read()方法堵在每个具体线程上而不阻塞主线程。

```reasonml
public class RedisServerBIOMultiThread
{
    public static void main(String[] args) throws IOException
    {
        ServerSocket serverSocket = new ServerSocket(6379);

        while(true)
        {
            Socket socket = serverSocket.accept();//阻塞1 ,等待客户端连接
            //每次都开一个线程
            new Thread(() -> {
                try {
                    InputStream inputStream = socket.getInputStream();
                    int length = -1;
                    byte[] bytes = new byte[1024];
                    System.out.println("-----333 等待读取");
                    while((length = inputStream.read(bytes)) != -1)//阻塞2 ,等待客户端发送数据
                    {
                        System.out.println("-----444 成功读取"+new String(bytes,0,length));
                        System.out.println("====================");
                        System.out.println();
                    }
                    inputStream.close();
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            },Thread.currentThread().getName()).start();

            System.out.println(Thread.currentThread().getName());

        }
    }
}
```

存在的问题：
**每来一个客户端，就要开辟一个线程，在操作系统中用户态不能直接开辟线程，需要调用内核来创建一个线程，其中还涉及到用户态的切换（上下文的切换），十分消耗资源。**

解决办法：
1）**使用线程池**，在链接少的情况下可以使用，但是在用户量大的情况下，你不知道线程池要多大，太大了内存可能不够，也不可行。
2）**NIO方式，这就引出了我们的NIO**

### **NIO(非阻塞IO):**

在NIO模式中，一切都是**非阻塞**的：
accept()方法是非阻塞的，如果没有客户端连接，就返回error
read()方法是非阻塞的，如果read()方法读取不到数据就返回error，如果读取到数据时只阻塞read()方法读数据的时间

在NIO模式中，只有一个进程：
**当一个客户端与服务器进行链接，这个socket就会加入到一个数组中，隔一段时间遍历一次，这样一个线程就能处理多个客户端的链接和数据了。**

```gradle
public class RedisServerNIO
{
    //将所有socket加入到这个数组中
    static ArrayList<SocketChannel> socketList = new ArrayList<>();
    static ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

    public static void main(String[] args) throws IOException
    {
        System.out.println("---------RedisServerNIO 启动等待中......");
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.bind(new InetSocketAddress("127.0.0.1",6379));
        serverSocket.configureBlocking(false);//设置为非阻塞模式

        while (true)
        {
            //遍历所有数组
            for (SocketChannel element : socketList)
            {
                int read = element.read(byteBuffer);
                if(read > 0)
                {
                    System.out.println("-----读取数据: "+read);
                    byteBuffer.flip();
                    byte[] bytes = new byte[read];
                    byteBuffer.get(bytes);
                    System.out.println(new String(bytes));
                    byteBuffer.clear();
                }
            }

            SocketChannel socketChannel = serverSocket.accept();
            if(socketChannel != null)
            {
                System.out.println("-----成功连接: ");
                socketChannel.configureBlocking(false);//设置为非阻塞模式
                socketList.add(socketChannel);
                System.out.println("-----socketList size: "+socketList.size());
            }
        }
    }
}
```

**NIO成功解决了BIO需要开启多线程的问题，NIO中一个线程就能解决多个socket，但还是存在两个问题：**
**问题一：对cpu不友好**
如果客户端非常多的话，每次遍历都要非常多的socket，**很多都是无效遍历**。

**问题二：线程上下文切换**
这个遍历是在用户态进行的，用户态判断socket是否有数据还是调用内核的read()方法实现的，**这就会涉及到用户态和内核态的切换，每遍历一个就要切换一次，开销非常大。**

**优点**：**不会阻塞在内核的等待数据过程**，每次发起的IO请求可以立刻返回，不用阻塞等待，实时性好。

**缺点**：**会不断询问内核，占用大量cpu时间，资源利用率低。**

改进：**让linux内核搞定上述需求**，我们将一批文件描述符通过一次系统调用传给内核，由内核层去遍历，才能真正解决这个问题，**IO多路复用应运而生**，**就是将上述工作直接放进Linux内核，不再两态转换。**

## **6.Redis如何处理并发客户端链接**

我们先来看看**redis是如何处理这么多客户端连接的**：

![企业微信截图_16461881318128.png](https://segmentfault.com/img/bVcYd4e)

Redis利用**linux内核函数**（下文会讲解）来实现IO多路复用，将**连接信息和事件放到队列中，队列再放到事件派发器，事件派发器把事件分给事件处理器。**

因为Redis**主线程是跑在单线程里面的**，所有操作都得按顺序执行，但是由于**读写操作等待用户的数据输出都是阻塞的，IO在一般的情况下不能直接返回**，这回**导致某一文件的IO阻塞导致整个进程无法对其它客户端提供服务**，而**IO多路复用就是解决这个问题的**。

所谓的I/O多路复用，就是通过一种机制，**让一个线程可以监视多个链接描述符**，一旦某个描述符**就绪**（一般都是**读就绪**或者**写就绪**），就能够通知程序进行相应的读写操作。这种机制的使用需要select 、 poll 、 epoll 来配合。**多个连接共用一个阻塞对象**，应用程序只需要在一个阻塞对象上等待，无需阻塞等待所有连接。当某条连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。

它的概念是：**多个Socket链接复用一根网线，这个功能是在内核+驱动层实现的。**

IO多路复用，也叫I/O multiplexing， 这里面的 multiplexing指的其实是在**单个线程通过记录跟踪每一个sock（I/O流）**的状态来**同时管理多个IO流**，目的是尽量多的提高服务器的吞吐能力。

> ## select/poll/epoll之间的区别
>
> |            | select             | poll             | epoll                                             |
> | :--------- | :----------------- | :--------------- | :------------------------------------------------ |
> | 数据结构   | bitmap             | 数组             | 红黑树                                            |
> | 最大连接数 | 1024               | 无上限           | 无上限                                            |
> | fd拷贝     | 每次调用select拷贝 | 每次调用poll拷贝 | fd首次调用epoll_ctl拷贝，每次调用epoll_wait不拷贝 |
> | 工作效率   | 轮询：O(n)         | 轮询：O(n)       | 回调：O(1)                                        |

![image.png](https://segmentfault.com/img/bVcYeW9)

![image.png](https://segmentfault.com/img/bVcYeYs)

## **7.Linux内核函数select, poll, epoll解析**

select, poll, epoll 都是I/O多路复用的具体的实现。

select方法：

Linux官网源码：
![image.png](https://segmentfault.com/img/bVcYeZH)

select函数监视的文件描述符分3类，分别是**readfds、writefds和exceptfds**，将用户传入的数组拷贝到内核空间，调用select函数**后会进行阻塞**，直到有描述符就绪（有数据 可读，可写或者有except）或者超时（timeout指定等待时间，如果立即返回设为null即可），**函数返回**。

当select函数返回后，**可以通过遍历fdset，来找到就绪的描述符。**

![image.png](https://segmentfault.com/img/bVcYe1A)

从代码中可以看出，select系统调用后，返回了一个位置后的&ret，这样用户态只需要很简单地进行二进制比较，就能很快知道哪些socket需要read数据，有效提高了效率。

优点：
select其实就是把NIO中**用户态要遍历的fd数组**（我们的每一个socket链接，安装进ArrayList里面的那个）**拷贝到了内核态**，让内核态来遍历，因为**用户态判断socket是否有数据还是要调用内核态的**，所有拷贝到内核态后，这样遍历判断的时候就**不用一直用用户态和内核态频繁切换了。**

缺点：
1）**bitmap最大1024位**，一个进程最多**只能处理1024个客户端**
2）**&rset不可重用**，每次socket有数据就相应的位**会被置位**
3）**文件描述符数组拷贝到了内核态，任然有开销**。select调用需要传入fd数组，需要拷贝一份到内核，**高并发场景下这样的拷贝消耗资源是惊人的**。（可优化为不复制）
4）**select并没有通知用户态哪一个socket用户有数据，仍需要O（n）的遍历。**select仅仅返回可读文件描述符的个数，具体哪个可读还是要用户自己遍历。（可优化为只返回给用户就绪的文件描述符，无需用户做无效的遍历）

我们自己模拟写的是，RedisServerNIO.java,只不过将它内核化了。

poll方法：

看一下linux内核源码
![image.png](https://segmentfault.com/img/bVcYe2w)

![image.png](https://segmentfault.com/img/bVcYe2P)

优点：

1）**poll使用pollfd数组来代替select中的bitmap，数组中没有1024的限制**，去掉了select只能监听1024个文件描述符的限制。

2）**当pollfds数组中有事件发生，相应revents置置为1，遍历的时候又置位回零，实现了pollfd数组的重用**

缺点：
1、pollfds数组拷贝到了内核态，**仍然有开销**
2、poll并没有通知用户态哪一个socket有数据，**仍然需要O(n)的遍历**

epoll方法：

epoll操作过程需要三个接口：

epoll_create：创建一个 epoll 句柄
![image.png](https://segmentfault.com/img/bVcYe4i)

epoll_ctl：向内核添加、修改或删除要监控的文件描述符
![image.png](https://segmentfault.com/img/bVcYe4j)

epoll_wait：类似发起了select() 调用
![image.png](https://segmentfault.com/img/bVcYe4l)

![image.png](https://segmentfault.com/img/bVcYe4p)

epoll **是只轮询那些真正发出了事件的流**，并且只依次顺序的处理就绪的流，这种做法就**避免了大量的无用操作**。 采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络 IO 的时间消耗），且 Redis 在内存中操作数据的速度非常快，也就是说内存内的操作不会成为影响Redis性能的瓶颈

**为什么redis一定要部署在Linux机器上才能发挥出该性能？**
因为只有linux有epoll函数，其它系统会自动降级成select函数。

![image.png](https://segmentfault.com/img/bVcYffw)

**三个方法对比：**

|                      | select                                       | poll                                         | epoll                                              |
| -------------------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------------- |
| 操作方式             | 遍历                                         | 遍历                                         | 回调                                               |
| 数据结构             | bitmap                                       | 数组                                         | 红黑树                                             |
| 最大连接数           | 1024（x86）或者2048（x64）                   | 无上限                                       | 无上限                                             |
| 最大支持文件描述符数 | 有最大值限制                                 | 65535                                        | 65535                                              |
| fd拷贝               | 每次调用，都需要把fd结合从用户态拷贝到内核态 | 每次调用，都需要把fd结合从用户态拷贝到内核态 | 只有首次调用的时候拷贝                             |
| 工作效率             | 每次都要遍历所有文件描述符，时间复杂度O（n） | 每次都要遍历所有文件描述符，时间复杂度O（n） | 每次只用遍历需要遍历的文件描述符，时间复杂度O（1） |

## **8.总结**

Redis快的一部分原因在于多路复用，多路复用快的原因是系统提供了这样的调用，使得原来的while循环里的多次系统调用变成了，一次系统调用+内核层遍历这些文件描述符。

![image.png](https://segmentfault.com/img/bVcYffv)

[redis](https://segmentfault.com/t/redis)[缓存](https://segmentfault.com/t/缓存)





## O多路复用原理

为什么 Redis 中要使用 I/O 多路复用这种技术呢？因为 Redis 是跑在**「单线程」**中的，所有的操作都是按照顺序线性执行的，但是**「由于读写操作等待用户输入 或 输出都是阻塞的」**，所以 I/O 操作在一般情况下往往不能直接返回，这会导致某一文件的 I/O 阻塞导，致整个进程无法对其它客户提供服务。而 I/O 多路复用就是为了解决这个问题而出现的。**「为了让单线程(进程)的服务端应用同时处理多个客户端的事件，Redis 采用了 IO 多路复用机制。」**

这里**“多路”**「指的是多个网络连接客户端，」**“复用”**指的是复用同一个线程(单进程)。I/O 多路复用其实是使用一个线程来检查多个 Socket 的就绪状态，在单个线程中通过记录跟踪每一个 socket（I/O流）的状态来管理处理多个 I/O 流。如下图是 Redis 的 I/O 多路复用模型：

![img](https://ask.qcloudimg.com/http-save/yehe-4332331/b426855bd6502c00cf3f5461d822edc5.png)

```javascript
#1.文件描述符(file descriptor)：
    Linux 系统中，把一切都看做是文件，当进程打开现有文件或创建新文件时，内核向进程返回一个文件描述符。可以理解文件描述符是一个索引，这样，要操作文件的时候，我们直接找到索引就可以对其进行操作了。我们将这个索引叫做文件描述符（file descriptor），简称fd。
```

复制

如上图对 Redis 的 I/O 多路复用模型进行一下描述说明：

- (1)一个 socket 客户端与服务端连接时，会生成对应一个套接字描述符(套接字描述符是文件描述符的一种)，每一个 socket 网络连接其实都对应一个文件描述符。
- (2)多个客户端与服务端连接时，Redis 使用 **「I/O 多路复用程序」** 将客户端 socket 对应的 FD 注册到监听列表(一个队列)中。当客服端执行 read、write 等操作命令时，I/O 多路复用程序会将命令封装成一个事件，并绑定到对应的 FD 上。
- (3)**「文件事件处理器」**使用 I/O 多路复用模块同时监控多个文件描述符（fd）的读写情况，当 `accept`、`read`、`write` 和 `close` 文件事件产生时，文件事件处理器就会回调 FD 绑定的事件处理器进行处理相关命令操作。

```javascript
#例如：以 Redis 的 I/O 多路复用程序 epoll 函数为例
    多个客户端连接服务端时，Redis 会将客户端 socket 对应的 fd 注册进 epoll，然后 epoll 同时监听多个文件描述符(FD)是否有数据到来，如果有数据来了就通知事件处理器赶紧处理，这样就不会存在服务端一直等待某个客户端给数据的情形。

#（I/O多路复用程序函数有 select、poll、epoll、kqueue）
```

复制

- (5)整个文件事件处理器是在单线程上运行的，但是通过 I/O 多路复用模块的引入，实现了同时对多个 FD 读写的监控，当其中一个 client 端达到写或读的状态，文件事件处理器就马上执行，从而就不会出现 I/O 堵塞的问题，提高了网络通信的性能。
- (6)如上图，Redis 的 I/O 多路复用模式使用的是 **「Reactor 设置模式」**的方式来实现。





# Redis  Interview

## 基础

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_1-说说什么是redis)1.说说什么是Redis?

![Redis图标](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-96e079f9-49a3-4c55-b0a4-47d043732b62.png)Redis图标

Redis是一种基于键值对（key-value）的NoSQL数据库。

比一般键值对数据库强大的地方，Redis中的value支持string（字符串）、hash（哈希）、 list（列表）、set（集合）、zset（有序集合）、Bitmaps（位图）、 HyperLogLog、GEO（地理信息定位）等多种数据结构，因此 Redis可以满足很多的应用场景。

而且因为Redis会将所有数据都存放在内存中，所以它的读写性能非常出色。

不仅如此，Redis还可以将内存的数据利用快照和日志的形式保存到硬盘上，这样在发生类似断电或者机器故障的时候，内存中的数据不会“丢失”。

除了上述功能以外，Redis还提供了键过期、发布订阅、事务、流水线、Lua脚本等附加功能。

总之，Redis是一款强大的性能利器。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_2-redis可以用来干什么)2.Redis可以用来干什么？

![Redis](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-b02e44b3-3299-450f-b767-4a862b5ac8ff.png)Redis

1. 缓存

   这是Redis应用最广泛地方，基本所有的Web应用都会使用Redis作为缓存，来降低数据源压力，提高响应速度。 ![Redis缓存](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-d44c2397-5994-452f-8b7b-eb85d2b87685.png)

2. 计数器 Redis天然支持计数功能，而且计数性能非常好，可以用来记录浏览量、点赞量等等。

3. 排行榜 Redis提供了列表和有序集合数据结构，合理地使用这些数据结构可以很方便地构建各种排行榜系统。

4. 社交网络 赞/踩、粉丝、共同好友/喜好、推送、下拉刷新。

5. 消息队列 Redis提供了发布订阅功能和阻塞队列的功能，可以满足一般消息队列功能。

6. 分布式锁 分布式环境下，利用Redis实现分布式锁，也是Redis常见的应用。

Redis的应用一般会结合项目去问，以一个电商项目的用户服务为例：

- Token存储：用户登录成功之后，使用Redis存储Token
- 登录失败次数计数：使用Redis计数，登录失败超过一定次数，锁定账号
- 地址缓存：对省市区数据的缓存
- 分布式锁：分布式环境下登录、注册等操作加分布式锁
- ……

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_3-redis-有哪些数据结构)3.Redis 有哪些数据结构？

![Redis基本数据结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-10434dc7-c7a3-4c1a-b484-de3fb37669ee.png) Redis有五种基本数据结构。

**string**

字符串最基础的数据结构。字符串类型的值实际可以是字符串（简单的字符串、复杂的字符串（例如JSON、XML））、数字 （整数、浮点数），甚至是二进制（图片、音频、视频），但是值最大不能超过512MB。

字符串主要有以下几个典型使用场景：

- 缓存功能
- 计数
- 共享Session
- 限速

**hash**

哈希类型是指键值本身又是一个键值对结构。

哈希主要有以下典型应用场景：

- 缓存用户信息
- 缓存对象

**list**

列表（list）类型是用来存储多个有序的字符串。列表是一种比较灵活的数据结构，它可以充当栈和队列的角色

列表主要有以下几种使用场景：

- 消息队列
- 文章列表

**set**

集合（set）类型也是用来保存多个的字符串元素，但和列表类型不一 样的是，集合中不允许有重复元素，并且集合中的元素是无序的。

集合主要有如下使用场景：

- 标签（tag）
- 共同关注

**sorted set**

有序集合中的元素可以排序。但是它和列表使用索引下标作为排序依据不同的是，它给每个元素设置一个权重（score）作为排序的依据。

有序集合主要应用场景：

- 用户点赞统计
- 用户排序

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_4-redis为什么快呢)4.Redis为什么快呢？

Redis的速度⾮常的快，单机的Redis就可以⽀撑每秒十几万的并发，相对于MySQL来说，性能是MySQL的⼏⼗倍。速度快的原因主要有⼏点：

1. **完全基于内存操作**
2. 使⽤单线程，避免了线程切换和竞态产生的消耗
3. 基于⾮阻塞的IO多路复⽤机制
4. C语⾔实现，优化过的数据结构，基于⼏种基础的数据结构，redis做了⼤量的优化，性能极⾼ ![Redis使用IO多路复用和自身事件模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e05bca61-4600-495c-b92a-25ac822e034e.png)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_5-能说一下i-o多路复用吗)5.能说一下I/O多路复用吗？

引用知乎上一个高赞的回答来解释什么是I/O多路复用。假设你是一个老师，让30个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：

- 第一种选择：按顺序逐个检查，先检查A，然后是B，之后是C、D。。。这中间如果有一个学生卡住，全班都会被耽误。这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。
- 第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 这种类似于为每一个用户创建一个进程或者- 线程处理连接。
- 第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。

第一种就是阻塞IO模型，第三种就是I/O复用模型。

![多路复用模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-eb541432-d68a-4dd9-b427-96c4dd607d64.png)多路复用模型

Linux系统有三种方式实现IO多路复用：select、poll和epoll。

例如epoll方式是将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。

这样，整个过程只在进行select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_6-redis为什么早期选择单线程)6. Redis为什么早期选择单线程？

官方解释：[https://redis.io/topics/faqopen in new window](https://redis.io/topics/faq)

![官方单线程解释](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-344b8461-98d4-495b-a697-70275b0abad6.png) 官方FAQ表示，因为Redis是基于内存的操作，CPU成为Redis的瓶颈的情况很少见，Redis的瓶颈最有可能是内存的大小或者网络限制。

如果想要最大程度利用CPU，可以在一台机器上启动多个Redis实例。

PS：网上有这样的回答，吐槽官方的解释有些敷衍，其实就是历史原因，开发者嫌多线程麻烦，后来这个CPU的利用问题就被抛给了使用者。

同时FAQ里还提到了， Redis 4.0 之后开始变成多线程，除了主线程外，它也有后台线程在处理一些较为缓慢的操作，例如清理脏数据、无用连接的释放、大 Key 的删除等等。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_7-redis6-0使用多线程是怎么回事)7.Redis6.0使用多线程是怎么回事?

Redis不是说用单线程的吗？怎么6.0成了多线程的？

Redis6.0的多线程是用多线程来处理数据的**读写和协议解析**，但是Redis**执行命令**还是单线程的。

![Redis6.0多线程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-b7b24e25-d2dc-4457-994f-95bdb3674b8e.png) 这样做的⽬的是因为Redis的性能瓶颈在于⽹络IO⽽⾮CPU，使⽤多线程能提升IO读写的效率，从⽽整体提⾼Redis的性能。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#持久化)持久化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_8-redis持久化方式有哪些-有什么区别)8.Redis持久化⽅式有哪些？有什么区别？

Redis持久化⽅案分为RDB和AOF两种。 ![Redis持久化两种方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-3bda4a46-adc3-4f0d-a135-b8ae5d4c0d5d.png)

**RDB**

RDB持久化是把当前进程数据生成**快照**保存到硬盘的过程，触发RDB持久化过程分为手动触发和自动触发。

RDB⽂件是⼀个压缩的⼆进制⽂件，通过它可以还原某个时刻数据库的状态。由于RDB⽂件是保存在硬盘上的，所以即使Redis崩溃或者退出，只要RDB⽂件存在，就可以⽤它来恢复还原数据库的状态。

手动触发分别对应save和bgsave命令: ![save和bgsave](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-ffe56e32-34c5-453d-8859-c2febbe6a038.png)

- save命令：阻塞当前Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用。
- bgsave命令：Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。

以下场景会自动触发RDB持久化：

- 使用save相关配置，如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave。
- 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点
- 执行debug reload命令重新加载Redis时，也会自动触发save操作
- 默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则自动执行bgsave。

**AOF**

AOF（append only file）持久化：以独立日志的方式记录每次写命令， 重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。

AOF的工作流程操作：命令写入 （append）、文件同步（sync）、文件重写（rewrite）、重启加载 （load） ![AOF工作流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-a9fb6202-b1a1-484d-a4fa-fef519090b44.png)流程如下：

1）所有的写入命令会追加到aof_buf（缓冲区）中。

2）AOF缓冲区根据对应的策略向硬盘做同步操作。

3）随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩 的目的。

4）当Redis服务器重启时，可以加载AOF文件进行数据恢复。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_9-rdb-和-aof-各自有什么优缺点)9.RDB 和 AOF 各自有什么优缺点？

**RDB | 优点**

1. 只有一个紧凑的二进制文件 `dump.rdb`，非常适合备份、全量复制的场景。
2. **容灾性好**，可以把RDB文件拷贝道远程机器或者文件系统张，用于容灾恢复。
3. **恢复速度快**，RDB恢复数据的速度远远快于AOF的方式

**RDB | 缺点**

1. **实时性低**，RDB 是间隔一段时间进行持久化，没法做到实时持久化/秒级持久化。如果在这一间隔事件发生故障，数据会丢失。
2. **存在兼容问题**，Redis演进过程存在多个格式的RDB版本，存在老版本Redis无法兼容新版本RDB的问题。

**AOF | 优点**

1. **实时性好**，aof 持久化可以配置 `appendfsync` 属性，有 `always`，每进行一次命令操作就记录到 aof 文件中一次。
2. 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。

**AOF | 缺点**

1. AOF 文件比 RDB **文件大**，且 **恢复速度慢**。
2. **数据集大** 的时候，比 RDB **启动效率低**。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_10-rdb和aof如何选择)10.RDB和AOF如何选择？

- 一般来说， 如果想达到足以媲美数据库的 **数据安全性**，应该 **同时使用两种持久化功能**。在这种情况下，当 Redis 重启的时候会优先载入 AOF 文件来恢复原始的数据，因为在通常情况下 AOF 文件保存的数据集要比 RDB 文件保存的数据集要完整。
- 如果 **可以接受数分钟以内的数据丢失**，那么可以 **只使用 RDB 持久化**。
- 有很多用户都只使用 AOF 持久化，但并不推荐这种方式，因为定时生成 RDB 快照（snapshot）非常便于进行数据备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快，除此之外，使用 RDB 还可以避免 AOF 程序的 bug。
- 如果只需要数据在服务器运行的时候存在，也可以不使用任何持久化方式。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_11-redis的数据恢复)11.Redis的数据恢复？

当Redis发生了故障，可以从RDB或者AOF中恢复数据。

恢复的过程也很简单，把RDB或者AOF文件拷贝到Redis的数据目录下，如果使用AOF恢复，配置文件开启AOF，然后启动redis-server即可。 ![Redis启动加载数据](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-f9aab5e9-a875-4316-9ec9-0c5650afe5c1.png)

**Redis** 启动时加载数据的流程：

1. AOF持久化开启且存在AOF文件时，优先加载AOF文件。
2. AOF关闭或者AOF文件不存在时，加载RDB文件。
3. 加载AOF/RDB文件成功后，Redis启动成功。
4. AOF/RDB文件存在错误时，Redis启动失败并打印错误信息。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_12-redis-4-0-的混合持久化了解吗)12.Redis 4.0 的混合持久化了解吗？

重启 Redis 时，我们很少使用 `RDB` 来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是重放 AOF 日志性能相对 `RDB` 来说要慢很多，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。

**Redis 4.0** 为了解决这个问题，带来了一个新的持久化选项——**混合持久化**。将 `rdb` 文件的内容和增量的 AOF 日志文件存在一起。这里的 AOF 日志不再是全量的日志，而是 **自持久化开始到持久化结束** 的这段时间发生的增量 AOF 日志，通常这部分 AOF 日志很小： ![混合持久化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-19c531e5-da95-495a-a4c4-d63a0b8bba95.png)

于是在 Redis 重启的时候，可以先加载 `rdb` 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，重启效率因此大幅得到提升。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#高可用)高可用

Redis保证高可用主要有三种方式：主从、哨兵、集群。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_13-主从复制了解吗)13.主从复制了解吗？

![Redis主从复制简图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-60497f1e-8afb-44b3-bb7a-d4c29e5ac484.png)Redis主从复制简图

**主从复制**，是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。前者称为 **主节点(master)**，后者称为 **从节点(slave)**。且数据的复制是 **单向** 的，只能由主节点到从节点。Redis 主从复制支持 **主从同步** 和 **从从同步** 两种，后者是 Redis 后续版本新增的功能，以减轻主节点的同步负担。

> 主从复制主要的作用?

- **数据冗余：** 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复：** 当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复 *(实际上是一种服务的冗余)*。
- **负载均衡：** 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 *（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点）*，分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
- **高可用基石：** 除了上述作用以外，主从复制还是哨兵和集群能够实施的 **基础**，因此说主从复制是 Redis 高可用的基础。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_14-redis主从有几种常见的拓扑结构)14.Redis主从有几种常见的拓扑结构？

Redis的复制拓扑结构可以支持单层或多层复制关系，根据拓扑复杂性可以分为以下三种：一主一从、一主多从、树状主从结构。

1.一主一从结构

一主一从结构是最简单的复制拓扑结构，用于主节点出现宕机时从节点提供故障转移支持。 ![一主一从结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-5d91a67c-dbff-4a8d-bf9d-1fe7602d5a27.png) 2.一主多从结构

一主多从结构（又称为星形拓扑结构）使得应用端可以利用多个从节点实现读写分离（见图6-5）。对于读占比较大的场景，可以把读命令发送到从节点来分担主节点压力。 ![一主多从结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-71074254-699a-480b-bbb0-c68f364a380b.png) 3.树状主从结构

树状主从结构（又称为树状拓扑结构）使得从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制。通过引入复制中间层，可以有效降低主节点负载和需要传送给从节点的数据量。 ![树状主从结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-dff14203-5e01-4d1b-a775-10ee444ada54.png)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_15-redis的主从复制原理了解吗)15.Redis的主从复制原理了解吗？

Redis主从复制的工作流程大概可以分为如下几步： ![Redis主从复制工作流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-21123b1e-68b4-436b-ac84-3365a49a81bd.png)

1. 保存主节点（master）信息 这一步只是保存主节点信息，保存主节点的ip和port。
2. 主从建立连接 从节点（slave）发现新的主节点后，会尝试和主节点建立网络连接。
3. 发送ping命令 连接建立成功后从节点发送ping请求进行首次通信，主要是检测主从之间网络套接字是否可用、主节点当前是否可接受处理命令。
4. 权限验证 如果主节点要求密码验证，从节点必须正确的密码才能通过验证。
5. 同步数据集 主从复制连接正常通信后，主节点会把持有的数据全部发送给从节点。
6. 命令持续复制 接下来主节点会持续地把写命令发送给从节点，保证主从数据一致性。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_16-说说主从数据同步的方式)16.说说主从数据同步的方式？

Redis在2.8及以上版本使用psync命令完成主从数据同步，同步过程分为：全量复制和部分复制。

![主从数据同步方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-7518f715-6dee-4e70-b972-8aed9879e451.png)主从数据同步方式

**全量复制** 一般用于初次复制场景，Redis早期支持的复制功能只有全量复制，它会把主节点全部数据一次性发送给从节点，当数据量较大时，会对主从节点和网络造成很大的开销。

全量复制的完整运行流程如下： ![全量复制](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-aa8d2960-b341-49cc-b04c-201241fd15de.png)

1. 发送psync命令进行数据同步，由于是第一次进行复制，从节点没有复制偏移量和主节点的运行ID，所以发送psync-1。
2. 主节点根据psync-1解析出当前为全量复制，回复+FULLRESYNC响应。
3. 从节点接收主节点的响应数据保存运行ID和偏移量offset
4. 主节点执行bgsave保存RDB文件到本地
5. 主节点发送RDB文件给从节点，从节点把接收的RDB文件保存在本地并直接作为从节点的数据文件
6. 对于从节点开始接收RDB快照到接收完成期间，主节点仍然响应读写命令，因此主节点会把这期间写命令数据保存在复制客户端缓冲区内，当从节点加载完RDB文件后，主节点再把缓冲区内的数据发送给从节点，保证主从之间数据一致性。
7. 从节点接收完主节点传送来的全部数据后会清空自身旧数据
8. 从节点清空数据后开始加载RDB文件
9. 从节点成功加载完RDB后，如果当前节点开启了AOF持久化功能， 它会立刻做bgrewriteaof操作，为了保证全量复制后AOF持久化文件立刻可用。

**部分复制** 部分复制主要是Redis针对全量复制的过高开销做出的一种优化措施， 使用psync{runId}{offset}命令实现。当从节点（slave）正在复制主节点 （master）时，如果出现网络闪断或者命令丢失等异常情况时，从节点会向 主节点要求补发丢失的命令数据，如果主节点的复制积压缓冲区内存在这部分数据则直接发送给从节点，这样就可以保持主从节点复制的一致性。 ![部分复制](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-87600c72-cc6a-4656-81b2-e71864c97f23.png)

1. 当主从节点之间网络出现中断时，如果超过repl-timeout时间，主节点会认为从节点故障并中断复制连接
2. 主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过主节点内部存在的复制积压缓冲区，依然可以保存最近一段时间的写命令数据，默认最大缓存1MB。
3. 当主从节点网络恢复后，从节点会再次连上主节点
4. 当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当作psync参数发送给主节点，要求进行部分复制操作。
5. 主节点接到psync命令后首先核对参数runId是否与自身一致，如果一 致，说明之前复制的是当前主节点；之后根据参数offset在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送+CONTINUE响应，表示可以进行部分复制。
6. 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_17-主从复制存在哪些问题呢)17.主从复制存在哪些问题呢？

主从复制虽好，但也存在一些问题：

- 一旦主节点出现故障，需要手动将一个从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点，整个过程都需要人工干预。
- 主节点的写能力受到单机的限制。
- 主节点的存储能力受到单机的限制。

第一个问题是Redis的高可用问题，第二、三个问题属于Redis的分布式问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_18-redis-sentinel-哨兵-了解吗)18.Redis Sentinel（哨兵）了解吗？

主从复制存在一个问题，没法完成自动故障转移。所以我们需要一个方案来完成自动故障转移，它就是Redis Sentinel（哨兵）。

![Redis Sentinel](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-8b1a055c-f077-49ff-9432-c194d4fc3639.png)Redis Sentinel

Redis Sentinel ，它由两部分组成，哨兵节点和数据节点：

- **哨兵节点：** 哨兵系统由一个或多个哨兵节点组成，哨兵节点是特殊的 Redis 节点，不存储数据，对数据节点进行监控。
- **数据节点：** 主节点和从节点都是数据节点；

在复制的基础上，哨兵实现了 **自动化的故障恢复** 功能，下面是官方对于哨兵功能的描述：

- **监控（Monitoring）：** 哨兵会不断地检查主节点和从节点是否运作正常。
- **自动故障转移（Automatic failover）：** 当 **主节点** 不能正常工作时，哨兵会开始 **自动故障转移操作**，它会将失效主节点的其中一个 **从节点升级为新的主节点**，并让其他从节点改为复制新的主节点。
- **配置提供者（Configuration provider）：** 客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
- **通知（Notification）：** 哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移。而配置提供者和通知功能，则需要在与客户端的交互中才能体现。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_19-redis-sentinel-哨兵-实现原理知道吗)19.Redis Sentinel（哨兵）实现原理知道吗？

哨兵模式是通过哨兵节点完成对数据节点的监控、下线、故障转移。 ![Redis Sentinel工作流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-4074d72a-886a-4892-8f55-80112005aad8.png)

- 定时监控

  

  Redis Sentinel通过三个定时监控任务完成对各个节点发现和监控：

  1. 每隔10秒，每个Sentinel节点会向主节点和从节点发送info命令获取最新的拓扑结构
  2. 每隔2秒，每个Sentinel节点会向Redis数据节点的__sentinel__：hello 频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息
  3. 每隔1秒，每个Sentinel节点会向主节点、从节点、其余Sentinel节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达

- **主观下线和客观下线** 主观下线就是哨兵节点认为某个节点有问题，客观下线就是超过一定数量的哨兵节点认为主节点有问题。 ![主观下线和客观下线](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-11839a24-9249-48a5-8c9d-888aa80d91dc.png)

1. 主观下线 每个Sentinel节点会每隔1秒对主节点、从节点、其他Sentinel节点发送ping命令做心跳检测，当这些节点超过 down-after-milliseconds没有进行有效回复，Sentinel节点就会对该节点做失败判定，这个行为叫做主观下线。
2. 客观下线 当Sentinel主观下线的节点是主节点时，该Sentinel节点会通过sentinel is- master-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过 <quorum>个数，Sentinel节点认为主节点确实有问题，这时该Sentinel节点会做出客观下线的决定

- **领导者Sentinel节点选举** Sentinel节点之间会做一个领导者选举的工作，选出一个Sentinel节点作为领导者进行故障转移的工作。Redis使用了Raft算法实现领导者选举。

- **故障转移**

  领导者选举出的Sentinel节点负责故障转移，过程如下： ![故障转移](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-0618a5e2-e94f-40d7-888a-e78019ba8f93.png)

  1. 在从节点列表中选出一个节点作为新的主节点，这一步是相对复杂一些的一步
  2. Sentinel领导者节点会对第一步选出来的从节点执行slaveof no one命令让其成为主节点
  3. Sentinel领导者节点会向剩余的从节点发送命令，让它们成为新主节点的从节点
  4. Sentinel节点集合会将原来的主节点更新为从节点，并保持着对其关注，当其恢复后命令它去复制新的主节点

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_20-领导者sentinel节点选举了解吗)20.领导者Sentinel节点选举了解吗？

Redis使用了Raft算法实 现领导者选举，大致流程如下： ![领导者Sentinel节点选举](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-6586ae3c-6b33-4c9f-8137-f02fc0e6cfae.png)

1. 每个在线的Sentinel节点都有资格成为领导者，当它确认主节点主观 下线时候，会向其他Sentinel节点发送sentinel is-master-down-by-addr命令， 要求将自己设置为领导者。
2. 收到命令的Sentinel节点，如果没有同意过其他Sentinel节点的sentinel is-master-down-by-addr命令，将同意该请求，否则拒绝。
3. 如果该Sentinel节点发现自己的票数已经大于等于max（quorum， num（sentinels）/2+1），那么它将成为领导者。
4. 如果此过程没有选举出领导者，将进入下一次选举。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_21-新的主节点是怎样被挑选出来的)21.新的主节点是怎样被挑选出来的？

选出新的主节点，大概分为这么几步： ![新的主节点](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-03976d35-20b6-4efe-aa9c-7d3759460d34.png)

1. 过滤：“不健康”（主观下线、断线）、5秒内没有回复过Sentinel节 点ping响应、与主节点失联超过down-after-milliseconds*10秒。
2. 选择slave-priority（从节点优先级）最高的从节点列表，如果存在则返回，不存在则继续。
3. 选择复制偏移量最大的从节点（复制的最完整），如果存在则返 回，不存在则继续。
4. 选择runid最小的从节点。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_22-redis-集群了解吗)22.Redis 集群了解吗？

前面说到了主从存在高可用和分布式的问题，哨兵解决了高可用的问题，而集群就是终极方案，一举解决高可用和分布式问题。 ![Redis 集群示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-5cbc6009-251e-4d5b-8f22-8d543938eccb.png)

1. **数据分区：** 数据分区 *(或称数据分片)* 是集群最核心的功能。集群将数据分散到多个节点，一方面 突破了 Redis 单机内存大小的限制，**存储容量大大增加**；**另一方面** 每个主节点都可以对外提供读服务和写服务，**大大提高了集群的响应能力**。
2. **高可用：** 集群支持主从复制和主节点的 **自动故障转移** *（与哨兵类似）*，当任一节点发生故障时，集群仍然可以对外提供服务。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_23-集群中数据如何分区)23.集群中数据如何分区？

分布式的存储中，要把数据集按照分区规则映射到多个节点，常见的数据分区规则三种： ![分布式数据分区](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-ceb49e41-dfd7-4d1e-91f9-c299437227d2.png)

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#方案一-节点取余分区)方案一：节点取余分区

节点取余分区，非常好理解，使用特定的数据，比如Redis的键，或者用户ID之类，对响应的hash值取余：hash（key）%N，来确定数据映射到哪一个节点上。

不过该方案最大的问题是，当节点数量变化时，如扩容或收缩节点，数据节点映射关 系需要重新计算，会导致数据的重新迁移。

![节点取余分区](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-8b1fcaec-37e6-420a-9ca2-03615232af17.png)节点取余分区

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#方案二-一致性哈希分区)方案二：一致性哈希分区

将整个 Hash 值空间组织成一个虚拟的圆环，然后将缓存节点的 IP 地址或者主机名做 Hash 取值后，放置在这个圆环上。当我们需要确定某一个 Key 需 要存取到哪个节点上的时候，先对这个 Key 做同样的 Hash 取值，确定在环上的位置，然后按照顺时针方向在环上“行走”，遇到的第一个缓存节点就是要访问的节点。

比如说下面 这张图里面，Key 1 和 Key 2 会落入到 Node 1 中，Key 3、Key 4 会落入到 Node 2 中，Key 5 落入到 Node 3 中，Key 6 落入到 Node 4 中。 ![一致性哈希分区](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-89bd1c1c-251c-4f53-bba3-fe945b2ae9e2.png)

这种方式相比节点取余最大的好处在于加入和删除节点只影响哈希环中 相邻的节点，对其他节点无影响。

但它还是存在问题：

- 缓存节点在圆环上分布不平均，会造成部分缓存节点的压力较大
- 当某个节点故障时，这个节点所要承担的所有访问都会被顺移到另一个节点上，会对后面这个节点造成力。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#方案三-虚拟槽分区)方案三：虚拟槽分区

这个方案 一致性哈希分区的基础上，引入了 **虚拟节点** 的概念。Redis 集群使用的便是该方案，其中的虚拟节点称为 **槽（slot）**。槽是介于数据和实际节点之间的虚拟概念，每个实际节点包含一定数量的槽，每个槽包含哈希值在一定范围内的数据。 ![虚拟槽分配](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e0ed9d62-3406-40db-8b01-c931f1020612.png)

在使用了槽的一致性哈希分区中，槽是数据管理和迁移的基本单位。槽解耦了数据和实际节点 之间的关系，增加或删除节点对系统的影响很小。仍以上图为例，系统中有 `4` 个实际节点，假设为其分配 `16` 个槽(0-15)；

- 槽 0-3 位于 node1；4-7 位于 node2；以此类推....

如果此时删除 `node2`，只需要将槽 4-7 重新分配即可，例如槽 4-5 分配给 `node1`，槽 6 分配给 `node3`，槽 7 分配给 `node4`，数据在其他节点的分布仍然较为均衡。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_24-能说说redis集群的原理吗)24.能说说Redis集群的原理吗？

Redis集群通过数据分区来实现数据的分布式存储，通过自动故障转移实现高可用。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#集群创建)集群创建

数据分区是在集群创建的时候完成的。 ![集群创建](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-046a512c-baab-4e3a-9409-2af58088cceb.png)

**设置节点** Redis集群一般由多个节点组成，节点数量至少为6个才能保证组成完整高可用的集群。每个节点需要开启配置cluster-enabled yes，让Redis运行在集群模式下。 ![节点和握手](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e6064ba6-fd6f-4270-92f9-68c0bb98fd4b.png)**节点握手** 节点握手是指一批运行在集群模式下的节点通过Gossip协议彼此通信， 达到感知对方的过程。节点握手是集群彼此通信的第一步，由客户端发起命 令：cluster meet{ip}{port}。完成节点握手之后，一个个的Redis节点就组成了一个多节点的集群。

**分配槽（slot）** Redis集群把所有的数据映射到16384个槽中。每个节点对应若干个槽，只有当节点分配了槽，才能响应和这些槽关联的键命令。通过 cluster addslots命令为节点分配槽。

![分配槽](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-15341792-e7a6-428c-a109-22827e02be5f.png)分配槽

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#故障转移)故障转移

Redis集群的故障转移和哨兵的故障转移类似，但是Redis集群中所有的节点都要承担状态维护的任务。

**故障发现** Redis集群内节点通过ping/pong消息实现节点通信，集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong 消息作为响应。如果在cluster-node-timeout时间内通信一直失败，则发送节 点会认为接收节点存在故障，把接收节点标记为主观下线（pfail）状态。 ![主观下线](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-84a2a89e-f9ea-4681-b748-1a4f1dee172b.png) 当某个节点判断另一个节点主观下线后，相应的节点状态会跟随消息在集群内传播。通过Gossip消息传播，集群内节点不断收集到故障节点的下线报告。当 半数以上持有槽的主节点都标记某个节点是主观下线时。触发客观下线流程。 ![主观下线和客观下线](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-b61a6109-7aea-45ab-a53c-267eebb9180a.png)

**故障恢复**

故障节点变为客观下线后，如果下线节点是持有槽的主节点则需要在它 的从节点中选出一个替换它，从而保证集群的高可用。

![故障恢复流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-0e5a49b3-cb5a-4aef-a81f-fce50a012a39.png)故障恢复流程

1. 资格检查 每个从节点都要检查最后与主节点断线时间，判断是否有资格替换故障 的主节点。
2. 准备选举时间 当从节点符合故障转移资格后，更新触发故障选举的时间，只有到达该 时间后才能执行后续流程。
3. 发起选举 当从节点定时任务检测到达故障选举时间（failover_auth_time）到达后，发起选举流程。
4. 选举投票 持有槽的主节点处理故障选举消息。投票过程其实是一个领导者选举的过程，如集群内有N个持有槽的主节 点代表有N张选票。由于在每个配置纪元内持有槽的主节点只能投票给一个 从节点，因此只能有一个从节点获得N/2+1的选票，保证能够找出唯一的从节点。 ![选举投票](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-d0e16ea3-6683-43f4-82a3-80478703ae06.png)
5. 替换主节点 当从节点收集到足够的选票之后，触发替换主节点操作。

> **部署Redis集群至少需要几个物理节点？**

在投票选举的环节，故障主节点也算在投票数内，假设集群内节点规模是3主3从，其中有2 个主节点部署在一台机器上，当这台机器宕机时，由于从节点无法收集到 3/2+1个主节点选票将导致故障转移失败。这个问题也适用于故障发现环节。因此部署集群时所有主节点最少需要部署在3台物理机上才能避免单点问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_25-说说集群的伸缩)25.说说集群的伸缩？

Redis集群提供了灵活的节点扩容和收缩方案，可以在不影响集群对外服务的情况下，为集群添加节点进行扩容也可以下线部分节点进行缩容。 ![集群的伸缩](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-dd3e9494-eddb-4861-85f7-2646018d93f6.png)其实，集群扩容和缩容的关键点，就在于槽和节点的对应关系，扩容和缩容就是将一部分`槽`和`数据`迁移给新节点。

例如下面一个集群，每个节点对应若干个槽，每个槽对应一定的数据，如果希望加入1个节点希望实现集群扩容时，需要通过相关命令把一部分槽和内容迁移给新节点。 ![扩容实例](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-1d24bb63-2b05-4db9-bd6b-983f16a4830e.png)缩容也是类似，先把槽和数据迁移到其它节点，再把对应的节点下线。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#缓存设计)缓存设计

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_26-什么是缓存击穿、缓存穿透、缓存雪崩)26.什么是缓存击穿、缓存穿透、缓存雪崩？

PS:这是多年黄历的老八股了，一定要理解清楚。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#缓存击穿)缓存击穿

一个并发访问量比较大的key在某个时间过期，导致所有的请求直接打在DB上。

![缓存击穿](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-86579ee6-9dae-4274-a5cc-af6812f48da4.png) 解决⽅案：

1. 加锁更新，⽐如请求查询A，发现缓存中没有，对A这个key加锁，同时去数据库查询数据，写⼊缓存，再返回给⽤户，这样后⾯的请求就可以从缓存中拿到数据了。 ![加锁更新](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-cf63911a-8501-493e-a375-8b47a9f33358.png)
2. 将过期时间组合写在value中，通过异步的⽅式不断的刷新过期时间，防⽌此类现象。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#缓存穿透)缓存穿透

缓存穿透指的查询缓存和数据库中都不存在的数据，这样每次请求直接打到数据库，就好像缓存不存在一样。

![缓存穿透](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-029951e6-8b99-4364-a570-010853deb594.png) 缓存穿透将导致不存在的数据每次请求都要到存储层去查询，失去了缓存保护后端存储的意义。

缓存穿透可能会使后端存储负载加大，如果发现大量存储层空命中，可能就是出现了缓存穿透问题。

缓存穿透可能有两种原因：

1. 自身业务代码问题
2. 恶意攻击，爬虫造成空命中

它主要有两种解决办法：

- **缓存空值/默认值**

一种方式是在数据库不命中之后，把一个空对象或者默认值保存到缓存，之后再访问这个数据，就会从缓存中获取，这样就保护了数据库。

![缓存空值/默认值](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-288af5a2-ae5a-427a-95e9-b4a658b01386.png)缓存空值/默认值

缓存空值有两大问题：

1. 空值做了缓存，意味着缓存层中存了更多的键，需要更多的内存空间（如果是攻击，问题更严重），比较有效的方法是针对这类数据设置一个较短的过期时间，让其自动剔除。
2. 缓存层和存储层的数据会有一段时间窗口的不一致，可能会对业务有一定影响。 例如过期时间设置为5分钟，如果此时存储层添加了这个数据，那此段时间就会出现缓存层和存储层数据的不一致。 这时候可以利用消息队列或者其它异步方式清理缓存中的空对象。

- **布隆过滤器** 除了缓存空对象，我们还可以在存储和缓存之前，加一个布隆过滤器，做一层过滤。

布隆过滤器里会保存数据是否存在，如果判断数据不不能再，就不会访问存储。 ![布隆过滤器](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-0e18ea40-a2e5-4fa6-989e-e771f6e4b0fc.png) 两种解决方案的对比： ![缓存空对象核布隆过滤器方案对比](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e8a382c9-4379-44ab-b1dc-fb598a228105.png)

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#缓存雪崩)缓存雪崩

某⼀时刻发⽣⼤规模的缓存失效的情况，例如缓存服务宕机、大量key在同一时间过期，这样的后果就是⼤量的请求进来直接打到DB上，可能导致整个系统的崩溃，称为雪崩。

![缓存雪崩](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-1464fe22-c463-4850-8989-b899510cb10e.png) 缓存雪崩是三大缓存问题里最严重的一种，我们来看看怎么预防和处理。

- **提高缓存可用性**

1. 集群部署：通过集群来提升缓存的可用性，可以利用Redis本身的Redis Cluster或者第三方集群方案如Codis等。
2. 多级缓存：设置多级缓存，第一级缓存失效的基础上，访问二级缓存，每一级缓存的失效时间都不同。

- **过期时间**

1. 均匀过期：为了避免大量的缓存在同一时间过期，可以把不同的 key 过期时间随机生成，避免过期时间太过集中。
2. 热点数据永不过期。

- **熔断降级**

1. 服务熔断：当缓存服务器宕机或超时响应时，为了防止整个系统出现雪崩，暂时停止业务服务访问缓存系统。
2. 服务降级：当出现大量缓存失效，而且处在高并发高负荷的情况下，在业务系统内部暂时舍弃对一些非核心的接口和数据的请求，而直接返回一个提前准备好的 fallback（退路）错误处理信息。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_27-能说说布隆过滤器吗)27.能说说布隆过滤器吗？

布隆过滤器，它是一个连续的数据结构，每个存储位存储都是一个`bit`，即`0`或者`1`, 来标识数据是否存在。

存储数据的时时候，使用K个不同的哈希函数将这个变量映射为bit列表的的K个点，把它们置为1。

![布隆过滤器](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-d0b8d85c-85dc-4843-b4be-d5d48338a44e.png)我们判断缓存key是否存在，同样，K个哈希函数，映射到bit列表上的K个点，判断是不是1：

- 如果全不是1，那么key不存在；
- 如果都是1，也只是表示key可能存在。

布隆过滤器也有一些缺点：

1. 它在判断元素是否在集合中时是有一定错误几率，因为哈希算法有一定的碰撞的概率。
2. 不支持删除元素。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_28-如何保证缓存和数据库数据的一致性)28.如何保证缓存和数据库数据的⼀致性？

根据CAP理论，在保证可用性和分区容错性的前提下，无法保证一致性，所以缓存和数据库的绝对一致是不可能实现的，只能尽可能保存缓存和数据库的最终一致性。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#选择合适的缓存更新策略)选择合适的缓存更新策略

**1. 删除缓存而不是更新缓存**

当一个线程对缓存的key进行写操作的时候，如果其它线程进来读数据库的时候，读到的就是脏数据，产生了数据不一致问题。

相比较而言，删除缓存的速度比更新缓存的速度快很多，所用时间相对也少很多，读脏数据的概率也小很多。 ![删除缓存和更新缓存](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-ebad0a67-3012-4466-a4dc-e834104c48f8.png)

1. **先更数据，后删缓存** 先更数据库还是先删缓存？这是一个问题。

更新数据，耗时可能在删除缓存的百倍以上。在缓存中不存在对应的key，数据库又没有完成更新的时候，如果有线程进来读取数据，并写入到缓存，那么在更新成功之后，这个key就是一个脏数据。

毫无疑问，先删缓存，再更数据库，缓存中key不存在的时间的时间更长，有更大的概率会产生脏数据。

![先更数据库还是先删缓存](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-5c929a9e-a723-43b3-8f3c-f22c83765f9d.png)目前最流行的缓存读写策略cache-aside-pattern就是采用先更数据库，再删缓存的方式。

##### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#缓存不一致处理)缓存不一致处理

如果不是并发特别高，对缓存依赖性很强，其实一定程序的不一致是可以接受的。

但是如果对一致性要求比较高，那就得想办法保证缓存和数据库中数据一致。

缓存和数据库数据不一致常见的两种原因：

- 缓存key删除失败
- 并发导致写入了脏数据

![缓存一致性](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-51b3453e-e1f1-4c65-9879-2aa59df23eca.png)缓存一致性

**消息队列保证key被删除** 可以引入消息队列，把要删除的key或者删除失败的key丢尽消息队列，利用消息队列的重试机制，重试删除对应的key。

![消息队列保证key被删除](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e4a61193-515a-409f-a436-2733abc3a86e.png)这种方案看起来不错，缺点是对业务代码有一定的侵入性。

**数据库订阅+消息队列保证key被删除** 可以用一个服务（比如阿里的 canal）去监听数据库的binlog，获取需要操作的数据。

然后用一个公共的服务获取订阅程序传来的信息，进行缓存删除操作。 ![数据库订阅+消息队列保证key被删除](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-37c07418-9cd8-43d9-90e7-0cb43b329025.png) 这种方式降低了对业务的侵入，但其实整个系统的复杂度是提升的，适合基建完善的大厂。

**延时双删防止脏数据** 还有一种情况，是在缓存不存在的时候，写入了脏数据，这种情况在先删缓存，再更数据库的缓存更新策略下发生的比较多，解决方案是延时双删。

简单说，就是在第一次删除缓存之后，过了一段时间之后，再次删除缓存。

![延时双删](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-fab24753-9c53-4432-9413-5feba07ae1e3.png)延时双删

这种方式的延时时间设置需要仔细考量和测试。

**设置缓存过期时间兜底**

这是一个朴素但是有用的办法，给缓存设置一个合理的过期时间，即使发生了缓存数据不一致的问题，它也不会永远不一致下去，缓存过期的时候，自然又会恢复一致。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_29-如何保证本地缓存和分布式缓存的一致)29.如何保证本地缓存和分布式缓存的一致？

PS:这道题面试很少问，但实际工作中很常见。

在日常的开发中，我们常常采用两级缓存：本地缓存+分布式缓存。

所谓本地缓存，就是对应服务器的内存缓存，比如Caffeine，分布式缓存基本就是采用Redis。

那么问题来了，本地缓存和分布式缓存怎么保持数据一致？ ![延时双删](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-6d4ab7e6-8337-4576-bbf0-79202a1c3331.png) Redis缓存，数据库发生更新，直接删除缓存的key即可，因为对于应用系统而言，它是一种中心化的缓存。

但是本地缓存，它是非中心化的，散落在分布式服务的各个节点上，没法通过客户端的请求删除本地缓存的key，所以得想办法通知集群所有节点，删除对应的本地缓存key。 ![本地缓存/分布式缓存保持一致](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-20c15f0d-fb3c-4922-94b1-edcd856658be.png)

可以采用消息队列的方式：

1. 采用Redis本身的Pub/Sub机制，分布式集群的所有节点订阅删除本地缓存频道，删除Redis缓存的节点，同事发布删除本地缓存消息，订阅者们订阅到消息后，删除对应的本地key。 但是Redis的发布订阅不是可靠的，不能保证一定删除成功。
2. 引入专业的消息队列，比如RocketMQ，保证消息的可靠性，但是增加了系统的复杂度。
3. 设置适当的过期时间兜底，本地缓存可以设置相对短一些的过期时间。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_30-怎么处理热key)30.怎么处理热key？

> **什么是热Key？** 所谓的热key，就是访问频率比较的key。

比如，热门新闻事件或商品，这类key通常有大流量的访问，对存储这类信息的 Redis来说，是不小的压力。

假如Redis集群部署，热key可能会造成整体流量的不均衡，个别节点出现OPS过大的情况，极端情况下热点key甚至会超过 Redis本身能够承受的OPS。

> **怎么处理热key？**

![热key处理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-6fa972ec-5531-48f2-a608-4465d79d4518.png) 对热key的处理，最关键的是对热点key的监控，可以从这些端来监控热点key:

1. 客户端 客户端其实是距离key“最近”的地方，因为Redis命令就是从客户端发出的，例如在客户端设置全局字典（key和调用次数），每次调用Redis命令时，使用这个字典进行记录。
2. 代理端 像Twemproxy、Codis这些基于代理的Redis分布式架构，所有客户端的请求都是通过代理端完成的，可以在代理端进行收集统计。
3. Redis服务端 使用monitor命令统计热点key是很多开发和运维人员首先想到，monitor命令可以监控到Redis执行的所有命令。

只要监控到了热key，对热key的处理就简单了：

1. 把热key打散到不同的服务器，降低压⼒
2. 加⼊⼆级缓存，提前加载热key数据到内存中，如果redis宕机，⾛内存查询

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_31-缓存预热怎么做呢)31.缓存预热怎么做呢？

所谓缓存预热，就是提前把数据库里的数据刷到缓存里，通常有这些方法：

1、直接写个缓存刷新页面或者接口，上线时手动操作

2、数据量不大，可以在项目启动的时候自动进行加载

3、定时任务刷新缓存.

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_32-热点key重建-问题-解决)32.热点key重建？问题？解决？

开发的时候一般使用“缓存+过期时间”的策略，既可以加速数据读写，又保证数据的定期更新，这种模式基本能够满足绝大部分需求。

但是有两个问题如果同时出现，可能就会出现比较大的问题：

- 当前key是一个热点key（例如一个热门的娱乐新闻），并发量非常大。
- 重建缓存不能在短时间完成，可能是一个复杂计算，例如复杂的 SQL、多次IO、多个依赖等。 在缓存失效的瞬间，有大量线程来重建缓存，造成后端负载加大，甚至可能会让应用崩溃。

> **怎么处理呢？**

要解决这个问题也不是很复杂，解决问题的要点在于：

- 减少重建缓存的次数。
- 数据尽可能一致。
- 较少的潜在危险。

所以一般采用如下方式：

1. 互斥锁（mutex key） 这种方法只允许一个线程重建缓存，其他线程等待重建缓存的线程执行完，重新从缓存获取数据即可。
2. 永远不过期 “永远不过期”包含两层意思：

- 从缓存层面来看，确实没有设置过期时间，所以不会出现热点key过期后产生的问题，也就是“物理”不过期。
- 从功能层面来看，为每个value设置一个逻辑过期时间，当发现超过逻辑过期时间后，会使用单独的线程去构建缓存。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_33-无底洞问题吗-如何解决)33.无底洞问题吗？如何解决？

> **什么是无底洞问题？**

2010年，Facebook的Memcache节点已经达到了3000个，承载着TB级别的缓存数据。但开发和运维人员发现了一个问题，为了满足业务要求添加了大量新Memcache节点，但是发现性能不但没有好转反而下降了，当时将这 种现象称为缓存的“**无底洞**”现象。

那么为什么会产生这种现象呢?

通常来说添加节点使得Memcache集群 性能应该更强了，但事实并非如此。键值数据库由于通常采用哈希函数将 key映射到各个节点上，造成key的分布与业务无关，但是由于数据量和访问量的持续增长，造成需要添加大量节点做水平扩容，导致键值分布到更多的 节点上，所以无论是Memcache还是Redis的分布式，批量操作通常需要从不同节点上获取，相比于单机批量操作只涉及一次网络操作，分布式批量操作会涉及多次网络时间。

> **无底洞问题如何优化呢？**

先分析一下无底洞问题：

- 客户端一次批量操作会涉及多次网络操作，也就意味着批量操作会随着节点的增多，耗时会不断增大。
- 网络连接数变多，对节点的性能也有一定影响。

常见的优化思路如下：

- 命令本身的优化，例如优化操作语句等。
- 减少网络通信次数。
- 降低接入成本，例如客户端使用长连/连接池、NIO等。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#redis运维)Redis运维

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_34-redis报内存不足怎么处理)34.Redis报内存不足怎么处理？

Redis 内存不足有这么几种处理方式：

- 修改配置文件 redis.conf 的 maxmemory 参数，增加 Redis 可用内存
- 也可以通过命令set maxmemory动态设置内存上限
- 修改内存淘汰策略，及时释放内存空间
- 使用 Redis 集群模式，进行横向扩容。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_35-redis的过期数据回收策略有哪些)35.Redis的过期数据回收策略有哪些？

Redis主要有2种过期数据回收策略： ![在这里插入图片描述](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-de726c67-e56f-4f4b-96be-5cb998659541.png)

**惰性删除**

惰性删除指的是当我们查询key的时候才对key进⾏检测，如果已经达到过期时间，则删除。显然，他有⼀个缺点就是如果这些过期的key没有被访问，那么他就⼀直⽆法被删除，⽽且⼀直占⽤内存。

**定期删除**

定期删除指的是Redis每隔⼀段时间对数据库做⼀次检查，删除⾥⾯的过期key。由于不可能对所有key去做轮询来删除，所以Redis会每次随机取⼀些key去做检查和删除。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_36-redis有哪些内存溢出控制-内存淘汰策略)36.Redis有哪些内存溢出控制/内存淘汰策略？

Redis所用内存达到maxmemory上限时会触发相应的溢出控制策略，Redis支持六种策略： ![Redis六种内存溢出控制策略](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-5be7405c-ee11-4d2b-bea4-9598f10a1b17.png)

1. noeviction：默认策略，不会删除任何数据，拒绝所有写入操作并返 回客户端错误信息，此 时Redis只响应读操作。
2. volatile-lru：根据LRU算法删除设置了超时属性（expire）的键，直 到腾出足够空间为止。如果没有可删除的键对象，回退到noeviction策略。
3. allkeys-lru：根据LRU算法删除键，不管数据有没有设置超时属性， 直到腾出足够空间为止。
4. allkeys-random：随机删除所有键，直到腾出足够空间为止。
5. volatile-random：随机删除过期键，直到腾出足够空间为止。
6. volatile-ttl：根据键值对象的ttl属性，删除最近将要过期数据。如果 没有，回退到noeviction策略。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_37-redis阻塞-怎么解决)37.Redis阻塞？怎么解决？

Redis发生阻塞，可以从以下几个方面排查： ![Redis阻塞排查](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e6a35258-7a78-4489-90b7-e47a4190802b.png)

- **API或数据结构使用不合理**

  通常Redis执行命令速度非常快，但是不合理地使用命令，可能会导致执行速度很慢，导致阻塞，对于高并发的场景，应该尽量避免在大对象上执行算法复杂 度超过O（n）的命令。

  对慢查询的处理分为两步：

  1. 发现慢查询： slowlog get{n}命令可以获取最近 的n条慢查询命令；
  2. 发现慢查询后，可以从两个方向去优化慢查询： 1）修改为低算法复杂度的命令，如hgetall改为hmget等，禁用keys、sort等命 令 2）调整大对象：缩减大对象数据或把大对象拆分为多个小对象，防止一次命令操作过多的数据。

- **CPU饱和的问题**

  单线程的Redis处理命令时只能使用一个CPU。而CPU饱和是指Redis单核CPU使用率跑到接近100%。

  针对这种情况，处理步骤一般如下：

  1. 判断当前Redis并发量是否已经达到极限，可以使用统计命令redis-cli-h{ip}-p{port}--stat获取当前 Redis使用情况
  2. 如果Redis的请求几万+，那么大概就是Redis的OPS已经到了极限，应该做集群化水品扩展来分摊OPS压力
  3. 如果只有几百几千，那么就得排查命令和内存的使用

- **持久化相关的阻塞**

  对于开启了持久化功能的Redis节点，需要排查是否是持久化导致的阻塞。

  1. fork阻塞 fork操作发生在RDB和AOF重写时，Redis主线程调用fork操作产生共享 内存的子进程，由子进程完成持久化文件重写工作。如果fork操作本身耗时过长，必然会导致主线程的阻塞。
  2. AOF刷盘阻塞 当我们开启AOF持久化功能时，文件刷盘的方式一般采用每秒一次，后台线程每秒对AOF文件做fsync操作。当硬盘压力过大时，fsync操作需要等 待，直到写入完成。如果主线程发现距离上一次的fsync成功超过2秒，为了 数据安全性它会阻塞直到后台线程执行fsync操作完成。
  3. HugePage写操作阻塞 对于开启Transparent HugePages的 操作系统，每次写命令引起的复制内存页单位由4K变为2MB，放大了512 倍，会拖慢写操作的执行时间，导致大量写操作慢查询。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_38-大key问题了解吗)38.大key问题了解吗？

Redis使用过程中，有时候会出现大key的情况， 比如：

- 单个简单的key存储的value很大，size超过10KB
- hash， set，zset，list 中存储过多的元素（以万为单位）

> **大key会造成什么问题呢？**

- 客户端耗时增加，甚至超时
- 对大key进行IO操作时，会严重占用带宽和CPU
- 造成Redis集群中数据倾斜
- 主动删除、被动删等，可能会导致阻塞

> **如何找到大key?**

- bigkeys命令：使用bigkeys命令以遍历的方式分析Redis实例中的所有Key，并返回整体统计信息与每个数据类型中Top1的大Key
- redis-rdb-tools：redis-rdb-tools是由Python写的用来分析Redis的rdb快照文件用的工具，它可以把rdb快照文件生成json文件或者生成报表用来分析Redis的使用详情。

> **如何处理大key?**

![大key处理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-e4aaafda-fce1-47f0-8b2b-7261d47b720b.png)大key处理

- **删除大key**
  - 当Redis版本大于4.0时，可使用UNLINK命令安全地删除大Key，该命令能够以非阻塞的方式，逐步地清理传入的Key。
  - 当Redis版本小于4.0时，避免使用阻塞式命令KEYS，而是建议通过SCAN命令执行增量迭代扫描key，然后判断进行删除。
- **压缩和拆分key**
  - 当vaule是string时，比较难拆分，则使用序列化、压缩算法将key的大小控制在合理范围内，但是序列化和反序列化都会带来更多时间上的消耗。
  - 当value是string，压缩之后仍然是大key，则需要进行拆分，一个大key分为不同的部分，记录每个部分的key，使用multiget等操作实现事务读取。
  - 当value是list/set等集合类型时，根据预估的数据规模来进行分片，不同的元素计算后分到不同的片。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/redis.html#_39-redis常见性能问题和解决方案)39.Redis常见性能问题和解决方案？

1. Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化。
2. 如果数据比较关键，某个 Slave 开启 AOF 备份数据，策略为每秒同步一次。
3. 为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。
4. 尽量避免在压力较大的主库上增加从库。
5. Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的 CPU 和内存资源，导致服务 load 过高，出现短暂服务暂停现象。
6. 为了 Master 的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现 Slave 对 Master 的替换，也即，如果 Master 挂了，可以立马启用 Slave1 做 Master，其他不变。



# Redis Interview2



Redis(Remote Dictionary Server) 是一个使用 C 语言编写的，开源的（BSD 许可）高性能非关系型（NoSQL）的键值对数据库。

Redis 可以存储键和五种不同类型的值之间的映射。键的类型只能为字符串，值支持五种数据类型**：字符串、列表、集合、散列表、有序集合。**

与传统数据库不同的是 Redis 的数据是存在内存中的，所以读写速度非常快，因此 redis 被广泛应用于缓存方向，每秒可以处理超过 ***10 万次***读写操作，是已知性能最快的 Key-Value DB。另外，Redis 也经常用来做分布式锁。除此之外，Redis 支持事务 、持久化、LUA 脚本、LRU 驱动事件、多种集群方案。

今天就来讲讲 Redis 的面试题，为复工后的面试做好准备。

![img](https://static001.geekbang.org/infoq/d6/d6b82f8604b76cf0fb6311f5db66a916.png)



![img](https://static001.geekbang.org/infoq/99/997cb8b59417c344c04d9eba3d505704.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## 概述



### **1、Redis 有哪些优缺点**

**优点**

- 读写性能优异， Redis 能读的速度是 110000 次/s，写的速度是 81000 次/s。
- 支持数据持久化，支持 AOF 和 RDB 两种持久化方式。
- 支持事务，Redis 的所有操作都是原子性的，同时 Redis 还支持对几个操作合并后的原子性执行。
- 数据结构丰富，除了支持 string 类型的 value 外还支持 hash、set、zset、list 等数据结构。
- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。

**缺点**

- 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此 Redis 适合的场景主要局限在较小数据量的高性能操作和运算上。
- Redis 不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复。
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换 IP 后还会引入数据不一致的问题，降低了系统的可用性。
- Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

### **2、为什么要用 Redis /为什么要用缓存**

主要从“高性能”和“高并发”这两点来看待这个问题。

- **高性能：**

假如用户第一次访问数据库中的某些数据。这个过程会比较慢，因为是从硬盘上读取的。将该用户访问的数据存在数缓存中，这样下一次再访问这些数据的时候就可以直接从缓存中获取了。**操作缓存就是直接操作内存，所以速度相当快。如果数据库中的对应数据改变的之后，同步改变缓存中相应的数据即可！**

![img](https://static001.geekbang.org/infoq/98/98ee674b09e7f48003910fff0449cdf4.png)



- 高并发：

直接操作缓存能够承受的请求是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。

![img](https://static001.geekbang.org/infoq/48/4810c762fa75a456bfe67c4caaf49ffe.png)



### **3、为什么要用 Redis 而不用 map/guava 做缓存?**

缓存分为本地缓存和分布式缓存。以 Java 为例，使用自带的 map 或者 guava 实现的是本地缓存，最主要的特点是轻量以及快速，生命周期随着 jvm 的销毁而结束，并且在多实例的情况下，每个实例都需要各自保存一份缓存，缓存不具有一致性。

使用 redis 或 memcached 之类的称为分布式缓存，在多实例的情况下，各实例共用一份缓存数据，缓存具有一致性。缺点是需要保持 redis 或 memcached 服务的高可用，整个程序架构上较为复杂。

### **4、Redis 为什么这么快**

1）完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度都是 O(1)；

2）数据结构简单，对数据操作也简单，Redis 中的数据结构是专门进行设计的；

3）采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

4）使用多路 I/O 复用模型，非阻塞 IO；

> IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的 select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听 Socket 和已连接 Socket。内核会一直监听这些 Socket 上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO流的效果

5 使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis 直接自己构建了 VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

![img](https://static001.geekbang.org/infoq/5c/5c09b7a9590e383ea169c4abfc2c98e3.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



**数据类型**



### **5、Redis 有哪些数据类型**

Redis 主要有 5 种数据类型，包括 String，List，Set，Zset，Hash，满足大部分的使用要求

![img](https://static001.geekbang.org/infoq/7b/7b653b4b2fe6091bac51ccbbf32a006a.png)



### **6、Redis 的应用场景**

- **总结一**

**计数器**：可以对 String 进行自增自减运算，从而实现计数器功能。Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

**缓存**：将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

**会话缓存**：可以使用 Redis 来统一存储多台应用服务器的会话信息。当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

**全页缓存（FPC）**：除基本的会话 token 之外，Redis 还提供很简便的 FPC 平台。以 Magento 为例，Magento 提供一个插件来使用 Redis 作为全页缓存后端。此外，对 WordPress 的用户来说，Pantheon 有一个非常好的插件 wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

**查找表**：例如 DNS 记录就很适合使用 Redis 进行存储。查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

**消息队列(发布/订阅功能)：**List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息。不过最好使用 Kafka、RabbitMQ 等消息中间件。

**分布式锁实现：**在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

**其它：**Set 可以实现交集、并集等操作，从而实现共同好友等功能。ZSet 可以实现有序性操作，从而实现排行榜等功能。

- **总结二**

Redis 相比其他缓存，有一个非常大的优势，就是支持多种数据类型。

数据类型说明 string 字符串，最简单的 k-v 存储 hashhash 格式，value 为 field 和 value，适合 ID-Detail 这样的场景。list 简单的 list，顺序列表，支持首位或者末尾插入数据 set 无序 list，查找速度快，适合交集、并集、差集处理 sorted set 有序的 set

其实，通过上面的数据类型的特性，基本就能想到合适的应用场景了。

string——适合最简单的 k-v 存储，类似于 memcached 的存储结构，短信验证码，配置信息等，就用这种类型来存储。

**hash**——一般 key 为 ID 或者唯一标示，value 对应的就是详情了。如商品详情，个人信息详情，新闻详情等。

**list**——因为 list 是有序的，比较适合存储一些有序且数据相对固定的数据。如省市区表、字典表等。因为 list 是有序的，适合根据写入的时间来排序，如：最新的***，消息队列等。

**set**——可以简单的理解为 ID-List 的模式，如微博中一个人有哪些好友，set 最牛的地方在于，可以对两个 set 提供交集、并集、差集操作。例如：查找两个人共同的好友等。

**Sorted Set**——是 set 的增强版本，增加了一个 score 参数，自动会根据 score 的值进行排序。比较适合类似于 top 10 等不根据插入的时间来排序的数据。

如上所述，虽然 Redis 不像关系数据库那么复杂的数据结构，但是，也能适合很多场景，比一般的缓存数据结构要多。了解每种数据结构适合的业务场景，不仅有利于提升开发效率，也能有效利用 Redis 的性能。

![img](https://static001.geekbang.org/infoq/80/80229ad3a56a5c3c3b6938f5bb863514.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **持久化**



### **7、什么是 Redis 持久化？**

持久化就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失。

### **8、Redis 的持久化机制是什么？各自的优缺点？**

Redis 提供两种持久化机制 RDB（默认） 和 AOF 机制:

**RDB：**是 Redis DataBase 缩写快照

RDB 是 Redis 默认的持久化方式。按照一定的时间将内存的数据以快照的形式保存到硬盘中，对应产生的数据文件为 dump.rdb。通过配置文件中的 save 参数来定义快照的周期。

![img](https://static001.geekbang.org/infoq/af/af1dbd9d0c8839a65448d0f6c44b4513.png)



**优点：**

1、只有一个文件 dump.rdb，方便持久化。

2、容灾性好，一个文件可以保存到安全的磁盘。

3、性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，所以是 IO 最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO 操作，保证了 redis 的高性能

4.相对于数据集大时，比 AOF 的启动效率更高。

**缺点：**

1、数据安全性低。RDB 是间隔一段时间进行持久化，如果持久化之间 redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候)

2、AOF（Append-only file)持久化方式：是指所有的命令行记录以 redis 命令请 求协议的格式完全持久化存储)保存为 aof 文件。

**AOF：持久化**

AOF 持久化(即 Append Only File 持久化)，则是将 Redis 执行的每次写命令记录到单独的日志文件中，当重启 Redis 会重新将持久化的日志中文件恢复数据。

当两种方式同时开启时，数据恢复 Redis 会优先选择 AOF 恢复。

![img](https://static001.geekbang.org/infoq/4c/4c2c0b26896688c57675531a085086e1.jpeg?x-oss-process=image%2Fresize%2Cp_80%2Fauto-orient%2C1)



- **优点：**

1、数据安全，aof 持久化可以配置 appendfsync 属性，有 always，每进行一次 命令操作就记录到 aof 文件中一次。

2、通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。

3、AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令 进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）)

- **缺点：**

1、AOF 文件比 RDB 文件大，且恢复速度慢。

2、数据集大的时候，比 rdb 启动效率低。

**优缺点是什么？**

AOF 文件比 RDB 更新频率高，优先使用 AOF 还原数据。

AOF 比 RDB 更安全也更大

RDB 性能比 AOF 好

如果两个都配了优先加载 AOF

**9、如何选择合适的持久化方式**

- 一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性，你应该同时使用两种持久化功能。在这种情况下，当 Redis 重启的时候会优先载入 AOF 文件来恢复原始的数据，因为在通常情况下 AOF 文件保存的数据集要比 RDB 文件保存的数据集要完整。
- 如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失，那么你可以只使用 RDB 持久化。
- 有很多用户都只使用 AOF 持久化，但并不推荐这种方式，因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快，除此之外，使用 RDB 还可以避免 AOF 程序的 bug。
- 如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化方式。

**10、Redis 持久化数据和缓存怎么做扩容？**

- 如果 Redis 被当做缓存使用，使用一致性哈希实现动态扩容缩容。
- 如果 Redis 被当做一个持久化存储使用，必须使用固定的 keys-to-nodes 映射关系，节点的数量一旦确定不能变化。否则的话(即 Redis 节点需要动态变化的情况），必须使用可以在运行时进行数据再平衡的一套系统，而当前只有 Redis 集群可以做到这样。



![img](https://static001.geekbang.org/infoq/12/121fd479c3f5ba4182cf3dfaaf1e206f.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **过期键的删除策略**



**11、Redis 的过期键的删除策略**

我们都知道，Redis 是 key-value 数据库，我们可以设置 Redis 中缓存的 key 的过期时间。Redis 的过期策略就是指当 Redis 中缓存的 key 过期了，Redis 如何处理。

过期策略通常有以下三种：

- **定时过期**：每个设置过期时间的 key 都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的 CPU 资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
- **惰性过期**：只有当访问一个 key 时，才会判断该 key 是否已过期，过期则清除。该策略可以最大化地节省 CPU 资源，却对内存非常不友好。极端情况可能出现大量的过期 key 没有再次被访问，从而不会被清除，占用大量内存。
- **定期过期**：每隔一定的时间，会扫描一定数量的数据库的 expires 字典中一定数量的 key，并清除其中已过期的 key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得 CPU 和内存资源达到最优的平衡效果。

(expires 字典会保存所有设置了过期时间的 key 的过期时间数据，其中，key 是指向键空间中的某个键的指针，value 是该键的毫秒精度的 UNIX 时间戳表示的过期时间。键空间是指该 Redis 集群中保存的所有键。)

Redis 中同时使用了惰性过期和定期过期两种过期策略。

### **12、Redis key 的过期时间和永久有效分别怎么设置？**

EXPIRE 和 PERSIST 命令。

**13、我们知道通过 expire 来设置 key 的过期时间，那么对过期的数据怎么处理呢?**

除了缓存服务器自带的缓存失效策略之外（Redis 默认的有 6 中策略可供选择），我们还可以根据具体的业务需求进行自定义的缓存淘汰，常见的策略有两种：

- 定时去清理过期的缓存；
- 当有用户请求过来时，再判断这个请求所用到的缓存是否过期，过期的话就去底层系统得到新数据并更新缓存。

两者各有优劣，第一种的缺点是维护大量缓存的 key 是比较麻烦的，第二种的缺点就是每次用户请求过来都要判断缓存失效，逻辑相对比较复杂！具体用哪种方案，大家可以根据自己的应用场景来权衡。

![img](https://static001.geekbang.org/infoq/b3/b32c6f8937a98057eb2fbb9df86c1c61.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **内存相关**



#### **14、MySQL 里有 2000w 数据，redis 中只存 20w 的数据，如何保证 redis 中的数据都是热点数据？**

redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。

#### **15、Redis 的内存淘汰策略有哪些？**

Redis 的内存淘汰策略是指在 Redis 的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。

**全局的键空间选择性移除**

- **noeviction**：当内存不足以容纳新写入数据时，新写入操作会报错。
- **allkeys-lru**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key。（这个是最常用的）
- **allkeys-random**：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key。

**设置过期时间的键空间选择性移除**

- **volatile-lru**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key。
- **volatile-random：**当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。
- **volatile-ttl：**当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。

**总结**

Redis 的内存淘汰策略的选取并不会影响过期的 key 的处理。内存淘汰策略用于处理内存不足时的需要申请额外空间的数据；过期策略用于处理过期的缓存数据。

#### **16、Redis 主要消耗什么物理资源？**

内存。

#### **17、Redis 的内存用完了会发生什么？**

如果达到设置的上限，Redis 的写命令会返回错误信息（但是读命令还可以正常返回。）或者你可以配置内存淘汰机制，当 Redis 达到内存上限时会冲刷掉旧的内容。

#### **18、Redis 如何做内存优化？**

可以好好利用 Hash,list,sorted set,set 等集合类型数据，因为通常情况下很多小的 Key-Value 可以用更紧凑的方式存放到一起。尽可能使用散列表（hashes），散列表（是说散列表里面存储的数少）使用的内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。比如你的 web 系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设置单独的 key，而是应该把这个用户的所有信息存储到一张散列表里面。

![img](https://static001.geekbang.org/infoq/aa/aa4f183c62bf67f5bdd27af82e952d1a.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **线程模型**

**19、Redis 线程模型**

Redis 基于 Reactor 模式开发了网络事件处理器，这个处理器被称为文件事件处理器（file event handler）。它的组成结构为 4 部分：多个套接字、IO 多路复用程序、文件事件分派器、事件处理器。因为文件事件分派器队列的消费是单线程的，所以 Redis 才叫单线程模型。

- 文件事件处理器使用 I/O 多路复用（multiplexing）程序来同时监听多个套接字， 并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
- 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时， 与操作相对应的文件事件就会产生， 这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

虽然文件事件处理器以单线程方式运行， 但通过使用 I/O 多路复用程序来监听多个套接字， 文件事件处理器既实现了高性能的网络通信模型， 又可以很好地与 redis 服务器中其他同样以单线程方式运行的模块进行对接， 这保持了 Redis 内部单线程设计的简单性。



## **事务**

**20、什么是事务？**

事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。

**21、Redis 事务的概念**

Redis 事务的本质是通过 MULTI、EXEC、WATCH 等一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis 事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

**22、Redis 事务的三个阶段**

- **事务开始 MULTI**
- **命令入队**
- **事务执行 EXEC**

事务执行过程中，如果服务端收到有 EXEC、DISCARD、WATCH、MULTI 之外的请求，将会把请求放入队列中排队。

**23、Redis 事务相关命令**

Redis 事务功能是通过 MULTI、EXEC、DISCARD 和 WATCH 四个原语实现的。

Redis 会将一个事务中的所有命令序列化，然后按顺序执行。

**1）**redis 不支持回滚，“Redis 在事务失败时不进行回滚，而是继续执行余下的命令”， 所以 Redis 的内部可以保持简单且快速。

**2）**如果在一个事务中的命令出现错误，那么所有的命令都不会执行；

**.3）**如果在一个事务中出现运行错误，那么正确的命令会被执行。

- WATCH 命令是一个乐观锁，可以为 Redis 事务提供 check-and-set （CAS）行为。可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到 EXEC 命令。
- MULTI 命令用于开启一个事务，它总是返回 OK。MULTI 执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当 EXEC 命令被调用时，所有队列中的命令才会被执行。
- EXEC：执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。当操作被打断时，返回空值 nil 。
- 通过调用 DISCARD，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。
- UNWATCH 命令可以取消 watch 对所有 key 的监控。

**24、事务管理（ACID）概述**

**原子性（Atomicity）：**原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。

**一致性（Consistency）：**事务前后数据的完整性必须保持一致。

**隔离性（Isolation）：**多个事务并发执行时，一个事务的执行不应影响其他事务的执行。

**持久性（Durability）：**持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

**Redis 的事务总是具有 ACID 中的一致性和隔离性，**其他特性是不支持的。当服务器运行在 AOF 持久化模式下，并且 appendfsync 选项的值为 always 时，事务也具有耐久性。

**25、Redis 事务支持隔离性吗？**

Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。因此，Redis 的事务是总是带有隔离性的。

**26、Redis 事务保证原子性吗，支持回滚吗？**

Redis 中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。

**27、Redis 事务其他实现**

- 基于 Lua 脚本，Redis 可以保证脚本内的命令一次性、按顺序地执行，其同时也不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完
- 基于中间标记变量，通过另外的标记变量来标识事务是否执行完成，读取数据时先读取该标记变量判断是否事务执行完成。但这样会需要额外写代码实现，比较繁琐。



![img](https://static001.geekbang.org/infoq/67/672b615e5707e26275b9044da4c3682c.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **集群方案**



### **28、哨兵模式**

![img](https://static001.geekbang.org/infoq/36/36d9b8cbf449aa379ad7bba7faa7361f.jpeg?x-oss-process=image%2Fresize%2Cp_80%2Fauto-orient%2C1)



**哨兵的介绍：**

sentinel，中文名是哨兵。哨兵是 redis 集群机构中非常重要的一个组件，主要有以下功能：

- **集群监控**：负责监控 redis master 和 slave 进程是否正常工作。
- **消息通知**：如果某个 redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
- **故障转移**：如果 master node 挂掉了，会自动转移到 slave node 上。
- **配置中心**：如果故障转移发生了，通知 client 客户端新的 master 地址。

**哨兵用于实现 redis 集群的高可用**，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作。

- 故障转移时，判断一个 master node 是否宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题。
- 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的，因为如果一个作为高可用机制重要组成部分的故障转移系统本身是单点的，那就很坑爹了。

**哨兵的核心知识**

- 哨兵至少需要 3 个实例，来保证自己的健壮性。
- 哨兵 + redis 主从的部署架构，是不保证数据零丢失的，只能保证 redis 集群的高可用性。
- 对于哨兵 + redis 主从这种复杂的部署架构，尽量在测试环境和生产环境，都进行充足的测试和演练。

### **29、官方 Redis Cluster 方案(服务端路由查询)**

![img](https://static001.geekbang.org/infoq/e6/e6bb14f713b45c6e1bc35059e8c2b476.png)



redis 集群模式的工作原理能说一下么？在集群模式下，redis 的 key 是如何寻址的？分布式寻址都有哪些算法？了解一致性 hash 算法吗？

**简介**

Redis Cluster 是一种服务端 Sharding 技术，3.0 版本开始正式提供。Redis Cluster 并没有使用一致性 hash，而是采用 slot(槽)的概念，一共分成 16384 个槽。将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行

**方案说明**

- 通过哈希的方式，将数据分片，每个节点均分存储一定哈希槽(哈希值)区间的数据，默认分配了 16384 个槽位
- 每份数据分片会存储在多个互为主从的多节点上
- 数据写入先写主节点，再同步到从节点(支持配置为阻塞同步)
- 同一分片多个节点间的数据不保持一致性
- 读取数据时，当客户端操作的 key 没有分配在该节点上时，redis 会返回转向指令，指向正确的节点
- 扩容时时需要需要把旧节点的数据迁移一部分到新节点

在 redis cluster 架构下，每个 redis 要放开两个端口号，比如一个是 6379，另外一个就是 加 1w 的端口号，比如 16379。

16379 端口号是用来进行节点间通信的，也就是 cluster bus 的东西，cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议，gossip 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

**节点间的内部通信机制**

（基本通信原理）集群元数据的维护有两种方式：集中式、Gossip 协议。redis cluster 节点间采用 gossip 协议进行通信。

**分布式寻址算法**

- hash 算法（大量缓存重建）
- 一致性 hash 算法（自动缓存迁移）+ 虚拟节点（自动负载均衡）
- redis cluster 的 hash slot 算法

**优点**

- 无中心架构，支持动态扩容，对业务透明
- 具备 Sentinel 的监控和自动 Failover(故障转移)能力
- 客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可
- 高性能，客户端直连 redis 服务，免去了 proxy 代理的损耗

**缺点**

- 运维也很复杂，数据迁移需要人工干预
- 只能使用 0 号数据库
- 不支持批量操作(pipeline 管道操作)
- 分布式逻辑和存储模块耦合等

**30、基于客户端分配**

![img](https://static001.geekbang.org/infoq/32/32572ce74d2e0f9afd40b0e0a91ef8b5.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



**简介**

Redis Sharding 是 Redis Cluster 出来之前，业界普遍使用的多 Redis 实例集群方法。其主要思想是采用哈希算法将 Redis 数据的 key 进行散列，通过 hash 函数，特定的 key 会映射到特定的 Redis 节点上。Java redis 客户端驱动 jedis，支持 Redis Sharding 功能，即 ShardedJedis 以及结合缓存池的 ShardedJedisPool

**优点**

优势在于非常简单，服务端的 Redis 实例彼此独立，相互无关联，每个 Redis 实例像单服务器一样运行，非常容易线性扩展，系统的灵活性很强

**缺点**

由于 sharding 处理放到客户端，规模进一步扩大时给运维带来挑战。

客户端 sharding 不支持动态增删节点。服务端 Redis 实例群拓扑结构有变化时，每个客户端都需要更新调整。连接不能共享，当应用规模增大时，资源浪费制约优化

### **31、基于代理服务器分片**

![img](https://static001.geekbang.org/infoq/46/4602b3bfa1b46867abee04ba06491ae0.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



**简介**

客户端发送请求到一个代理组件，代理解析客户端的数据，并将请求转发至正确的节点，最后将结果回复给客户端

**特征**

- 透明接入，业务程序不用关心后端 Redis 实例，切换成本低
- Proxy 的逻辑和存储的逻辑是隔离的
- 代理层多了一次转发，性能有所损耗

**业界开源方案**

Twtter 开源的 Twemproxy

豌豆荚开源的 Codis

### **32、Redis 主从架构**

单机的 redis，能够承载的 QPS 大概就在上万到几万不等。对于缓存来说，一般都是用来支撑读高并发的。因此架构做成主从(master-slave)架构，一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读。所有的读请求全部走从节点。这样也可以很轻松实现水平扩容，**支撑读高并发。**

![img](https://static001.geekbang.org/infoq/03/038df579d464dc923e9847aa1e446470.png)



redis replication -> 主从架构 -> 读写分离 -> 水平扩容支撑读高并发

**redis replication 的核心机制**

- redis 采用异步方式复制数据到 slave 节点，不过 redis2.8 开始，slave node 会周期性地确认自己每次复制的数据量；
- 一个 master node 是可以配置多个 slave node 的；
- slave node 也可以连接其他的 slave node；
- slave node 做复制的时候，不会 block master node 的正常工作；
- slave node 在做复制的时候，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了；
- slave node 主要用来进行横向扩容，做读写分离，扩容的 slave node 可以提高读的吞吐量。

注意，如果采用了主从架构，那么建议必须开启 master node 的持久化，不建议用 slave node 作为 master node 的数据热备，因为那样的话，如果你关掉 master 的持久化，可能在 master 宕机重启的时候数据是空的，然后可能一经过复制， slave node 的数据也丢了。

另外，master 的各种备份方案，也需要做。万一本地的所有文件丢失了，从备份中挑选一份 rdb 去恢复 master，这样才能**确保启动的时候，是有数据的**，即使采用了后续讲解的高可用机制，slave node 可以自动接管 master node，但也可能 sentinel 还没检测到 master failure，master node 就自动重启了，还是可能导致上面所有的 slave node 数据被清空。

**redis 主从复制的核心原理**

当启动一个 slave node 的时候，它会发送一个 PSYNC 命令给 master node。

如果这是 slave node 初次连接到 master node，那么会触发一次 full resynchronization 全量复制。此时 master 会启动一个后台线程，开始生成一份 RDB 快照文件。

同时还会将从客户端 client 新收到的所有写命令缓存在内存中。RDB 文件生成完毕后， master 会将这个 RDB 发送给 slave，slave 会先**写入本地磁盘，然后再从本地磁盘加载到内存中。**

接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。

slave node 如果跟 master node 有网络故障，断开了连接，会自动重连，连接之后 master node 仅会复制给 slave 部分缺少的数据。

![img](https://static001.geekbang.org/infoq/a7/a77de7365d59e0899a4cad8d358006ae.png)



**过程原理**

- 当从库和主库建立 MS 关系后，会向主数据库发送 SYNC 命令
- 主库接收到 SYNC 命令后会开始在后台保存快照(RDB 持久化过程)，并将期间接收到的写命令缓存起来
- 当快照完成后，主 Redis 会将快照文件和所有缓存的写命令发送给从 Redis
- 从 Redis 接收到后，会载入快照文件并且执行收到的缓存的命令
- 之后，主 Redis 每当接收到写命令时就会将命令发送从 Redis，从而保证数据的一致

**缺点**

所有的 slave 节点数据的复制和同步都由 master 节点来处理，会照成 master 节点压力太大，使用主从从结构来解决

**33、Redis 集群的主从复制模型是怎样的？**

为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型，每个节点都会有 N-1 个复制品

**34、生产环境中的 redis 是怎么部署的？**

redis cluster，10 台机器，5 台机器部署了 redis 主实例，另外 5 台机器部署了 redis 的从实例，每个主实例挂了一个从实例，5 个节点对外提供读写服务，每个节点的读写高峰 qps 可能可以达到每秒 5 万，5 台机器最多是 25 万读写请求/s。

机器是什么配置？32G 内存+ 8 核 CPU + 1T 磁盘，但是分配给 redis 进程的是 10g 内存，一般线上生产环境，redis 的内存尽量不要超过 10g，超过 10g 可能会有问题。

5 台机器对外提供读写，一共有 50g 内存。

因为每个主实例都挂了一个从实例，所以是高可用的，任何一个主实例宕机，都会自动故障迁移，redis 从实例会自动变成主实例继续提供读写服务。

你往内存里写的是什么数据？每条数据的大小是多少？商品数据，每条数据是 10kb。100 条数据是 1mb，10 万条数据是 1g。常驻内存的是 200 万条商品数据，占用内存是 20g，仅仅不到总内存的 50%。目前高峰期每秒就是 3500 左右的请求量。

其实大型的公司，会有基础架构的 team 负责缓存集群的运维。

**35、说说 Redis 哈希槽的概念？**

Redis 集群没有使用一致性 hash,而是引入了哈希槽的概念，Redis 集群有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽，集群的每个节点负责一部分 hash 槽。

**36、Redis 集群会有写操作丢失吗？为什么？**

Redis 并不能保证数据的强一致性，这意味这在实际中集群在特定的条件下可能会丢失写操作。

**37、Redis 集群之间是如何复制的？**

异步复制

**38、Redis 集群最大节点个数是多少？**

16384 个

**39、Redis 集群如何选择数据库？**

Redis 集群目前无法做数据库选择，默认在 0 数据库。

![img](https://static001.geekbang.org/infoq/48/48652b547ca5cfae7400513628bacce5.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **分区**  提高利用率



**40、Redis 是单线程的，如何提高多核 CPU 的利用率？**

可以在同一个服务器部署多个 Redis 的实例，并把他们当作不同的服务器来使用，在某些时候，无论如何一个服务器是不够的， 所以，如果你想使用多个 CPU，你可以考虑一下分片（shard）。

**41、为什么要做 Redis 分区？**

分区可以让 Redis 管理更大的内存，Redis 将可以使用所有机器的内存。如果没有分区，你最多只能使用一台机器的内存。分区使 Redis 的计算能力通过简单地增加计算机得到成倍提升，Redis 的网络带宽也会随着计算机和网卡的增加而成倍增长。

**42、你知道有哪些 Redis 分区实现方案？**

- 客户端分区就是在客户端就已经决定数据会被存储到哪个 redis 节点或者从哪个 redis 节点读取。大多数客户端已经实现了客户端分区。
- 代理分区 意味着客户端将请求发送给代理，然后代理决定去哪个节点写数据或者读数据。代理根据分区规则决定请求哪些 Redis 实例，然后根据 Redis 的响应结果返回给客户端。redis 和 memcached 的一种代理实现就是 Twemproxy
- 查询路由(Query routing) 的意思是客户端随机地请求任意一个 redis 实例，然后由 Redis 将请求转发给正确的 Redis 节点。Redis Cluster 实现了一种混合形式的查询路由，但并不是直接将请求从一个 redis 节点转发到另一个 redis 节点，而是在客户端的帮助下直接 redirected 到正确的 redis 节点。

**43、Redis 分区有什么缺点？**

- 涉及多个 key 的操作通常不会被支持。例如你不能对两个集合求交集，因为他们可能被存储到不同的 Redis 实例（实际上这种情况也有办法，但是不能直接使用交集指令）。
- 同时操作多个 key,则不能使用 Redis 事务.
- 分区使用的粒度是 key，不能使用一个非常长的排序 key 存储一个数据集（The partitioning granularity is the key, so it is not possible to shard a dataset with a single huge key like a very big sorted set）
- 当使用分区的时候，数据处理会非常复杂，例如为了备份你必须从不同的 Redis 实例和主机同时收集 RDB / AOF 文件。
- 分区时动态扩容或缩容可能非常复杂。Redis 集群在运行时增加或者删除 Redis 节点，能做到最大程度对用户透明地数据再平衡，但其他一些客户端分区或者代理分区方法则不支持这种特性。然而，有一种预分片的技术也可以较好的解决这个问题。



![img](https://static001.geekbang.org/infoq/a5/a52050269fb55d9b3ee427039d08d8c2.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **分布式问题**



### **44、Redis 实现分布式锁**

Redis 为单进程单线程模式，采用队列模式将并发访问变成串行访问，且多客户端对 Redis 的连接并不存在竞争关系 Redis 中可以使用 SETNX 命令实现分布式锁。

当且仅当 key 不存在，将 key 的值设为 value。若给定的 key 已经存在，则 SETNX 不做任何动作。

SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。

返回值：设置成功，返回 1 。设置失败，返回 0 。

![img](https://static001.geekbang.org/infoq/e4/e48cc370a02dd6aa4eddd1abe1776d33.png)



**使用 SETNX 完成同步锁的流程及事项如下：**

使用 SETNX 命令获取锁，若返回 0（key 已存在，锁已存在）则获取失败，反之获取成功。

为了防止获取锁后程序出现异常，导致其他线程/进程调用 SETNX 命令总是返回 0 而进入死锁状态，需要为该 key 设置一个“合理”的过期时间。

释放锁，使用 DEL 命令将锁数据删除。

### **45、如何解决 Redis 的并发竞争 Key 问题**

所谓 Redis 的并发竞争 Key 的问题也就是多个系统同时对一个 key 进行操作，但是最后执行的顺序和我们期望的顺序不同，这样也就导致了结果的不同！

**推荐一种方案：分布式锁**（zookeeper 和 redis 都可以实现分布式锁）。（如果不存在 Redis 的并发竞争 Key 问题，不要使用分布式锁，这样会影响性能）

基于 zookeeper 临时有序节点可以实现的分布式锁。大致思想为：每个客户端对某个方法加锁时，在 zookeeper 上的与该方法对应的指定节点的目录下，生成一个唯一的瞬时有序节点。判断是否获取锁的方式很简单，只需要判断有序节点中序号最小的一个。当释放锁的时候，只需将这个瞬时节点删除即可。同时，其可以避免服务宕机导致的锁无法释放，而产生的死锁问题。完成业务流程后，删除对应的子节点释放锁。

在实践中，当然是从以可靠性为主。所以首推 Zookeeper。

参考：[https://www.jianshu.com/p/8bddd381de06](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F8bddd381de06)

### **46、分布式 Redis 是前期做还是后期规模上来了再做好？为什么？**

既然 Redis 是如此的轻量（单实例只使用 1M 内存），为防止以后的扩容，最好的办法就是一开始就启动较多实例。即便你只有一台服务器，你也可以一开始就让 Redis 以分布式的方式运行，使用分区，在同一台服务器上启动多个实例。

一开始就多设置几个 Redis 实例，例如 32 或者 64 个实例，对大多数用户来说这操作起来可能比较麻烦，但是从长久来看做这点牺牲是值得的。

这样的话，当你的数据不断增长，需要更多的 Redis 服务器时，你需要做的就是仅仅将 Redis 实例从一台服务迁移到另外一台服务器而已（而不用考虑重新分区的问题）。一旦你添加了另一台服务器，你需要将你一半的 Redis 实例从第一台机器迁移到第二台机器。

### **47、什么是 RedLock**

Redis 官方站提出了一种权威的基于 Redis 实现分布式锁的方式名叫 Redlock，此种方式比原先的单节点的方法更安全。它可以保证以下特性：

- **安全特性：**互斥访问，即永远只有一个 client 能拿到锁
- **避免死锁**：最终 client 都可能拿到锁，不会出现死锁的情况，即使原本锁住某资源的 client crash 了或者出现了网络分区
- **容错性**：只要大部分 Redis 节点存活就可以正常提供服务



![img](https://static001.geekbang.org/infoq/34/34266615fd4bf0f0ab953ba3c4dbf9d8.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



## **缓存异常**

### **48、缓存雪崩**

缓存雪崩是指缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。

**解决方案：**

- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
- 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
- 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

### **49、缓存穿透**

缓存穿透是指缓存和数据库中都没有的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉。

**解决方案：**

- 接口层增加校验，如用户鉴权校验，id 做基础校验，id<=0 的直接拦截；
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将 key-value 对写为 key-null，缓存有效时间可以设置短点，如 30 秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个 id 暴力攻击
- 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

**附加：**

对于空间的利用到达了一种极致，那就是 Bitmap 和布隆过滤器(Bloom Filter)。

**Bitmap：**典型的就是哈希表

缺点是，Bitmap 对于每个元素只能记录 1bit 信息，如果还想完成额外的功能，恐怕只能靠牺牲更多的空间、时间来完成了。

**布隆过滤器（推荐）**

就是引入了 k(k>1)k(k>1)个相互独立的哈希函数，保证在给定的空间、误判率下，完成元素判重的过程。

它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

Bloom-Filter 算法的核心思想就是利用多个不同的 Hash 函数来解决“冲突”。

Hash 存在一个冲突（碰撞）的问题，用同一个 Hash 得到的两个 URL 的值有可能相同。为了减少冲突，我们可以多引入几个 Hash，如果通过其中的一个 Hash 值我们得出某元素不在集合中，那么该元素肯定不在集合中。只有在所有的 Hash 函数告诉我们该元素在集合中时，才能确定该元素存在于集合中。这便是 Bloom-Filter 的基本思想。

Bloom-Filter 一般用于在大数据量的集合中判定某元素是否存在。

**50、缓存击穿**

**缓存击穿**是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。和缓存雪崩不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

**解决方案**

- 设置热点数据永远不过期。
- 加互斥锁，互斥锁

**51、缓存预热**

缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！

**解决方案：**

- 直接写个缓存刷新页面，上线时手工操作一下；
- 数据量不大，可以在项目启动的时候自动进行加载；
- 定时刷新缓存；

**52、缓存降级**

当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。、

**缓存降级**的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

- 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；
- **警告**：有些服务在一段时间内成功率有波动（如在 95~100%之间），可以自动降级或人工降级，并发送告警；
- **错误**：比如可用率低于 90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；
- **严重错误：**比如因为特殊原因数据错误了，此时需要紧急人工降级。

服务降级的目的，是为了防止 Redis 服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis 出现问题，不去数据库查询，而是直接返回默认值给用户。



**53、热点数据和冷数据**

热点数据，缓存才有价值。

对于冷数据而言，大部分数据可能还没有再次访问到就已经被挤出内存，不仅占用内存，而且价值不大。频繁修改的数据，看情况考虑使用缓存

对于热点数据，比如我们的某 IM 产品，生日祝福模块，当天的寿星列表，缓存以后可能读取数十万次。再举个例子，某导航产品，我们将导航信息，缓存以后可能读取数百万次。

数据更新前至少读取两次，缓存才有意义。这个是最基本的策略，如果缓存还没有起作用就失效了，那就没有太大价值了。

那存不存在，修改频率很高，但是又不得不考虑缓存的场景呢？有！比如，这个读取接口对数据库的压力很大，但是又是热点数据，这个时候就需要考虑通过缓存手段，减少数据库的压力，比如我们的某助手产品的，点赞数，收藏数，分享数等是非常典型的热点数据，但是又不断变化，此时就需要将数据同步保存到 Redis 缓存，减少数据库压力。

**54、缓存热点 key**

缓存中的一个 Key(比如一个促销商品)，在某个时间点过期的时候，恰好在这个时间点对这个 Key 有大量的并发请求过来，这些请求发现缓存过期一般都会从后端 DB 加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端 DB 压垮。

**解决方案：**

对缓存查询加锁，如果 KEY 不存在，就加锁，然后查 DB 入缓存，然后解锁；其他进程如果发现有锁就等待，然后等解锁后返回数据或者进入 DB 查询

![img](https://static001.geekbang.org/infoq/8a/8aedeacd1e457bde5e863367859ee389.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



**常用工具**



**55、Redis 支持的 Java 客户端都有哪些？官方推荐用哪个？**

Redisson、Jedis、lettuce 等等，官方推荐使用 Redisson。

**56、Redis 和 Redisson 有什么关系？**

Redisson 是一个高级的分布式协调 Redis 客服端，能帮助用户在分布式环境中轻松实现一些 Java 的对象 (Bloom filter, BitSet, Set, SetMultimap, ScoredSortedSet, SortedSet, Map, ConcurrentMap, List, ListMultimap, Queue, BlockingQueue, Deque, BlockingDeque, Semaphore, Lock, ReadWriteLock, AtomicLong, CountDownLatch, Publish / Subscribe, HyperLogLog)。

**57、Jedis 与 Redisson 对比有什么优缺点？**

Jedis 是 Redis 的 Java 实现的客户端，其 API 提供了比较全面的 Redis 命令的支持；Redisson 实现了分布式和可扩展的 Java 数据结构，和 Jedis 相比，功能较为简单，不支持字符串操作，不支持排序、事务、管道、分区等 Redis 特性。Redisson 的宗旨是促进使用者对 Redis 的关注分离，从而让使用者能够将精力更集中地放在处理业务逻辑上。

![img](https://static001.geekbang.org/infoq/a6/a691c0a6a1c5beec92ab8c0b2065718c.webp?x-oss-process=image%2Fresize%2Cp_80%2Fformat%2Cpng)



**其他问题**

**58、Redis 与 Memcached 的区别**

两者都是非关系型内存键值数据库，现在公司一般都是用 Redis 来实现缓存，而且 Redis 自身也越来越强大了！Redis 与 Memcached 主要有以下不同：

![img](https://static001.geekbang.org/infoq/7e/7ed4f38363df4d23cee6066f3060e9be.jpeg?x-oss-process=image%2Fresize%2Cp_80%2Fauto-orient%2C1)



(1) memcached 所有的值均是简单的字符串，redis 作为其替代者，支持更为丰富的数据类型

(2) redis 的速度比 memcached 快很多

(3) redis 可以持久化其数据

**59、如何保证缓存与数据库双写时的数据一致性？**

你只要用缓存，就可能会涉及到缓存与数据库双存储双写，你只要是双写，就一定会有数据一致性的问题，那么你如何解决一致性问题？

一般来说，就是如果你的系统不是严格要求缓存+数据库必须一致性的话，缓存可以稍微的跟数据库偶尔有不一致的情况，最好不要做这个方案，读请求和写请求串行化，串到一个内存队列里去，这样就可以保证一定不会出现不一致的情况

串行化之后，就会导致系统的吞吐量会大幅度的降低，用比正常情况下多几倍的机器去支撑线上的一个请求。

还有一种方式就是可能会暂时产生不一致的情况，但是发生的几率特别小，就是先更新数据库，然后再删除缓存。

![img](https://static001.geekbang.org/infoq/43/431b10ea8c53f47d06a6c8d7b7015729.png)



**60、Redis 常见性能问题和解决方案？**

Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化。

如果数据比较关键，某个 Slave 开启 AOF 备份数据，策略为每秒同步一次。

为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。

尽量避免在压力较大的主库上增加从库

Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的 CPU 和内存资源，导致服务 load 过高，出现短暂服务暂停现象。

为了 Master 的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关系为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现 Slave 对 Master 的替换，也即，如果 Master 挂了，可以立马启用 Slave1 做 Master，其他不变。

**61、Redis 官方为什么不提供 Windows 版本？**

因为目前 Linux 版本已经相当稳定，而且用户量很大，无需开发 windows 版本，反而会带来兼容性等问题。

**62、一个字符串类型的值能存储最大容量是多少？**

512M

**63、Redis 如何做大量数据插入？**

Redis2.6 开始 redis-cli 支持一种新的被称之为 pipe mode 的新模式用于执行大量数据插入工作。

**64、假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如果将它们全部找出来？**

使用 keys 指令可以扫出指定模式的 key 列表。

对方接着追问：如果这个 redis 正在给线上的业务提供服务，那使用 keys 指令会有什么问题？

这个时候你要回答 redis 关键的一个特性：redis 的单线程的。keys 指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用 scan 指令，scan 指令可以无阻塞的提取出指定模式的 key 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 keys 指令长。

**65、使用 Redis 做过异步队列吗，是如何实现的？**

使用 list 类型保存数据信息，rpush 生产消息，lpop 消费消息，当 lpop 没有消息时，可以 sleep 一段时间，然后再检查有没有信息，如果不想 sleep 的话，可以使用 blpop, 在没有信息的时候，会一直阻塞，直到信息的到来。redis 可以通过 pub/sub 主题订阅模式实现一个生产者，多个消费者，当然也存在一定的缺点，当消费者下线时，生产的消息会丢失。

**66、Redis 如何实现延时队列？**

使用 sortedset，使用时间戳做 score, 消息内容作为 key,调用 zadd 来生产消息，消费者使用 zrangbyscore 获取 n 秒之前的数据做轮询处理。

**67、Redis 回收进程如何工作的？**

- 一个客户端运行了新的命令，添加了新的数据。
- Redis 检查内存使用情况，如果大于 maxmemory 的限制， 则根据设定好的策略进行回收。
- 一个新的命令被执行，等等。
- 所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。

如果一个命令的结果导致大量内存被使用（例如很大的集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

**68、Redis 回收使用的是什么算法？**

LRU 算法。

**资料已整理成文档，需要获取的小伙伴可以+ VX:  mxk6072**



# Redis 3

## 如何保障MySQL和Redis的数据一致性？

[沉默王二](https://tobebetterjavaer.com/about-the-author/)2022年5月21日**MySQL****MySQL**约 2029 字大约 7 分钟

------

此页内容

- [不好的方案](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#不好的方案)

- - [1. 先写 MySQL，再写 Redis](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_1-先写-mysql-再写-redis)
  - [2. 先写 Redis，再写 MySQL](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_2-先写-redis-再写-mysql)
  - [3. 先删除 Redis，再写 MySQL](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_3-先删除-redis-再写-mysql)

- [好的方案](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#好的方案)

- - [4. 先删除 Redis，再写 MySQL，再删除 Redis](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_4-先删除-redis-再写-mysql-再删除-redis)
  - [5. 先写 MySQL，再删除 Redis](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_5-先写-mysql-再删除-redis)
  - [6. 先写 MySQL，通过 Binlog，异步更新 Redis](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_6-先写-mysql-通过-binlog-异步更新-redis)

- [几种方案比较](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#几种方案比较)

> 整理：沉默王二，戳[转载链接open in new window](https://mp.weixin.qq.com/s/RL4Bt_UkNcnsBGL_9w37Zg)，作者：楼仔，戳[原文链接open in new window](https://mp.weixin.qq.com/s/l7v4s1VekIPNi7KZuUgwGQ)。

如何保障 MySQL 和 Redis 的数据一致性？这个问题很早之前我就遇到过，但是一直没有仔细去研究，上个月看了极客的课程，有一篇文章专门有过讲解，刚好有粉丝也问我这个问题，所以感觉有必要单独出一篇。

**之前也看了很多相关的文章，但是感觉讲的都不好**，很多文章都会去讲各种策略，比如（旁路缓存）策略、（读穿 / 写穿）策略和（写回）策略等，感觉意义真的不大，然后有的文章也只讲了部分情况，也没有告诉最优解。

我直接先抛一下结论：**在满足实时性的条件下，不存在两者完全保存一致的方案，只有最终一致性方案。** 根据网上的众多解决方案，总结出 6 种，直接看目录：

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-537a505f-1f3f-4f23-b5e3-209c8c8a9281.png)

### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#不好的方案)不好的方案

[#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_1-先写-mysql-再写-redis)1. 先写 MySQL，再写 Redis

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-9ac6e9ab-dd82-40a5-b71b-836c745ed8ac.png)

> 图解说明：
>
> - 这是一副时序图，描述请求的先后调用顺序；
> - 橘黄色的线是请求 A，黑色的线是请求 B；
> - 橘黄色的文字，是 MySQL 和 Redis 最终不一致的数据；
> - 数据是从 10 更新为 11；
> - 后面所有的图，都是这个含义，不再赘述。

请求 A、B 都是先写 MySQL，然后再写 Redis，在高并发情况下，如果请求 A 在写 Redis 时卡了一会，请求 B 已经依次完成数据的更新，就会出现图中的问题。

这个图已经画的很清晰了，我就不用再去啰嗦了吧，**不过这里有个前提，就是对于读请求，先去读 Redis，如果没有，再去读 DB，但是读请求不会再回写 Redis。** 大白话说一下，就是读请求不会更新 Redis。

#### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_2-先写-redis-再写-mysql)2. 先写 Redis，再写 MySQL

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-c50e7ef0-40aa-4931-982a-a9aa31faa6f1.png)

同“先写 MySQL，再写 Redis”，看图可秒懂。

#### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_3-先删除-redis-再写-mysql)3. 先删除 Redis，再写 MySQL

这幅图和上面有些不一样，前面的请求 A 和 B 都是更新请求，这里的请求 A 是更新请求，**但是请求 B 是读请求，且请求 B 的读请求会回写 Redis。**

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-0fec5605-5530-4b12-af0e-2b529c41e6e6.png)

请求 A 先删除缓存，可能因为卡顿，数据一直没有更新到 MySQL，导致两者数据不一致。

**这种情况出现的概率比较大，因为请求 A 更新 MySQL 可能耗时会比较长，而请求 B 的前两步都是查询，会非常快。**

### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#好的方案)好的方案

#### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_4-先删除-redis-再写-mysql-再删除-redis)4. 先删除 Redis，再写 MySQL，再删除 Redis

对于“先删除 Redis，再写 MySQL”，如果要解决最后的不一致问题，其实再对 Redis 重新删除即可，**这个也是大家常说的“缓存双删”。**

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-1fe439cd-83fe-487f-a7ba-578a84839616.png)

为了便于大家看图，对于蓝色的文字，“删除缓存 10”必须在“回写缓存10”后面，那如何才能保证一定是在后面呢？**网上给出的第一个方案是，让请求 A 的最后一次删除，等待 500ms。**

对于这种方案，看看就行，反正我是不会用，太 Low 了，风险也不可控。

**那有没有更好的方案呢，我建议异步串行化删除，即删除请求入队列**

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-6cb3caf1-c85b-4361-8e29-9e71361fd0c8.png)

异步删除对线上业务无影响，串行化处理保障并发情况下正确删除。

如果双删失败怎么办，网上有给 Redis 加一个缓存过期时间的方案，这个不敢苟同。**个人建议整个重试机制，可以借助消息队列的重试机制，也可以自己整个表，记录重试次数**，方法很多。

> 简单小结一下：
>
> - “缓存双删”不要用无脑的 sleep 500 ms；
> - 通过消息队列的异步&串行，实现最后一次缓存删除；
> - 缓存删除失败，增加重试机制。

#### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_5-先写-mysql-再删除-redis)5. 先写 MySQL，再删除 Redis

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-1f0e26a8-49c3-469e-a193-08f9766943aa.png)

对于上面这种情况，对于第一次查询，请求 B 查询的数据是 10，但是 MySQL 的数据是 11，**只存在这一次不一致的情况，对于不是强一致性要求的业务，可以容忍。**（那什么情况下不能容忍呢，比如秒杀业务、库存服务等。）

当请求 B 进行第二次查询时，因为没有命中 Redis，会重新查一次 DB，然后再回写到 Reids。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-268696d3-a7e9-4762-9fe6-283859d5b0ba.png)

这里需要满足 2 个条件：

- 缓存刚好自动失效；
- 请求 B 从数据库查出 10，回写缓存的耗时，比请求 A 写数据库，并且删除缓存的还长。

对于第二个条件，我们都知道更新 DB 肯定比查询耗时要长，所以出现这个情况的概率很小，同时满足上述条件的情况更小。

#### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#_6-先写-mysql-通过-binlog-异步更新-redis)6. 先写 MySQL，通过 Binlog，异步更新 Redis

这种方案，主要是监听 MySQL 的 Binlog，然后通过异步的方式，将数据更新到 Redis，这种方案有个前提，查询的请求，不会回写 Redis。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/mysql/redis-shuju-yizhixing-0da55874-8cf7-4c5a-995b-a0e6611bfac2.png)

这个方案，会保证 MySQL 和 Redis 的最终一致性，但是如果中途请求 B 需要查询数据，如果缓存无数据，就直接查 DB；如果缓存有数据，查询的数据也会存在不一致的情况。

**所以这个方案，是实现最终一致性的终极解决方案，但是不能保证实时性。**

### [#](https://tobebetterjavaer.com/mysql/redis-shuju-yizhixing.html#几种方案比较)几种方案比较

我们对比上面讨论的 6 种方案：

1. 先写 Redis，再写 MySQL

- **这种方案，我肯定不会用**，万一 DB 挂了，你把数据写到缓存，DB 无数据，这个是灾难性的；
- 我之前也见同学这么用过，如果写 DB 失败，对 Redis 进行逆操作，那如果逆操作失败呢，是不是还要搞个重试？

1. 先写 MySQL，再写 Redis

- **对于并发量、一致性要求不高的项目，很多就是这么用的**，我之前也经常这么搞，但是不建议这么做；
- 当 Redis 瞬间不可用的情况，需要报警出来，然后线下处理。

1. 先删除 Redis，再写 MySQL

- 这种方式，我还真没用过，**直接忽略吧。**

1. 先删除 Redis，再写 MySQL，再删除 Redis

- 这种方式虽然可行，但是**感觉好复杂**，还要搞个消息队列去异步删除 Redis。

1. 先写 MySQL，再删除 Redis

- **比较推荐这种方式**，删除 Redis 如果失败，可以再多重试几次，否则报警出来；
- 这个方案，是实时性中最好的方案，在一些高并发场景中，推荐这种。

1. 先写 MySQL，通过 Binlog，异步更新 Redis

- **对于异地容灾、数据汇总等，建议会用这种方式**，比如 binlog + kafka，数据的一致性也可以达到秒级；
- 纯粹的高并发场景，不建议用这种方案，比如抢购、秒杀等。

**个人结论：**

- **实时一致性方案**：采用“先写 MySQL，再删除 Redis”的策略，这种情况虽然也会存在两者不一致，但是需要满足的条件有点苛刻，**所以是满足实时性条件下，能尽量满足一致性的最优解。**
- **最终一致性方案**：采用“先写 MySQL，通过 Binlog，异步更新 Redis”，可以通过 Binlog，结合消息队列异步更新 Redis，**是最终一致性的最优解。**

------

> 整理：沉默王二，戳[转载链接open in new window](https://mp.weixin.qq.com/s/RL4Bt_UkNcnsBGL_9w37Zg)，作者：楼仔，戳[原文链接open in new window](https://mp.weixin.qq.com/s/l7v4s1VekIPNi7KZuUgwGQ)。