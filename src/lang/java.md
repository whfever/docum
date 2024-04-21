# Idea

Power  Couple 

1.    think  then code
2. ​    see  three  solution
3. ​    30 minutes medium

## Setting

### Compile

> 更改编译内存

![image-20230724114913224](https://article.biliimg.com/bfs/article/c504f4f282bbb2df9d37f5df0c977aac4d2e01a2.png)



### Debugger

> stepping

![image-20230724115434856](https://article.biliimg.com/bfs/article/7702ed3e3bb384059378c311acf6307a48638409.png)





### Templete



> psvm   自动填充

# Java多线程并发编程

## 一、线程的状态

从操作系统层面划分

![thread-status](https://mynamelancelot.github.io/img/concurrent/thread-status.png)

------

从java代码的角度来进行划分

![thread-status-java](https://mynamelancelot.github.io/img/concurrent/thread-status-java.png)

## 二、JDK创建线程的方式

- 继承Thread类

```
public class ThreadTest {
  public static void main(String[] args) {
    Thread thread = new ThreadDemo();
    thread.start();
  }
}

class ThreadDemo extends Thread {
  @Override
  public void run() {
    System.out.println("do something");
  }
}
```

- 实现Runnable接口

```
public class ThreadTest {
  public static void main(String[] args) {
    Thread thread = new Thread(new RunnableDemo());
    thread.start();
  }
}

class RunnableDemo implements Runnable {
  @Override
  public void run() {
    System.out.println("do something");
  }
}
```

- 实现Callable接口携带返回值

```
public class ThreadTest {
  public static void main(String[] args) throws Exception {
    FutureTask<String> futureTask = new FutureTask<>(new CallableDemo());
    Thread thread = new Thread(futureTask);
    thread.start();     //此时Callable接口已经被调用
    String result = futureTask.get();
    System.out.println(result);
  }
}

class CallableDemo implements Callable<String> {
  @Override
  public String call() throws Exception {
    return "do something";
  }
}
```

- 定时器

```
public class ThreadTest {
  public static void main(String[] args) {
    Timer timer = new Timer();
    timer.schedule(new TimerTaskDemo(),1000);
  }
}

class TimerTaskDemo extends TimerTask {
  @Override
  public void run() {
    System.out.println("do something");
  }
}
```

- 线程池

```
public class ThreadTest {
  public static void main(String[] args) {
    ExecutorService executor = Executors.newFixedThreadPool(5);
    executor.execute(new ExecutorDemo());
    executor.shutdown();	//线程池销毁，正在执行和在队列等待的线程不会销毁
  }
}

class ExecutorDemo implements Runnable {
  @Override
  public void run() {
    System.out.println("do something");
  }
}
```

## 三、线程带来的风险

### 线程安全性问题

**出现线程安全性问题的条件**

1. 在多线程的环境下
2. 必须有共享资源
3. 对共享资源进行非原子性操作

------

**解决线程安全性问题的方法**

- 针对多个线程操作同一共享资源——不共享资源（ThreadLocal、不共享、操作无状态化、不可变）
- 针对多个线程进行非原子性操作——将非原子性操作改成原子性操作（使用加锁机制来保证可见性和有序性以及原子性、使用JDK自带的原子性操作的类、JUC提供的相应的并发工具类）

### 活跃性问题

**死锁问题**

指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象（至少两个资源），若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

> 比喻解释：有两根筷子，线程A和线程B都抢到了一个，互不相让
>
> ![dead-lock](https://mynamelancelot.github.io/img/concurrent/dead-lock.png)

```
public class DeadLockTest {

  public static final Object resource1 = new Object();
  public static final Object resource2 = new Object();

  public static void main(String[] args) {

    new Thread(() -> {
      synchronized (resource1) {
        try {
          Thread.sleep(500);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "：等待");
        synchronized (resource2) {
          System.out.println(Thread.currentThread().getName() + "：可以执行");
        }
      }
    }, "A").start();

    new Thread(() -> {
      synchronized (resource2) {
        try {
          Thread.sleep(500);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "：等待");
        synchronized (resource1) {
          System.out.println(Thread.currentThread().getName() + "：可以执行");
        }
      }
    }, "B").start();
  }

}
```

**活锁问题**

活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束

> 比喻解释：线程A和B再同一个家庭（总共有10w），一个每             月攒1w（攒够20w不攒了，出去玩），一个每月花1w（花光去打工）
>
> ![live-lock](https://mynamelancelot.github.io/img/concurrent/live-lock.png)

```
public class LiveLock {
  static volatile int count = 10;

  // 无法解锁，一个想加到20，一个想减到0
  public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
      // 期望减到 0 退出循环
      while (count > 0) {
        try {
          TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
          throw new RuntimeException(e);
        }

        count--;
        System.out.println("t1 count: " + count);
      }
    }, "t1");
    Thread t2 = new Thread(() -> {
      // 期望超过 20 退出循环
      while (count < 20) {
        try {
          TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
          throw new RuntimeException(e);
        }

        count++;
        System.out.println("t2 count: " + count);
      }
    }, "t2");

    t1.start();
    t2.start();

    try {
      t1.join();
      t2.join();
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }

    System.out.println("main end");
  }
}
```

**饥饿问题**

是指如果线程A占用了资源R，线程B又请求封锁R，于是B等待。线程C也请求资源R，当线程A释放了R上的封锁后，系统首先批准了线程C的请求，线程B仍然等待。然后线程D又请求封锁R，当线程C释放了R上的封锁之后，系统又批准了线程D的请求……，线程B可能永远等待。

> 比喻解释：餐厅有两个人（厨师兼服务员），来了两波客人，客人都要求贴身服务。这样就没人做饭了，于是客人永远也吃不到饭了
>
> ![hunger](https://mynamelancelot.github.io/img/concurrent/hunger.png)

```
public class ThreadStarvingTest {

  public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(2);

    executorService.submit(() -> {
      System.out.println(currentThreadName() + "接待客人");
      Future<String> cook = executorService.submit(() -> {
        System.out.println(currentThreadName() + "做回锅肉");
        return "回锅肉";
      });

      try {
        System.out.println(currentThreadName() + "上" + cook.get()); 
      } catch (InterruptedException e) {
        e.printStackTrace();
      } catch (ExecutionException e) {
        e.printStackTrace();
      }
    });

    executorService.submit(() -> {
      System.out.println(currentThreadName() + "接待客人");
      Future<String> cook = executorService.submit(() -> {
        System.out.println(currentThreadName() + "做鱼香肉丝");
        return "鱼香肉丝";
      });

      try {
        System.out.println(currentThreadName() + "上" + cook.get());
      } catch (InterruptedException e) {
        e.printStackTrace();
      } catch (ExecutionException e) {
        e.printStackTrace();
      }
    });

  }

  public static String currentThreadName() {
    return Thread.currentThread().getName();
  }

}
```

### 性能问题

1. 线程的生命周期开销非常高。在线程切换时存在CPU上下文切换开销，内存同步也存在着开销。
2. 消耗过多的CPU资源。如果可运行的线程数量多于可用处理器的数量，那么有线程将会被闲置。大量空闲的线程会占用许多内存，给垃圾回收器带来压力，而且大量的线程在竞争CPU资源时还将产生其他性能的开销。
3. 降低稳定性

## 四、synchronized的原理和使用

synchronized是一个：`非公平`、`悲观`、`独享`、`互斥`、`可重入`锁。自JDK6优化以后，基于JVM可以根据竞争激烈程度，从`偏向锁`–»`轻量级锁`–»`重量级锁`升级。

### synchronized的用法

- 修饰方法

  - 修饰普通方法，相当于锁当前对象，调用者，即指 this对象

  ```
  public synchronized void method() {
    System.out.println("do something");
  }
  ```

  - 修饰静态方法，相当于锁当前类对象，也指 `className.class`

  ```
  public static synchronized void method() {
    System.out.println("do something");
  }
  ```

- 修饰代码块【可以缩小锁的范围，提升性能】

  - 锁普通对象和锁this

  ```
  public void method() {
    synchronized(this) {
      System.out.println("do something");
    }
  }
  ```

  - 锁定类对象`className.class`

  ```
  public void method() {
    synchronized(SynchronizedTest.class) {
      System.out.println("do something");
    }
  }
  ```

### synchronized3种级别锁原理

使用了对象的[MarkWord](https://mynamelancelot.github.io/java/JVM.html#对象的结构)

1. MarkWord总共有四种状态:无锁状态、偏向锁、轻量级锁和重量级锁。
2. 随着锁的竞争：偏向锁–>轻量级锁–>重量级锁，只能升级
3. 不同锁状态的MarkWord结构不同

**MarkWord 数据一览**

![markworld](https://mynamelancelot.github.io/img/concurrent/markworld.png)

#### 偏向锁的原理

**偏向锁状态的消息头结构**

- 消息头锁标志位01
- 是否是偏向锁【0或1】
- 偏向线程ID

**一个对象创建时**

- 如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的，thread、epoch、age 都为 0
- 偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数`-XX:BiasedLockingStartupDelay=0`来禁用延迟
- 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，第一次用到 hashcode 时才会赋值（那么使用hash方法会使偏向锁失效）

**偏向锁衍化过程**

- 如果3个条件都满足【锁标志位–01】，【是否是偏向锁–1】，【偏向线程ID–当前线程】则直接获取到锁，不需要任何CAS操作，而且执行完同步代码块后，并不会修改这3个条件
- 如果一个新的线程尝试获取锁发现【锁标志位–01】，【是否是偏向锁 –1】，【偏向线程ID–不是自己】，则会触发一个check判断偏向锁指向的线程ID锁是否已经使用完了
  - 如果使用完了，则不存在竞争，CAS把偏向线程ID指向自己，这个对象锁就归自己所有了
  - 如果还在使用，竞争成立，则挂起当前线程，并达到原偏向线程释放锁之后将对象锁升级为轻量级锁

**偏向锁的好处**

- 老线程重复使用锁，无需任何CAS操作
- 新线程获取偏向锁，但是没有竞争，只需要在满足条件的时候CAS偏向线程ID即可
- 完美支持重入功能，而且没有任何CAS操作

#### 轻量级【自旋】锁

**轻量级锁状态的消息头结构**

①消息头锁标志位00

②指向栈中锁记录的指针

**轻量级锁衍化过程**

- 创建锁记录（Lock Record）对象，每个线程都的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word

![img](https://mynamelancelot.github.io/img/concurrent/create_lock_record.png)

- 让锁记录中 Object reference 指向锁对象，并尝试用 cas 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录

![create_lock_success](https://mynamelancelot.github.io/img/concurrent/create_lock_success.png)

> 如果 cas 失败，有两种情况
>
> - 其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程
> - 自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数，如果退出则计数减一

![img](https://mynamelancelot.github.io/img/concurrent/add_lock_record.png)

**轻量级锁衍化过程细节描述**

- 检查消息头的状态：是否处于无锁状态（偏向锁状态结束，等安全点后释放锁了，就是无锁状态），不是无锁状态挂起当前线程等待锁释放
- 所有争夺的线程都会拷贝一份消息头到各自的线程栈的`lock record`中，叫做`displace Mark Word`，并且记录自己的唯一线程标识符
- CAS把公有的消息头，变成指向自己线程标识符，这个时候消息头的数据结构发生改变，变成线程引用
- CAS成功的线程会执行同步操作，等到需要释放锁时把`displace Mark Word`写回到公有消息头里面，释放锁
- 重入的时候，无需要任何的操作，只需要在自己的`displace Mark Word`中标记一下
- CAS争夺锁失败的线程会发生自旋，自旋一定次数后还是失败的话，会修改消息头的状态为重量级锁，并且自身进入阻塞状态，等待拥有锁的线程执行结束。

**轻量级锁的优势**

 在获取锁的耗时不长的时候（比如锁的执行时间短、或者争抢的线程不多可以很快获得锁），通过一定次数的自旋，避免了重量级锁的线程阻塞和切换，提升了响应速度也兼顾了CPU的性能。

#### 重量级锁

![monitor](https://mynamelancelot.github.io/img/concurrent/monitor.png)

**重量级锁状态的消息头结构**

①消息头锁标志位10

②指向互斥量（重量级锁）的指针

**重量级锁衍化过程**

- 所有的竞争线程首先通过CAS拼接到`Contention List`这个队列里面
- 当锁的所有者释放的时候，会把一些线程推入到`EntryList`当中，然后`EntryList`开始CAS竞争，竞争成功就拿到锁，其它线程开始阻塞，等待下一次机会
- 当有锁线程调用obj.wait方法时，它会释放锁进入`WaitSet`，当调用`notify`方法，会从`waitSet`中随机选一个线程，`notifyAll`就是全部进行操作，让他们进入`EntryList`

**重量级锁特点**

- 处于ContentionList、EntryList、WaitSet中的线程都处于阻塞状态，该阻塞是由操作系统来完成的
- Synchronized是非公平锁。Synchronized在线程进入ContentionList时，等待的线程会先尝试自旋获取锁，如果获取不到就进入ContentionList，这明显对于已经进入队列的线程是不公平的。

## 五、早期线程通信机制

### wait\notify\notifyAll

`wait()`、`notify`/`notifyAll()`必须要在synchronized代码块执行，且一定是操作同一个对象【wait\notify\notifyAll会操作synchronized锁定对象】。由于 wait()、notify/notifyAll() 在synchronized 代码块执行，说明当前线程一定是获取了锁的。当线程执行wait()方法时候，会释放当前的锁，然后让出CPU，进入`等待状态`。只有当`notify/notifyAll()`被执行时候，才会随机唤醒一个或多个正处于等待状态的线程，然后继续往下执行，直到执行完synchronized 代码块的代码或是中途遇到wait() ，再次释放锁。

> notify/notifyAll() 的执行只是唤醒沉睡的线程，而不会立即释放锁。所以尽量在使用了notify/notifyAll() 后立即退出临界区，以唤醒其他线程让其获得锁

```
public class NotifyTest {

  public static void main(String[] args) throws InterruptedException {
    NotifyRunnable notifyRunnable = new NotifyRunnable();
    new Thread(notifyRunnable).start();
    TimeUnit.SECONDS.sleep(5);
    notifyRunnable.setFlag(false);
    synchronized (notifyRunnable) {
      notifyRunnable.notify();
    }
  }
}

class NotifyRunnable implements Runnable {
  private volatile boolean flag = true;

  public void setFlag(boolean flag) {
    this.flag = flag;
  }

  public synchronized void deal() {
    try {
      // synchronized的是当期对象，这里需要while循环，被唤醒之后是继续执行的
      while(flag) {
        wait();
      }
      System.out.println("唤醒了一个线程");
    } catch ( InterruptedException e) {
      e.printStackTrace();
    }
  }

  @Override
  public void run() {
    deal();
  }
}
```

当一个线程生命周期结束【线程run完成】，会发送这个线程对象的notifyAll()

### park & unpark

每个线程都有自己的一个 Parker 对象，由三部分组成 `_counter `，` _cond `和` _mutex `。打个比喻线程就像一个旅人，Parker 就像他随身携带的背包，条件变量就好比背包中的帐篷。`_counter `就好比背包中的备用干粮（0 为耗尽，1 为充足）

（1）调用 park 就是要看需不需要停下来歇息

- 如果备用干粮耗尽，那么钻进帐篷歇息
- 如果备用干粮充足，那么不需停留，继续前进

（2）调用 unpark，就好比令干粮充足

- 如果这时线程还在帐篷，就唤醒让他继续前进
- 如果这时线程还在运行，那么下次他调用 park 时，仅是消耗掉备用干粮，不需停留继续前进（仅会补充一份备用干粮）

```
public class Paker {

  public static void main(String[] args) {
    Thread t1  = new Thread(() -> {
      System.out.println("start...");

      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        throw new RuntimeException(e);
      }

      System.out.println("park...");
      LockSupport.park();
      System.out.println("resume...");
    });

    t1.start();
    try {
      TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }

    LockSupport.unpark(t1);
  }
}
```

**特点**

与 Object 的 wait & notify 相比

- wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用较为重量级
- park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll 是唤醒所有等待线程，就不那么【精确】
- park & unpark 可以先 unpark，而 wait & notify 不能先 notify

### interrupt方法

（1）打断 sleep，wait，join 的线程，会清空打断状态

```
public class InterruptTest {

  public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(()->{
      try {
        TimeUnit.SECONDS.sleep(3);
      } catch (InterruptedException e) {
        throw new RuntimeException(e);
      }
    }, "t1");
    t1.start();
    TimeUnit.SECONDS.sleep(1);
    t1.interrupt();
    // 打印false
    System.out.println(" 打断状态: " + t1.isInterrupted());
  }
}
```

（2）打断正常运行的线程，不会清空打断状态

```
public class InterruptTest {

  public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(()->{
      while(true) {
        Thread current = Thread.currentThread();
        boolean interrupted = current.isInterrupted();
        if(interrupted) {
          System.out.println(" 打断状态: " + interrupted);
          break;
        }
      }
    }, "t2");
    t1.start();
    TimeUnit.SECONDS.sleep(1);
    t1.interrupt();
  }
}
```

（3）打断 park 线程，不会清空打断状态

```
public class InterruptTest {

  public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
      System.out.println("park...");
      LockSupport.park();
      System.out.println("unpark...");
      // 打印 true
      System.out.println("打断状态： " + Thread.currentThread().isInterrupted());
    }, "t1");
    t1.start();
    TimeUnit.SECONDS.sleep(1);
    t1.interrupt();
  }

}
```

### 线程的join

`join`内部使用的是判断线程是否存活如果存活一直调用`wait()`。唤醒原理是当调用者的线程死亡时自动发送的`notifyAll()`，此时`wait()`被唤醒且线程已死亡。`join`这种机制并不常用。

```
public class JoinDemo {

  public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
      try {
        TimeUnit.SECONDS.sleep(3);
      } catch (InterruptedException e) {
        System.out.println(e);
      }
    });
    t1.start();

    try {
      t1.join();
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }

    // 3s 之后才会打印，即t1执行完成打印
    System.out.println("main end");
  }
}
```

`join`源码

```
public final void join() throws InterruptedException {
  join(0);
}

public final synchronized void join(long millis) throws InterruptedException {
  long base = System.currentTimeMillis();
  long now = 0;

  if (millis < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }

  if (millis == 0) {
    // 如果线程存活一直调用wait（即使被打断也会继续阻塞）
    while (isAlive()) {
      wait(0);
    }
  } else {
    // 有时间的等待，考虑了打断的影响
    while (isAlive()) {
      long delay = millis - now;
      if (delay <= 0) {
        break;
      }
      wait(delay);
      now = System.currentTimeMillis() - base;
    }
  }
}
```

> 如果threadObj.join()线程对象不是存活状态不会产生阻塞
>
> 如果threadObj.join()线程对象是存活状态直接调用notify()是不会放行的

## 六、volatile原理与使用

### volatile原子可见性

**Java内存模型规定在多线程情况下，线程操作主内存（类比内存条）变量，需要通过线程独有的工作内存（类比CPU高速缓存）拷贝主内存变量副本来进行。此处的所谓内存模型要区别于通常所说的虚拟机堆模型**

![work-memory](https://mynamelancelot.github.io/img/concurrent/work-memory.png)

> 如果是一个大对象，并不会从主内存完全拷贝一份，而是这个被访问对象引用的对象、对象中的字段可能存在拷贝

**线程独有的工作内存和进程内存（主内存）之间通过8中原子操作来实现，如下所示**

![main-work-swap](https://mynamelancelot.github.io/img/concurrent/main-work-swap.png)

`read load` 从主存复制变量到当前工作内存

`use assign` 执行代码，改变共享变量值，可以多次出现

`store write` 用工作内存数据刷新主存相关内容

这些操作并不是原子性，也就是在`read load`之后，如果主内存变量发生修改之后，线程工作内存中的值由于已经加载，不会产生对应的变化，所以计算出来的结果会和预期不一样，对于volatile修饰的变量，jvm虚拟机只是保证从主内存加载到线程工作内存的值是最新的。

### volatile的禁止指令重排序

**volatile止指令重排序的实现原理**

`volatile`变量的禁止指令重排序是基于内存屏障（Memory Barrier）实现**【synchronized不具有此功能】**。内存屏障又称内存栅栏，是一个CPU指令，内存屏障会导致JVM无法优化屏障内的指令集。

- 对`volatile`变量的写指令后会加入写屏障，对共享变量的改动，都同步到主存当中
- 对`volatile`变量的读指令前会加入读屏障，对共享变量的读取，加载的是主存中最新数据

> 如果单例模式中的懒汉式变量没有使用volatile仅仅使用synchronized双重检测加锁依旧会因为重排序问题产生线程安全性问题[参见](https://mynamelancelot.github.io/java/concurrent.html#lazy-questions)。

## 七、原子操作类

JDK提供了原子类型操作类，保证原子性，保证线程安全，这些类使用了CAS算法进行无锁运算避免阻塞的发生。

**使用原子的方式更新基本类型**

AtomicBoolean

AtomicInteger

AtomicLong

**使用原子的方式更新数组类型**

AtomicIntegerArray

AtomicLongArray

AtomicReferenceArray

**使用原子的方式更新引用类型**

AtomicReference

AtomicStampedReference【使用时间戳记录引用版本，解决[ABA问题](https://mynamelancelot.github.io/java/concurrent.html#aba)，但时间戳相同也会产生[ABA问题](https://mynamelancelot.github.io/java/concurrent.html#aba)】

AtomicMarkableReference【使用Boolean类型记录引用版本，适用在只需要知道对象是否有被修改的情景】

**使用原子的方式更新字段**

AtomicIntegerFieldUpdater

AtomicLongFieldUpdater

AtomicReferenceFieldUpdater

**高并发环境下更好性能的更新基本类型**

DoubleAdder

LongAdder

DoubleAccumulator

LongAccumulator

> 将基数进行拆分成为数组，这样共享资源变多，每次一个线程抢占更新一个数组的元素，最后进行运算，在线程数量不变的情况下，共享资源变多可增加并发效率。因为操作的是数组，所以以上四个类还解决了[伪共享](https://mynamelancelot.github.io/java/concurrent.html#false-sharing)问题
>
> LongAccumulator相比LongAdder 可以提供累加器初始非0值和指定累加规则【比如乘法】，后者只能默认为0且只能为相加

## 八、Lock&AQS&Condition

### Lock接口是JDK锁实现的标准

```
public interface Lock {

  // 获取锁。如果锁不可用，将禁用当前线程，并且在获得锁之前，该线程将一直处于休眠状态
  void lock();

  // 获取锁，等待过程中如果当前线程被中断，则抛出异常，此时加锁还未成功不需要释放资源
  void lockInterruptibly() throws InterruptedException;

  // 仅在调用时锁为空闲状态才获取该锁。如果锁可用立即返回值true。如果锁不可用，立即返回值 false
  boolean tryLock();

  // 如果锁在给定的等待时间内空闲，并且当前线程未被中断，则获取锁，如果等待期间被中断，则抛出异常
  boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

  // 释放锁。锁必须由当前线程持有。调用Condition.await()将在等待前以原子方式释放锁
  void unlock();

  // 返回绑定到此Lock实例的新Condition实例
  Condition newCondition();
}
```

**锁相关类**

- ReentrantLock 可重入锁
- ReentrantReadWriteLock 可重入读写锁，非公平状态下，可能会发生饥饿问题
- StampedLock JDK8对ReentrantReadWriteLock的升级[[详见\]](https://mynamelancelot.github.io/java/concurrent.html#StampedLock)

### AQS抽象类

`AbstractQueuedSynchronizer`是`ReadWriteLock`、`ReentrantLock`、`StampedLock`、`Semaphore`、`ReentrantReadWriteLock`、`SynchronousQueue`、`FutureTask`等同步类的内部工具帮助类。

AQS支持独占锁（exclusive）和共享锁（share）两种模式

- 独占锁：只能被一个线程获取到（Reentrantlock）
- 共享锁：可以被多个线程同时获取（CountDownLatch，ReadWriteLock）

无论是独占锁还是共享锁，本质上都是对AQS内部的一个变量state的获取。state是一个原子的int变量，用来表示锁状态、资源数等。

![QIKsHg.png](https://mynamelancelot.github.io/img/concurrent/QIKsHg.png)

**AQS内部实现了两个队列，一个同步队列，一个条件队列**

![QIMk8I.png](https://mynamelancelot.github.io/img/concurrent/QIMk8I.png)

同步队列的作用是：当线程获取资源失败之后，就进入同步队列的尾部保持自旋等待，不断判断自己是否是链表的头节点，如果是头节点，就不断参试获取资源，获取成功后则退出同步队列。 同步队列的作用是：为Lock实现的一个基础同步器，并且一个线程可能会有多个条件队列，只有在使用了Condition才会存在条件队列。

**AQS的基本属性和方法**

```
//头节点
private transient volatile Node head;

//尾节点
private transient volatile Node tail;

//共享变量，使用volatile修饰保证线程可见性，用于记录了加锁次数
private volatile int state;

// 队列头节点，头结点不储存数据
private transient volatile Node head;

// 队列尾节点
private transient volatile Node tail;
```

**同步队列和条件队列都是由一个个Node组成的**

```
static final class Node {
  /** 节点为共享模式下等待 */
  static final Node SHARED = new Node();

  /** 节点为独占模式下等待 */
  static final Node EXCLUSIVE = null;

  /** 当前节点由于超时或中断被取消 */
  static final int CANCELLED =  1;

  /** 表示当前节点的前节点被阻塞 */
  static final int SIGNAL    = -1;

  /** 当前节点在等待condition */
  static final int CONDITION = -2;

  /** 状态需要向后传播 */
  static final int PROPAGATE = -3;

  volatile int waitStatus;

  volatile Node prev;

  volatile Node next;

  volatile Thread thread;

  Node nextWaiter;
}
```

**重要方法的源码解析**

```
//独占模式下获取资源
public final void acquire(int arg) {
  if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

- `acquire(int arg)`首先调用`tryAcquire(arg)`尝试直接获取资源具体实现由子类负责。
- 如果直接获取到资源，直接return
- 如果没有直接获取到资源，将当前线程加入等待队列的尾部，并标记为独占模式，使线程在等待队列中自旋等待获取资源，直到获取资源成功才返回，如果线程在等待的过程中被中断过，就返回true，否则返回false
  - 返回false就不需要判断中断了，直接return
  - 返回true执行`selfInterrupt()`方法，而这个方法就是简单的中断当前线程`Thread.currentThread().interrupt();`其作用就是补上在自旋时没有响应的中断

可以看出在整个方法中，最重要的就是`acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`

首先看`Node addWaiter(Node mode)`，这个方法的作用就是添加一个等待者，添加等待者就是将该节点加入等待队列

```
private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode);
  Node pred = tail;
  //尝试快速入队
  if (pred != null) { //队列已经初始化
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node; //快速入队成功后，就直接返回了
    }
  }
  //快速入队失败，也就是说队列都还每初始化
  enq(node);
  return node;
}

//执行入队
private Node enq(final Node node) {
  for (;;) {
    Node t = tail;
    // 如果没有队尾即还没初始化
    if (t == null) { 
      //如果队列为空，用一个空节点充当队列头
      if (compareAndSetHead(new Node()))
        //尾部指针也指向队列头
        tail = head;
    } else {
      //队列已经初始化，入队
      node.prev = t;
      if (compareAndSetTail(t, node)) {
        t.next = node;
        //打断循环
        return t;
      }
    }
  }
}
```

然后看`acquireQueued(final Node node, int arg)`，等待出队

```
final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    boolean interrupted = false;
    for (;;) {
      //拿到node的上一个节点
      final Node p = node.predecessor();
      //前置节点为head，说明可以尝试获取资源。排队成功后，尝试拿锁
      if (p == head && tryAcquire(arg)) {
        //获取成功，更新head节点
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      //尝试拿锁失败后，根据条件进行park
      if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}
//获取资源失败后，检测并更新等待状态
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  int ws = pred.waitStatus;
  if (ws == Node.SIGNAL)
    return true;
  if (ws > 0) {
    do {
      //如果前节点取消了，那就往前找到一个等待状态的接替你，并排在它的后面
      node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0);
    pred.next = node;
  } else {
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
  }
  return false;
}
//阻塞当前线程，返回中断状态
private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this);
  return Thread.interrupted();
}
```

具体的`boolean tryAcquire(int acquires)`实现有所不同

```
// 公平锁的实现
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
  int c = getState();
  if (c == 0) {
    if (!hasQueuedPredecessors() &&
        compareAndSetState(0, acquires)) {
      setExclusiveOwnerThread(current);
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) {
    int nextc = c + acquires;
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false;
}

// 非公平锁的实现
final boolean nonfairTryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
  int c = getState();
  if (c == 0) {
    if (compareAndSetState(0, acquires)) {
      setExclusiveOwnerThread(current);
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) {
    int nextc = c + acquires;
    if (nextc < 0) // overflow
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false;
}
```

**AQS的设计思想**

（1）获取锁的逻辑

```
while(state 状态不允许获取) {
  if(队列中还没有此线程) {
    入队并阻塞
  }
}
当前线程出队
```

（2）释放锁的逻辑

```
if(state 状态允许了) {
  恢复阻塞的线程(s) 
}
```

### Condition

condition对象是依赖于lock对象的，即condition对象需要通过lock对象进行创建出来(调用Lock对象的newCondition()方法)，condition是用于替代`wait()/notify()/notifyAll()`的接口标准。**condition调用时必须和lock、unlock联用，如果没被独占会被抛出异常（因为存储在Node的nextWaiter上）**

```
public interface Condition {

  // 一直等待可能抛出中断异常
  void await() throws InterruptedException;

  // 一直等待不会被中断异常打扰
  void awaitUninterruptibly();

  // 等待指定纳秒，期间可能抛出中断异常
  long awaitNanos(long nanosTimeout) throws InterruptedException;

  // 等待指定时间，期间可能抛出中断异常
  boolean await(long time, TimeUnit unit) throws InterruptedException;

  // 等待到指定时间，期间可能抛出中断异常
  boolean awaitUntil(Date deadline) throws InterruptedException;

  // 发送信号，解除任意一个等待
  void signal();

  // 发送信号，解除所有等待
  void signalAll();
}
```

`ConditionObject`是`Condition`的实现类是一个`AQS`的内部类，`condition`可以实现各种队列【有界队列、阻塞队列等】，`ConditionObject`实现了一个FIFO的等待队列`await`方法在队列后面添加一个元素，`signal`释放一个元素【有序释放】。

## 九、线程工具类

### ThreadLocal

ThreadLoal变量，线程局部变量，同一个 ThreadLocal 所包含的对象，在不同的 Thread 中有不同的副本。

- 因为每个 Thread 内有自己的实例副本，且该副本只能由当前 Thread 使用

- 每个 Thread 有自己的实例副本，且其它 Thread 不可访问，那就不存在多线程间共享的问题

- ThreadLoal正真保存变量数据的是ThreadLoalMap

- ThreadLocal类的有个初始化方法是可以被重写的，用于赋初值

  ```
  protected T initialValue() {
    return null;
  }
  ```

**ThreadLocal使用示例**

```
public class ThreadLocalDemo implements Runnable {
  private static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
    @Override
    protected Integer initialValue() {
      return 0;
    }
  };


  @Override
  public void run() {
    threadLocal.set(threadLocal.get()+1);
  }
}
```

**ThreadLocal内存泄露**

![ThreadLocalMemoryLeak](https://mynamelancelot.github.io/img/concurrent/ThreadLocalMemoryLeak.png)

ThreadLocal本身并不存储值，它依赖于Thread类中的ThreadLocalMap，当调用set(T value)时，ThreadLocal将自身作为Key，值作为Value存储到Thread类中的ThreadLocalMap中（存储和获取其实都是操作的Thread对象），这就相当于所有线程读写的都是自身的一个私有副本，线程之间的数据是隔离的，互不影响，也就不存在线程安全问题了。

由于ThreadLocal对象是弱引用，如果外部没有强引用指向它，它就会被GC回收，导致Entry的Key为null，如果这时value外部也没有强引用指向它，那么value就永远也访问不到了，按理也应该被GC回收，但是由于Entry对象还在强引用value，导致value无法被回收，这时「内存泄漏」就发生了，value成了一个永远也无法被访问，但是又无法被回收的对象。

Entry对象属于ThreadLocalMap，ThreadLocalMap属于Thread，如果线程本身的生命周期很短，短时间内就会被销毁，那么「内存泄漏」立刻就会得到解决，只要线程被销毁，value也会随之被回收。问题是，线程本身是非常珍贵的计算机资源，很少会去频繁的创建和销毁，一般都是通过线程池来使用，这就将线程的生命周期大大拉长，「内存泄漏」的影响也会越来越大。

### CountDownLatch

 CountDownLatch是一个同步类工具，不涉及锁定，当count的值为零时当前线程继续运行。在不涉及同步，只涉及线程通信的时候，使用它较为合适。

```
// 计算多行数字合
public class CountDownLatchDemo {

  private  CountDownLatch countDownLatch;

  private int storeNum[];

  public CountDownLatchDemo(final int numsLines) {
    this.countDownLatch = new CountDownLatch(numsLines);
    this.storeNum = new int[numsLines];
  }

  //计算每行数字合
  private void calc(int index, int calcNums[]){
    storeNum[index] = 0;
    for (int i = 0; i < calcNums.length; i++) {
      storeNum[index] += calcNums[i];
    }
    countDownLatch.countDown();
  }

  //总计
  private int calcSum() throws InterruptedException {
    countDownLatch.await();
    int result = 0;

    for (int i = 0; i < storeNum.length; i++) {
      result+= storeNum[i];
    }
    return result;
  }

  public static void main(String[] args) throws InterruptedException {

    CountDownLatchDemo countDownLatchDemo = new CountDownLatchDemo(nums.length);
    for (int i = 0; i < nums.length; i++) {
      int finalI = i;
      new Thread(new Runnable() {
        @Override
        public void run() {
          countDownLatchDemo.calc(finalI, nums[finalI]);
        }
      }).start();
    }
    int sum = countDownLatchDemo.calcSum();
    System.out.println(sum);
  }
}
```

**CountDownLatch的原理**

- 构造函数其实是初始化AQS的state

```
public CountDownLatch(int count) {
  if (count < 0) throw new IllegalArgumentException("count < 0");
  this.sync = new Sync(count);
}

Sync(int count) {
  setState(count);
}
```

- await()是自旋等待计数变为0的过程

```
public void await() throws InterruptedException {
  sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
  if (Thread.interrupted())
    throw new InterruptedException();
  // tryAcquireShared(arg) ==》return (getState() == 0) ? 1 : -1;
  // 即当state不为0时进入，为0直接运行不阻塞
  if (tryAcquireShared(arg) < 0)
    doAcquireSharedInterruptibly(arg);
}

private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
  // 加入一个共享队列元素
  final Node node = addWaiter(Node.SHARED);
  boolean failed = true;
  try {
    for (;;) {
      final Node p = node.predecessor();
      if (p == head) {
        // tryAcquireShared(arg) ==》(getState() == 0) ? 1 : -1
        // 即如果是队列首元素就等待释放
        int r = tryAcquireShared(arg);
        if (r >= 0) {
          setHeadAndPropagate(node, r);
          p.next = null; // help GC
          failed = false;
          return;
        }
      }
      if (shouldParkAfterFailedAcquire(p, node) &&
          parkAndCheckInterrupt())
        throw new InterruptedException();
    }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}
}
```

### CyclicBarrier

 CyclicBarrier可以使一定数量的线程反复地在栅栏位置处汇集。当线程到达栅栏位置时将调用await方法，这个方法将阻塞直到指定数量的线程到达栅栏位置。如果指定数量的线程到达栅栏位置，那么栅栏将打开，然后栅栏将被重置以便下次使用。

```
public class CyclicBarrierDemo {

  public void meet(CyclicBarrier cyclicBarrier){
    String threadName = Thread.currentThread().getName();
    System.out.println(threadName + " 到场 ");
    try {
      Thread.sleep(1000);
      cyclicBarrier.await();
    } catch (InterruptedException e) {
      // 当前线程被中断
      e.printStackTrace();
    } catch (BrokenBarrierException e) {
      // 不是被中断的线程，但是其它相关线程await()时发生中断
      e.printStackTrace();
    }
    System.out.println(threadName + " 准备开会发言");
  }

  public static void main(String[] args) {
    CyclicBarrierDemo cyclicBarrierDemo = new CyclicBarrierDemo();
    CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
    for (int i = 0; i < 5; i++) {
      new Thread(new Runnable() {
        @Override
        public void run() {
          cyclicBarrierDemo.meet(cyclicBarrier);
        }
      }).start();
    }
  }
}
```

**CyclicBarrier的原理**

 CyclicBarrier内部使用了`ReentrantLock`和`Condition`完成栅栏操作

### Semaphore

 Semaphore控制了最多同时执行的线程个数，但不控制线程创建的个数，线程创建之后会阻塞不是不创建线程。Semaphore可以灵活控制释放和需要解锁资源的个数。

```
public class SemaphoreDemo {

  private Semaphore semaphore = new Semaphore(5);

  public void doSomething() {
    try {
      /**
        * 在 semaphore.acquire() 和 semaphore.release()之间的代码，同一时刻只允许制定个数的线程进入，
        * 因为semaphore的构造方法是5，则同一时刻只允许5个线程进入，其他线程只能等待。
        **/
      semaphore.acquire();
      System.out.println(Thread.currentThread().getName() + ":doSomething start-" + new Date());
      Thread.sleep(2000);
      System.out.println(Thread.currentThread().getName() + ":doSomething end-" + new Date());
      semaphore.release();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

  public static void main(String[] args) {
    SemaphoreDemo service = new SemaphoreDemo();
    for (int i = 0; i < 20; i++) {
      new Thread(new Runnable() {
        @Override
        public void run() {
          service.doSomething();
        }
      }).start();
    }
  }
}
```

**Semaphore的原理**

 Semaphore内部使用了AQS来完成等待队列和计数。

### Exchanger

 一个线程在完成一定的事务后想与另一个线程交换数据，则第一个先拿出数据的线程会一直等待第二个线程，直到第二个线程拿着数据到来时才能彼此交换对应数据。

```
// 每次计算完成交换结果
public class ExchangerTest {
  static class Producer extends Thread {
    private Exchanger<Integer> exchanger;
    private static int data = 0;
    Producer(Exchanger<Integer> exchanger) {
      super("Producer");
      this.exchanger = exchanger;
    }

    @Override
    public void run() {
      for (int i=1; i<5; i++) {
        try {
          TimeUnit.SECONDS.sleep(1);
          data = i;
          System.out.println(getName()+" 交换前:" + data);
          data = exchanger.exchange(data);
          System.out.println(getName()+" 交换后:" + data);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
    }
  }

  static class Consumer extends Thread {
    private Exchanger<Integer> exchanger;
    private static int data = 0;
    Consumer(Exchanger<Integer> exchanger) {
      super("Consumer");
      this.exchanger = exchanger;
    }

    @Override
    public void run() {
      while (true) {
        data = 0;
        System.out.println(getName()+" 交换前:" + data);
        try {
          TimeUnit.SECONDS.sleep(1);
          data = exchanger.exchange(data);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println(getName()+" 交换后:" + data);
      }
    }
  }

  public static void main(String[] args) throws InterruptedException {
    Exchanger<Integer> exchanger = new Exchanger<Integer>();
    new Producer(exchanger).start();
    new Consumer(exchanger).start();
  }
}
```

## 十、Future模式

 从JDK5开始提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。Future模式的核心思想是能够让执行线程在原来需要同步等待的这段时间用来做其他的事情。（因为可以异步获得执行结果，所以不用一直同步等待去获得执行结果）

```
public class FutureDemo {

  private static class Product implements Callable<Product> {
    @Override
    public Product call() throws Exception {
      System.out.println("生成产品中...");
      Thread.sleep(5000);
      System.out.println("生成产品结束...");
      return new Product();
    }
  }

  public static void main(String[] args) {
    FutureTask<Product> futureTask = new FutureTask<>(new Product());
    // 使用FutureTask创建一个thread，FutureTask实现了Runnable接口
    Thread thread = new Thread(futureTask);
    // 启动线程任务
    thread.start();
    // 此时FutureTask任务在其它线程中执行，主线程不受影响
    System.out.println("干点其他事...");
    try {
      // 此时需要FutureTask任务执行的结果，会在此一直等候
      Product product = futureTask.get();
    } catch (InterruptedException e) {
      System.out.println("发生中断");
    } catch (ExecutionException e) {
      System.out.println("执行过程中发生异常 " + e.getMessage());
    }
  }
}
```

FutureTask实现了Runnable接口，其实Callable接口会在FutureTask的run方法被调用时执行`call()`方法，在调用`get()`方法是等待`call()`方法执行完成，获取执行结果。

### CompleteFuture

Future获得异步执行结果 有两种方法：调用Get()或者轮询isDOne()是否为True，这两种方法都不是太好，因为主线程也会被迫等待。为了减少这种等待 JDK8引入了这个CompleteFuture对Future做了改进，可以传入回调对象

- 异步执行，无返回结果，但是返回了Future对象

  ```
  public static void main(String[] args) {
    Runnable runnable = () -> {
      System.out.println("执行无返回结果的异步任务-开始");
      try {
        TimeUnit.SECONDS.sleep(5);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("执行无返回结果的异步任务-结束");
    
    };
    CompletableFuture<Void> futureVoid = CompletableFuture.runAsync(runnable);
    futureVoid.join();
  }
  ```

- 异步执行阻塞获取结果

  ```
  public static void main(String[] args) {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
      System.out.println("执行有返回值的异步任务");
      try {
        TimeUnit.SECONDS.sleep(5);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      return "Hello World";
    });
    String result = future.get();
    System.out.println(result);
  }
  ```

- 异步执行，阻塞获取结果后对结果进行运算处理，不改变最终结果

  ```
  public static void main(String[] args) {        
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
      System.out.println("执行有返回值的异步任务");
      try {
        TimeUnit.SECONDS.sleep(5);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      return "Hello World";
    });
    
    String result2 = future.whenComplete(new BiConsumer<String, Throwable>() {
      @Override
      public void accept(String t, Throwable action) {
        t = t + 1;
        System.out.println("任务执行后结果处理");
      }
    }).exceptionally(new Function<Throwable, String>() {
      @Override
      public String apply(Throwable t) {
        System.out.println("任务执行后结果额外处理-如果有异常进入此处");
        return "异常结果";
      }
    }).get();
    
    String result = future.get();
    System.out.println(result);
    System.out.println("最终结果 " + result2);
  }
  ```

- thenCombine会将两个任务的执行结果作为所提供函数的参数，且该方法有返回值

  ```
  public static void main(String[] args) {        
    CompletableFuture<Integer> cf1 = CompletableFuture.supplyAsync(() -> {
      System.out.println(Thread.currentThread() + " cf1 do something....");
      return 1;
    });
    
    CompletableFuture<Integer> cf2 = CompletableFuture.supplyAsync(() -> {
      System.out.println(Thread.currentThread() + " cf2 do something....");
      return 2;
    });
    
    CompletableFuture<Integer> cf3 = cf1.thenCombine(cf2, (a, b) -> {
      System.out.println(Thread.currentThread() + " cf3 do something....");
      return a + b;
    });
      
    System.out.println("cf3结果->" + cf3.get());
  }
  ```

- thenAcceptBoth同样将两个任务的执行结果作为方法入参，但是无返回值

  ```
  public static void main(String[] args) {        
    CompletableFuture<Integer> cf4 = CompletableFuture.supplyAsync(() -> {
      System.out.println(Thread.currentThread() + " cf4 do something....");
      return 1;
    });
    
    CompletableFuture<Integer> cf5 = CompletableFuture.supplyAsync(() -> {
      System.out.println(Thread.currentThread() + " cf5 do something....");
      return 2;
    });
    
    CompletableFuture<Void> cf6 = cf4.thenAcceptBoth(cf5, (a, b) -> {
      System.out.println(Thread.currentThread() + " cf6 do something....");
      System.out.println("处理结果不返回："+(a + b));
    });
  }
  ```

- allOf多个任务都执行完成后才会执行，只有有一个任务执行异常，则返回的CompletableFuture执行get方法时会抛出异常，如果都是正常执行，则get返回null；anyOf多个任务只要有一个任务执行完成，则返回的CompletableFuture执行get方法时会抛出异常，如果都是正常执行，则get返回执行完成任务的结果

  ```
  public static void main(String[] args) {         
    CompletableFuture<String> cf7 = CompletableFuture.supplyAsync(() -> {
      try {
        System.out.println(Thread.currentThread() + " cf7 do something....");
        Thread.sleep(2000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("cf7 任务完成");
      return "cf7 任务完成";
    });
    
    CompletableFuture<String> cf8 = CompletableFuture.supplyAsync(() -> {
      try {
        System.out.println(Thread.currentThread() + " cf8 do something....");
        Thread.sleep(5000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("cf8 任务完成");
      return "cf8 任务完成";
    });
    
    CompletableFuture<String> cf9 = CompletableFuture.supplyAsync(() -> {
      try {
        System.out.println(Thread.currentThread() + " cf9 do something....");
        Thread.sleep(3000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("cf9 任务完成");
      return "cf9 任务完成";
    });
    
    CompletableFuture<Void> cfAll = CompletableFuture.allOf(cf7, cf8, cf9);
    System.out.println("cfAll结果->" + cfAll.get());
    
    CompletableFuture<Object> cfAny = CompletableFuture.anyOf(cf7, cf8, cf9);
    System.out.println("cfAny结果->" + cfAny.get());
  }
  ```

## 十一、Fork/Join框架

 Fork/Join 框架：就是在必要的情况下，将一个大任务，进行拆分(fork)成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行join 汇总。详细参见[Fork/Join框架](https://mynamelancelot.github.io/java/java8.html#forkjoin)

## 十二、并发容器

### CopyOnWrite容器

【`CopyOnWriteArrayList`、`CopyOnWriteArraySet`】

 CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器添加元素，再将原容器的引用指向新的容器。这样做的好处是可以并发读，不需要加锁。

 在进行同时进行add()操作时依旧会加锁，此时不影响任何读操作。

```
public boolean add(E e) {
  final ReentrantLock lock = this.lock;
  lock.lock();
  try {
    Object[] elements = getArray();
    int len = elements.length;
    Object[] newElements = Arrays.copyOf(elements, len + 1);
    newElements[len] = e;
    setArray(newElements);
    return true;
  } finally {
    lock.unlock();
  }
}
```

### 并发Map

【`ConcurrentHashMap`、`ConcurrentSkipListMap（支持排序）`】

 `ConcurrentHashMap`内部使用段（Segment）来表示这些不同的部分，每个段其实就是一个小的HashTable，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。把一个整体分成了16个段。也就是说最高支持16个线程的并发修改操作。而且大量使用volatile关键字，第一时间获取修改的内容。

### 非阻塞Queue

【`ConcurrentLinkedQueue`、`ConcurrentLinkedDeque`】

 是一个适用于高并发场景下的队列，通过CAS无锁的方式，实现了高并发状态下的高性能。它是一个基于链接节点的无界线程安全队列。该队列不允许null元素。

- add()和offer()都是加入元素，无区别
- poll()和peek()都是取头元素节点，前者会删除元素，后者不会

### 阻塞Queue

【`ArrayBlockingQueue`、`LinkedBlockingQueue`、`LinkedBlockingDeque`、`PriorityBlockingQueue`】

- `ArrayBlockingQueue`：基于数组的有界阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便从缓存队列中的数据对象，其实内部没实现读写分离，也就意味着生产者和消费者不能完全并行，长度需要定义，可以指定先进先出或者先进后出，在很多场合非常适合使用。
- `LinkedBlockingQueue`：基于链表的阻塞队列，同ArrayBlockingQueue类似，其内部也维持着一个数据缓存队列。LinkedBlockingQueue内部采用分离锁（读写分离两个锁），从而实现生产者和消费者操作的完全并行运行。他是一个无界队列（如果初始化指定长度则为有界队列）。
- `PriorityBlockingQueue`：基于优先级的阻塞队列（队列中的对象必须实现Comparable接口），内部控制线程同步的锁采用公平锁，也是无界队列。

### 特殊Queue

`DelayQueue`：带有延迟时间的Queue，其中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue中的元素必须实现Delayed接口，DelayQueue是一个没有大小限制的队列。DelayQueue支持阻塞和非阻塞两种模式。

`SynchronousQueue`：一种没有缓冲的队列，生存者生产的数据直接会被消费者获取并消费。每个put操作必须等待一个take，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。

## 十三、线程池

### 线程池的优势

- 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 使用原始API创建线程池

```
/**
 * corePoolSize:线程池的核心大小，当提交一个任务到线程池时，线程池会创建一个线程来执行任务
 *              即使其他空闲的基本线程能够执行新任务也会创建线程，如需要执行的任务数大于线
 *              程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads()方法
 *              线程池会提前创建并启动所有基本线程。
 * maximumPoolSize:线程池最大大小，线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数
 *                 小于最大线程数大于等于核心线程数，则线程池会再创建新的线程执行任务。如果使用
 *                 了无界的任务队列这个参数就没什么效果。
 * keepAliveTime:救急线程活动保持时间，线程池的工作线程空闲后，保持存活的时间。所以如果任务很多
 *               并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
 * TimeUnit:救急线程活动保持时间的单位，可选的单位有天(DAYS)，小时(HOURS)，分钟(MINUTES)，
 *          毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 
 *          千分之一微秒)
 * workQueue:任务对列，用于保存等待执行的任务的阻塞队列
 *           - ArrayBlockingQueue：基于数组结构的有界阻塞队列
 *           - LinkedBlockingQueue：基于链表的阻塞队列,如果没构造函数没传入队列大小则为无界队列
 *                                  Executors.newFixedThreadPool()使用了这个队列
 *           - SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用
 *                               移除操作，否则插入操作一直处于阻塞状态
 *                               Executors.newCachedThreadPool使用了这个队列
 *           - PriorityBlockingQueue：个具有优先级得无限阻塞队列
 * threadFactory:用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，
 *               Debug和定位问题时非常有帮助。
 * handler:当队列和线程池都满了，必须采取一种策略处理提交的新任务。
 *         - AbortPolicy：默认策略，无法处理新任务时抛出异常
 *         - CallerRunsPolicy：使用调用者所在线程来运行任务
 *         - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务
 *         - DiscardPolicy：不处理，丢弃掉
 */
public ThreadPoolExecutor(int corePoolSize, 
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler);
```

- 线程池源代码概览

```
public class ThreadPoolExecutor extends AbstractExecutorService {

  public void execute(Runnable command) {
    if (command == null)
      throw new NullPointerException();
    // 获取线程池控制状态
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) { // worker数量小于corePoolSize
      if (addWorker(command, true)) // 添加worker
        return;
      // 不成功则再次获取线程池控制状态
      c = ctl.get();
    }
    // 线程池处于RUNNING状态，将用户自定义的Runnable对象添加进workQueue队列
    if (isRunning(c) && workQueue.offer(command)) { 
      // 再次检查，获取线程池控制状态
      int recheck = ctl.get();
      // 线程池不处于RUNNING状态，将自定义任务从workQueue队列中移除
      if (! isRunning(recheck) && remove(command)) 
        // 拒绝执行命令
        reject(command);
      else if (workerCountOf(recheck) == 0) // worker数量等于0
        // 添加worker
        addWorker(null, false);
    }
    else if (!addWorker(command, false)) // 添加worker失败
      // 拒绝执行命令
      reject(command);
  }
}
private boolean addWorker(Runnable firstTask, boolean core) {
  //...
  if (workerAdded) { // worker被添加
    // 开始执行worker的run方法，调用线程start启动线程
    t.start();  
    // 设置worker已开始标识
    workerStarted = true;
  }
  //...
  return workerStarted;
}

private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
  public void run() {
    runWorker(this);
  } 
  final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
      while (task != null || (task = getTask()) != null) {
        w.lock();
        if ((runStateAtLeast(ctl.get(), STOP) ||
             (Thread.interrupted() &&
              runStateAtLeast(ctl.get(), STOP))) &&
            !wt.isInterrupted())
          wt.interrupt();
        try {
          beforeExecute(wt, task);
          Throwable thrown = null;
          try {
            // 执行run方法
            task.run();
          } catch (RuntimeException x) {
            thrown = x; throw x;
          } catch (Error x) {
            thrown = x; throw x;
          } catch (Throwable x) {
            thrown = x; throw new Error(x);
          } finally {
            afterExecute(task, thrown);
          }
        } finally {
          task = null;
          w.completedTasks++;
          w.unlock();
        }
      }
      completedAbruptly = false;
    } finally {
      processWorkerExit(w, completedAbruptly);
    }
  }
}
```

### Executors创建线程池

```
public class Executors {

  // 创建一个立即执行无缓冲的线程池
  public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
  }
  
  // 创建一个固定大小的线程池，corePoolSize和maximumPoolSize相等即不会在队列满了创建线程
  // LinkedBlockingQueue默认构造器为无界队列，队列不可能满
  public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
  }
  
  // 创建一个单一线程运行的线程池，corePoolSize和maximumPoolSize相等即不会在队列满了创建线程
  // LinkedBlockingQueue默认构造器为无界队列，队列不可能满
  public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1,
                 0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));
  }
  
  // 创建一个可以线程可以定时调度的线程池
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
  }
  
  public class ScheduledThreadPoolExecutor extends ThreadPoolExecutor implements 
        ScheduledExecutorService
    public ScheduledThreadPoolExecutor(int corePoolSize) {
      super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, 
              new DelayedWorkQueue());
    }
  }

  // 创建一个可执行Fork/Join任务的线程池
  public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool(parallelism,
       ForkJoinPool.defaultForkJoinWorkerThreadFactory,null, true);
  }
}
```

### 关闭线程池

关闭线程池源码

```
/*
  线程池状态变为 SHUTDOWN
   - 不会接收新任务
   - 但已提交任务会执行完
   - 此方法不会阻塞调用线程的执行
*/
public void shutdown() {
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
    checkShutdownAccess();
    // 修改线程池状态
    advanceRunState(SHUTDOWN);
    // 仅会打断空闲线程
    interruptIdleWorkers();
    onShutdown(); // 扩展点 ScheduledThreadPoolExecutor
  } finally {
    mainLock.unlock();
  }
  // 尝试终结(没有运行的线程可以立刻终结，如果还有运行的线程也不会等)
  tryTerminate();
}
/*
  线程池状态变为 STOP
   - 不会接收新任务
   - 会将队列中的任务返回
   - 并用 interrupt 的方式中断正在执行的任务
*/
public List<Runnable> shutdownNow() {
  List<Runnable> tasks;
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
    checkShutdownAccess();
    // 修改线程池状态
    advanceRunState(STOP);
    // 打断所有线程
    interruptWorkers();
    // 获取队列中剩余任务
    tasks = drainQueue();
  } finally {
    mainLock.unlock();
  }
  // 尝试终结
  tryTerminate();
  return tasks;
}
```

## 附录

### 查看进程线程的方法

**linux**

```
ps -ef           # 查看所有进程
ps -fT -p <PID>  # 查看某个进程（PID）的所有线程
top -H -p <PID>  # 查看某个进程（PID）的所有线程
```

------

**Java**

```
jps             # 命令查看所有 Java 进程
jstack <PID>    # 查看某个Java进程（PID）的所有线程状态
jconsole        # 来查看某个Java进程中线程的运行情况（图形界面）
```

### 线程常见方法

- start()

  启动一个新线程，在新的线程运行 run 方法中的代码。start 方法只是让线程进入就绪，里面代码不一定立刻运行（CPU 的时间片还没分给它）。每个线程对象的start方法只能调用一次，如果调用了多次会出现IllegalThreadStateException

- run()

  新线程启动后会调用的方法。如果在构造 Thread 对象时传递了 Runnable 参数，则线程启动后会调用 Runnable 中的 run 方法，否则默认不执行任何操作。但可以创建 Thread 的子类对象，来覆盖默认行为

- join() / join(long n)

  等待线程运行结束

- getId()

  获取线程长整型的唯一 id

- getName() / setName(String)

  获取/修改 线程名

- getPriority() / setPriority(int)

  获取/修改 线程优先级。java中规定线程优先级是1~10 的整数，较大的优先级能提高该线程被 CPU 调度的机率

- getState()

  获取线程状态，NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED

- isInterrupted()

  判断是否被打断， 不会清除打断标记

- isAlive()

  线程是否存活（还没有运行完毕）

- interrupt()

  打断线程，如果被打断线程正在 sleep，wait，join 会导致被打断的线程抛出 InterruptedException，并清除打断标记；如果打断的正在运行的线程，则会设置打断标记 ；park 的线程被打断，也会设置打断标记

- ${interrupted()}^{static}$

  判断当前线程是否被打断 ，会清除打断标记

- ${currentThread()}^{static}$

  获取当前正在执行的线程

- ${sleep(long n)}^{static}$

  让当前执行的线程休眠n毫秒，休眠时让出 cpu 的时间片给其它线程

- ${yield()}^{static}$

  提示线程调度器让出当前线程对CPU的使用，主要是为了测试和调试

### 线程中断

 interrupted()是Java提供的一种中断机制，Thread.stop, Thread.suspend, Thread.resume都已经被废弃了，所以在Java中没有办法立即停止一条线程。因此，Java提供了一种用于停止线程的机制`中断`。

- 中断只是一种协作机制，Java没有给中断增加任何语法，中断的过程完全需要程序员自己实现。若要中断一个线程，需要手动调用该线程的interrupted方法，该方法也仅仅是将线程对象的中断标识设成true；接着需要自己写代码不断地检测当前线程的标识位；如果为true，表示别的线程要求这条线程中断，此时究竟该做什么需要自己写代码实现。
- 每个线程对象中都有一个标识，用于表示线程是否被中断；该标识位为true表示中断，为false表示未中断

**中断的相关方法**

```
// 本地方法，返回线程中断状态，并且根据参数判断是否需要清除标识位
private native boolean isInterrupted(boolean ClearInterrupted);

// 返回线程中断状态，不清空标识
public boolean isInterrupted() {return isInterrupted(false);}

// 静态方法，返回当前线程中断状态，清空标识
public static boolean interrupted() { return currentThread().isInterrupted(true);}
```

**使用中断举例**

```
public class InterruptTest{

  private static ReentrantLock reentrantLock = new ReentrantLock();

  private static class InterruptRunnable implements Runnable {
    @Override
    public void run() {
      String currentThreadName =  Thread.currentThread().getName();
      reentrantLock.lock();
      while (!Thread.interrupted()) {
        System.out.println(currentThreadName + " do business logic");
        for (int i = 0; i < 2000000000; i++) {

        }
        System.out.println(currentThreadName + " do business logic over");
      }
      // 线程发生中断
      System.out.println(currentThreadName + " happen interrupt do something");
      // 必须释放资源否则“T-2”线程一直阻塞
      reentrantLock.unlock();
      // 释放资源
      System.out.println(currentThreadName + " release resources");
    }
  }

  public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(new InterruptRunnable(),"T-1");
    thread.start();
    TimeUnit.SECONDS.sleep(1);
    Thread thread2 = new Thread(new InterruptRunnable(), "T-2");
    thread2.start();
    thread.interrupt();
    // 如果调用的是thread.stop()可以释放synchronized资源但是不会释放Lock实现类的资源
  }
}
```

 Java类库中提供的一些可能会发生阻塞的方法都会抛InterruptedException异常，当在某一条线程中调用这些方法时，这个方法可能会被阻塞很长时间，就可以在别的线程中调用当前线程对象的interrupt方法触发这些函数抛出InterruptedException异常，这时候需要自己处理中断了。

### CAS算法

 CAS算法【Compare-And-Swap】是硬件对于并发操作共享数据的支持。CAS是一种无锁算法，CAS有3个操作数①内存值V；②旧的预期值A；③要修改为的新值B；

 更新规则当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。其实就是拿副本中的预期值与主存中的值作比较，如果相等就继续替换新值，如果不相等就说明主存中的值已经被别的线程修改，就继续重试。

![img](https://mynamelancelot.github.io/img/concurrent/CAS.png)

CAS 的底层是 lock cmpxchg 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性。在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的。

**ABA问题**

![ABA](https://mynamelancelot.github.io/img/concurrent/ABA.png)

 线程A从内存中获取到预期值A之后开始进行逻辑运算，会有一个时间差。在这个时间差之间有某些线程对这个内存值V进行了一系列操作，但是最终又恢复了和线程A拿到预期值V时候一样的内存值。线程A进行CAS算法发现内存值V等于预期值A（实际V被改变过只是比较时还是原值）。此时线程A仍然能CAS成功，但是中间多出的那些过程仍然可能引发问题。

> 根据实际情况，判断是否处理ABA问题。如果ABA问题并不会影响我们的业务结果，可以不处理。

 可以通过加一个标记位来解决这个问题。即：如果在用到该引用时，都对该引用标记位进行推进，进行比较时除了要对比内存值外，还要对比标记位的值是否一样，这样就解决了ABA问题。

### 指令重排序

 Java编译器为了优化程序的性能，会重新对字节码指令排序，虽然会重排序，但是指令重排序运行的结果一定是正常的。在程序运行过程中（多核CPU环境下）也可能处理器会对执行的指令进行重排序。

 在单线程中对我们程序的帮助一定是正向的，它能够很好的优化我们程序的性能。但是在多线程环境下，如果由于出现指令重排序情况导致线程安全性问题，这种情况下比较少见（出现概率小，难复现）很隐蔽。例如：双检查锁单例模式

**指令重排序需要满足一定条件才能进行的，能够进行指令重排序的地方，需要看这个段代码是否具有数据依赖性**

| 名称   | 代码示例     | 说明                         |
| :----- | :----------- | :--------------------------- |
| 写后读 | a = 1;b = a; | 写一个变量之后，再读这个位置 |
| 写后写 | a = 1;a = 2; | 写一个变量之后，再写这个变量 |
| 读后写 | a = b;b = 1; | 读一个变量之后，再写这个变量 |

> 在单线程环境下编译器、runtime和处理器都遵循`as-if-serial`语义【不管怎么重排序，执行结果不变】

**final变量的指令重排序**

```
/**
 * 步骤1：为new出来的对象开辟内存空间
 * 步骤2：初始化，执行构造器方法的逻辑代码片段
 * 步骤3：完成obj引用的赋值操作，将其指向刚刚开辟的内存地址
 * 这三个步骤可能进行指令重排序变为：步骤1、步骤3、步骤2
 * 如果发生重排序可能obj不为null但是未初始化完成
 */
ObjectVarilabe obj = new ObjectVarilabe()
```

- 在构造函数内对一个final变量的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这个两个操作不能被重排序，即需要先初始化final变量才可以给构造的对象地址赋给引用【普通域可能在地址赋给引用之后才初始化】

  ```
    public class FinalExample {
      int i = 0;
      final int f;
      static FinalExample obj;
      
      public FinalExample() { // 构造函数
        i = 1;              // 写普通域
        f = 2;              // 写final域
      }
      
      
      public static void writer() { 
        /**
          	 * 此代码执行之时，另一线程读取obj不为null那么obj.f一定初始化完成
          	 * 另一线程读取obj不为null，但是i不一定初始化完成
          	 */
        obj = new FinalExample();	
      }
    }
  ```

- 初次读一个包含final的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序

  ```
    // ①一定发生在③之前，但是①不一定发生在②之前
    public static void reader() { 
      FinalExample example = obj;    // ①读对象引用
      System.out.println(example.i); // ②读普通域
      System.out.println(example.f); // ③读final域
    }
  ```

### happen-before

 JMM(java memory model)可以通过happens-before关系向提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）**happen-before是可见性保证，不是发生性保证**

- **定义**

  1）如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。

  2）两个操作之间存在happens-before关系，**并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行**。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序合法。

- **具体规则**

  - 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作
  - 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁
  - volatile变量规则：对一个volatile变量，happens-before于任意后续对这个volatile域的读
  - 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C
  - start()规则：如果线程A执行操作ThreadB.start()，那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
  - Join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
  - 程序中断规则：对线程interrupted()方法的调用先行于被中断线程的代码检测到中断时间的发生。
  - 对象finalize规则：一个对象的初始化完成（构造函数执行结束）先行于发生它的finalize()方法的开始

### 伪共享

**缓存行**

 CPU的存取最小单位是缓存行。一般一行缓存行有64字节。所以使用缓存时，并不是一个一个字节使用，而是一行缓存行、一行缓存行这样使用。

 在64位系统下，Java数组对象头固定占16字节，而long类型占8个字节。所以16+8*6=64字节，刚好等于一条缓存行的长度，即一个缓存行可以装填6个long类型的数组数据。

**伪共享**

 伪共享的非标准定义为：缓存系统中是以缓存行（cache line）为单位存储的，当多线程修改互相独立的变量时，如果这些变量位于同一个缓存行，只要这个缓存行里有一个变量失效则此缓存行所有数据全部失效。

 在JDK8以前，我们一般是在属性间填充长整型变量来分隔每一组属性。JDK8之后加入`@Contended`注解方式【JVM需要添加参数-XX:-RestrictContended才能开启此功能 】

### 懒汉式指令重排序问题

```
public class Singleton {

  private static volatile Singleton instance;

  private Singleton() {
  }

  public static Singleton getInstance() {
    if(instance==null){                    
      synchronized (Singleton.class){    
        if(instance==null){           
          instance=new Singleton();
        }
      }
    }
    return instance;
  }
}
```

`instance=new Singleton()`这个语句不是一个原子操作，编译后会多条字节码指令：

- 步骤1：为new出来的对象开辟内存空间
- 步骤2：初始化，执行构造器方法的逻辑代码片段
- 步骤3：完成instance引用的赋值操作，将其指向刚刚开辟的内存地址

这三个步骤可能进行指令重排序变为：步骤1、步骤3、步骤2

在发生重排序时其它线程如果进入`getInstance()`方法会在第一次判断是否为null时通过，导致拿到一个还未完全初始化完成的地址空间，此时如果进行操作很可能发生线程安全性问题

### StampedLock

StampedLock是Java 8新增的一个读写锁，它是对ReentrantReadWriteLock的改进。

```
public class Demo {

  private int balance;

  private StampedLock lock = new StampedLock();

  //===================================读写锁转化====================================
  public void conditionReadWrite (int value) {
    // 首先加独占读锁判断balance的值是否符合更新的条件
    long stamp = lock.readLock();
    try{
      while (balance > 0) {
        // 下面需要进行写操作，所以转化为写锁
        // ReentrantReadWriteLock是直接加写锁，锁升级
        long writeStamp = lock.tryConvertToWriteLock(stamp);
        if(writeStamp != 0) { // 成功转换成为写锁
          stamp = writeStamp;
          balance += value;
          break;
        } else {
          // 没有转换成写锁，这里需要首先释放读锁，然后再拿到写锁
          lock.unlockRead(stamp);

          //。。。

          // 获取写锁
          stamp = lock.writeLock();
        }
      }
    }finally{
      lock.unlock(stamp); 
    }
  }

  //====================================乐观读操作===================================
  public void optimisticRead() {
    // 获取乐观读锁
    long stamp = lock.tryOptimisticRead();
    int c = balance;
    // 这里其它可能会出现了写操作，因此要进行判断读锁获取的是否有问题
    if(!lock.validate(stamp)) {
      // 要从新读取
      long readStamp = lock.readLock();
      c = balance;
      stamp = readStamp;
    }
    lock.unlockRead(stamp);
  }

  //==============以下read/write操作和ReentrantReadWriteLock效果完全一样==============
  public void read () {
    long stamp = lock.readLock();
    lock.tryOptimisticRead();
    int c = balance;
    // ...
    lock.unlockRead(stamp);
  }

  public void write(int value) {
    long stamp = lock.writeLock();
    balance += value;
    lock.unlockWrite(stamp);
  }
}
```

Copyright © By-WangYuKun, All Rights Reserved



# Java  SE Interview

## Java 概述

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_1-什么是-java)1.什么是 Java？

![下辈子还学Java](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-1.png)下辈子还学Java

PS：碎怂 Java，有啥好介绍的。哦，面试啊。

Java 是一门面向对象的编程语言，不仅吸收了 C++语言的各种优点，还摒弃了 C++里难以理解的多继承、指针等概念，因此 Java 语言具有功能强大和简单易用两个特征。Java 语言作为静态面向对象编程语言的优秀代表，极好地实现了面向对象理论，允许程序员以优雅的思维方式进行复杂的编程 。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_2-java-语言有哪些特点)2.Java 语言有哪些特点？

Java 语言有很多优秀（可吹）的特点，以下几个是比较突出的：

![Java语言特点](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-2.png)Java语言特点

- 面向对象（封装，继承，多态）；
- 平台无关性，平台无关性的具体表现在于，Java 是“一次编写，到处运行（Write Once，Run any Where）”的语言，因此采用 Java 语言编写的程序具有很好的可移植性，而保证这一点的正是 Java 的虚拟机机制。在引入虚拟机之后，Java 语言在不同的平台上运行不需要重新编译。
- 支持多线程。C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持；
- 编译与解释并存；

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_3-jvm、jdk-和-jre-有什么区别)3.JVM、JDK 和 JRE 有什么区别？

**JVM**：Java Virtual Machine，Java 虚拟机，Java 程序运行在 Java 虚拟机上。针对不同系统的实现（Windows，Linux，macOS）不同的 JVM，因此 Java 语言可以实现跨平台。

**JRE**： Java 运⾏时环境。它是运⾏已编译 Java 程序所需的所有内容的集合，包括 Java 虚拟机（JVM），Java 类库，Java 命令和其他的⼀些基础构件。但是，它不能⽤于创建新程序。

**JDK**: Java Development Kit，它是功能⻬全的 Java SDK。它拥有 JRE 所拥有的⼀切，还有编译器（javac）和⼯具（如 javadoc 和 jdb）。它能够创建和编译程序。

简单来说，JDK 包含 JRE，JRE 包含 JVM。

![JDK、JRE、JVM关系](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-3.png)JDK、JRE、JVM关系

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_4-说说什么是跨平台性-原理是什么)4.说说什么是跨平台性？原理是什么

所谓跨平台性，是指 Java 语言编写的程序，一次编译后，可以在多个系统平台上运行。

实现原理：Java 程序是通过 Java 虚拟机在系统平台上运行的，只要该系统可以安装相应的 Java 虚拟机，该系统就可以运行 java 程序。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_5-什么是字节码-采用字节码的好处是什么)5.什么是字节码？采用字节码的好处是什么?

所谓的字节码，就是 Java 程序经过编译之类产生的.class 文件，字节码能够被虚拟机识别，从而实现 Java 程序的跨平台性。

**Java** 程序从源代码到运行主要有三步：

- **编译**：将我们的代码（.java）编译成虚拟机可以识别理解的字节码(.class)
- **解释**：虚拟机执行 Java 字节码，将字节码翻译成机器能识别的机器码
- **执行**：对应的机器执行二进制机器码

![Java程序执行过程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-4.png)Java程序执行过程

只需要把 Java 程序编译成 Java 虚拟机能识别的 Java 字节码，不同的平台安装对应的 Java 虚拟机，这样就可以可以实现 Java 语言的平台无关性。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_6-为什么说-java-语言-编译与解释并存)6.为什么说 Java 语言“编译与解释并存”？

高级编程语言按照程序的执行方式分为**编译型**和**解释型**两种。

简单来说，编译型语言是指编译器针对特定的操作系统将源代码一次性翻译成可被该平台执行的机器码；解释型语言是指解释器对源程序逐行解释成特定平台的机器码并立即执行。

比如，你想读一本外国的小说，你可以找一个翻译人员帮助你翻译，有两种选择方式，你可以先等翻译人员将全本的小说（也就是源码）都翻译成汉语，再去阅读，也可以让翻译人员翻译一段，你在旁边阅读一段，慢慢把书读完。

Java 语言既具有编译型语言的特征，也具有解释型语言的特征，因为 Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（`\*.class` 文件），这种字节码必须再经过 JVM，解释成操作系统能识别的机器码，在由操作系统执行。因此，我们可以认为 Java 语言**编译**与**解释**并存。

![编译与解释](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-5.png)编译与解释

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#基础语法)基础语法

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_7-java-有哪些数据类型)7.Java 有哪些数据类型？

**定义：**Java 语言是强类型语言，对于每一种数据都定义了明确的具体的数据类型，在内存中分配了不同大小的内存空间。

Java 语言数据类型分为两种：**基本数据类型**和**引用数据类型**。

![Java数据类型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-6.png)Java数据类型

**基本数据类型：**

- 数值型
  - 整数类型（byte、short、int、long）
  - 浮点类型（float、double）
- 字符型（char）
- 布尔型（boolean）

Java 基本数据类型范围和默认值：

| 基本类型  | 位数 | 字节 | 默认值  |
| --------- | ---- | ---- | ------- |
| `int`     | 32   | 4    | 0       |
| `short`   | 16   | 2    | 0       |
| `long`    | 64   | 8    | 0L      |
| `byte`    | 8    | 1    | 0       |
| `char`    | 16   | 2    | 'u0000' |
| `float`   | 32   | 4    | 0f      |
| `double`  | 64   | 8    | 0d      |
| `boolean` | 1    |      | false   |

**引用数据类型：**

- 类（class）
- 接口（interface）
- 数组([])

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_8-自动类型转换、强制类型转换-看看这几行代码)8.自动类型转换、强制类型转换？看看这几行代码？

Java 所有的数值型变量可以相互转换，当把一个表数范围小的数值或变量直接赋给另一个表数范围大的变量时，可以进行自动类型转换；反之，需要强制转换。

![Java自动类型转换方向](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-7.png)Java自动类型转换方向

这就好像，小杯里的水倒进大杯没问题，但大杯的水倒进小杯就不行了，可能会溢出。

> `float f=3.4`，对吗？

不正确。3.4 是双精度数，将双精度型（double）赋值给浮点型（float）属于下转型（down-casting，也称为窄化）会造成精度损失，因此需要强制类型转换`float f =(float)3.4;`或者写成`float f =3.4F`

> `short s1 = 1; s1 = s1 + 1；`对吗？`short s1 = 1; s1 += 1;`对吗？

对于 short s1 = 1; s1 = s1 + 1;编译出错，由于 1 是 int 类型，因此 s1+1 运算结果也是 int 型，需要强制转换类型才能赋值给 short 型。

而 short s1 = 1; s1 += 1;可以正确编译，因为 s1+= 1;相当于 s1 = (short(s1 + 1);其中有隐含的强制类型转换。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_9-什么是自动拆箱-封箱)9.什么是自动拆箱/封箱？

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

Java 可以自动对基本数据类型和它们的包装类进行装箱和拆箱。

![装箱和拆箱](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-8.png)装箱和拆箱

举例：



```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_10-和-有什么区别)10.&和&&有什么区别？

&运算符有两种用法：`短路与`、`逻辑与`。

&&运算符是短路与运算。逻辑与跟短路与的差别是非常巨大的，虽然二者都要求运算符左右两端的布尔值都是 true 整个表达式的值才是 true。

&&之所以称为短路运算是因为，如果&&左边的表达式的值是 false，右边的表达式会被直接短路掉，不会进行运算。很多时候我们可能都需要用&&而不是&。

例如在验证用户登录时判定用户名不是 null 而且不是空字符串，应当写为`username != null &&!username.equals("")`，二者的顺序不能交换，更不能用&运算符，因为第一个条件如果不成立，根本不能进行字符串的 equals 比较，否则会产生 NullPointerException 异常。

**注意**：逻辑或运算符（|）和短路或运算符（||）的差别也是如此。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_11-switch-是否能作用在-byte-long-string-上)11.switch 是否能作用在 byte/long/String 上？

Java5 以前 switch(expr)中，expr 只能是 byte、short、char、int。

从 Java 5 开始，Java 中引入了枚举类型， expr 也可以是 enum 类型。

从 Java 7 开始，expr 还可以是字符串(String)，但是长整型(long)在目前所有的版本中都是不可以的。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_12-break-continue-return-的区别及作用)12.break ,continue ,return 的区别及作用？

- break 跳出整个循环，不再执行循环(**结束当前的循环体**)
- continue 跳出本次循环，继续执行下次循环(**结束正在执行的循环 进入下一个循环条件**)
- return 程序返回，不再执行下面的代码(**结束当前的方法 直接返回**)

![break 、continue 、return](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-9.png)break 、continue 、return

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_13-用最有效率的方法计算-2-乘以-8)13.用最有效率的方法计算 2 乘以 8？

2 << 3。**位运算**，数字的二进制位左移三位相当于乘以 2 的三次方。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_14-说说自增自减运算-看下这几个代码运行结果)14.说说自增自减运算？看下这几个代码运行结果？

在写代码的过程中，常见的一种情况是需要某个整数类型变量增加 1 或减少 1，Java 提供了一种特殊的运算符，用于这种表达式，叫做自增运算符（++)和自减运算符（--）。

++和--运算符可以放在变量之前，也可以放在变量之后。

当运算符放在变量之前时(前缀)，先自增/减，再赋值；当运算符放在变量之后时(后缀)，先赋值，再自增/减。

例如，当 `b = ++a` 时，先自增（自己增加 1），再赋值（赋值给 b）；当 `b = a++` 时，先赋值(赋值给 b)，再自增（自己增加 1）。也就是，++a 输出的是 a+1 的值，a++输出的是 a 值。

用一句口诀就是：“符号在前就先加/减，符号在后就后加/减”。

> 看一下这段代码运行结果？



```java
int i  = 1;
i = i++;
System.out.println(i);
```

答案是 1。有点离谱对不对。

对于 JVM 而言，它对自增运算的处理，是会先定义一个临时变量来接收 i 的值，然后进行自增运算，最后又将临时变量赋给了值为 2 的 i，所以最后的结果为 1。

相当于这样的代码：



```java
int i = 1；
int temp = i;
i++；
i = temp;
System.out.println(i);
```

> 这段代码会输出什么？



```java
int count = 0;
for(int i = 0;i < 100;i++)
{
    count = count++;
}
System.out.println("count = "+count);
```

答案是 0。

和上面的题目一样的道理，同样是用了临时变量，count 实际是等于临时变量的值。



```java
int autoAdd(int count)
{
    int temp = count;
    count = coutn + 1;
    return temp;
}
```

PS：笔试面试可能会碰到的奇葩题，开发这么写，见一次吊一次。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#面向对象)面向对象

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_15-面向对象和面向过程的区别)15.⾯向对象和⾯向过程的区别?

- **⾯向过程** ：面向过程就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候再一个一个的一次调用就可以。
- **⾯向对象** ：面向对象，把构成问题的事务分解成各个对象，而建立对象的目的也不是为了完成一个个步骤，而是为了描述某个事件在解决整个问题的过程所发生的行为。 目的是为了写出通用的代码，加强代码的重用，屏蔽差异性。

用一个比喻：面向过程是编年体；面向对象是纪传体。

![面向对象和面向过程的区别](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-10.png)面向对象和面向过程的区别

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_16-面向对象有哪些特性)16.面向对象有哪些特性

![面向对象三大特征](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-11.png)面向对象三大特征

- **封装**

  封装把⼀个对象的属性私有化，同时提供⼀些可以被外界访问的属性的⽅法。

- **继承**

  继承是使⽤已存在的类的定义作为基础创建新的类，新类的定义可以增加新的属性或新的方法，也可以继承父类的属性和方法。通过继承可以很方便地进行代码复用。

> 关于继承有以下三个要点：

1. ⼦类拥有⽗类对象所有的属性和⽅法（包括私有属性和私有⽅法），但是⽗类中的私有属性和⽅法⼦类是⽆法访问，只是拥有。
2. ⼦类可以拥有⾃⼰属性和⽅法，即⼦类可以对⽗类进⾏扩展。
3. ⼦类可以⽤⾃⼰的⽅式实现⽗类的⽅法。

- **多态**

  所谓多态就是指程序中定义的引⽤变量所指向的具体类型和通过该引⽤变量发出的⽅法调⽤在编程时并不确定，⽽是在程序运⾏期间才确定，即⼀个引⽤变量到底会指向哪个类的实例对象，该引⽤变量发出的⽅法调⽤到底是哪个类中实现的⽅法，必须在由程序运⾏期间才能决定。

  在 Java 中有两种形式可以实现多态：继承（多个⼦类对同⼀⽅法的重写）和接⼝（实现接⼝并覆盖接⼝中同⼀⽅法）。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_17-重载-overload-和重写-override-的区别)17.重载（overload）和重写（override）的区别？

方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。

- 重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载；
- 重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。

方法重载的规则：

1. 方法名一致，参数列表中参数的顺序，类型，个数不同。
2. 重载与方法的返回值无关，存在于父类和子类，同类中。
3. 可以抛出不同的异常，可以有不同修饰符。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_18-访问修饰符-public、private、protected、以及不写-默认-时的区别)18.访问修饰符 public、private、protected、以及不写（默认）时的区别？

Java 中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。Java 支持 4 种不同的访问权限。

- **default** (即默认，什么也不写）: 在同一包内可见，不使用任何修饰符。可以修饰在类、接口、变量、方法。
- **private** : 在同一类内可见。可以修饰变量、方法。**注意：不能修饰类（外部类）**
- **public** : 对所有类可见。可以修饰类、接口、变量、方法
- **protected** : 对同一包内的类和所有子类可见。可以修饰变量、方法。**注意：不能修饰类（外部类）**。

![访问修饰符和可见性](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-12.png)访问修饰符和可见性

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_19-this-关键字有什么作用)19.this 关键字有什么作用？

this 是自身的一个对象，代表对象本身，可以理解为：**指向对象本身的一个指针**。

this 的用法在 Java 中大体可以分为 3 种：

1. 普通的直接引用，this 相当于是指向当前对象本身
2. 形参与成员变量名字重名，用 this 来区分：



```java
public Person(String name,int age){
    this.name=name;
    this.age=age;
}
```

1. 引用本类的构造函数

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_20-抽象类-abstract-class-和接口-interface-有什么区别)20.抽象类(abstract class)和接口(interface)有什么区别？

1. 接⼝的⽅法默认是 public ，所有⽅法在接⼝中不能有实现(Java 8 开始接⼝⽅法可以有默认实现），⽽抽象类可以有⾮抽象的⽅法。
2. 接⼝中除了 static 、 final 变量，不能有其他变量，⽽抽象类中则不⼀定。
3. ⼀个类可以实现多个接⼝，但只能实现⼀个抽象类。接⼝⾃⼰本身可以通过 extends 关键字扩展多个接⼝。
4. 接⼝⽅法默认修饰符是 public ，抽象⽅法可以有 public 、 protected 和 default 这些修饰符（抽象⽅法就是为了被重写所以不能使⽤ private 关键字修饰！）。
5. 从设计层⾯来说，抽象是对类的抽象，是⼀种模板设计，⽽接⼝是对⾏为的抽象，是⼀种⾏为的规范。

> 1. 在 JDK8 中，接⼝也可以定义静态⽅法，可以直接⽤接⼝名调⽤。实现类和实现是不可以调⽤的。如果同时实现两个接⼝，接⼝中定义了⼀样的默认⽅法，则必须重写，不然会报错。
> 2. jdk9 的接⼝被允许定义私有⽅法 。

总结⼀下 jdk7~jdk9 Java 中接⼝的变化：

1. 在 jdk 7 或更早版本中，接⼝⾥⾯只能有常量变量和抽象⽅法。这些接⼝⽅法必须由选择实现接⼝的类实现。
2. jdk 8 的时候接⼝可以有默认⽅法和静态⽅法功能。
3. jdk 9 在接⼝中引⼊了私有⽅法和私有静态⽅法。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_21-成员变量与局部变量的区别有哪些)21.成员变量与局部变量的区别有哪些？

1. **从语法形式上看**：成员变量是属于类的，⽽局部变量是在⽅法中定义的变量或是⽅法的参数；成员变量可以被 public , private , static 等修饰符所修饰，⽽局部变量不能被访问控制修饰符及 static 所修饰；但是，成员变量和局部变量都能被 final 所修饰。
2. **从变量在内存中的存储⽅式来看**：如果成员变量是使⽤ static 修饰的，那么这个成员变量是属于类的，如果没有使⽤ static 修饰，这个成员变量是属于实例的。对象存于堆内存，如果局部变量类型为基本数据类型，那么存储在栈内存，如果为引⽤数据类型，那存放的是指向堆内存对象的引⽤或者是指向常量池中的地址。
3. **从变量在内存中的⽣存时间上看**：成员变量是对象的⼀部分，它随着对象的创建⽽存在，⽽局部变量随着⽅法的调⽤⽽⾃动消失。
4. **成员变量如果没有被赋初值**：则会⾃动以类型的默认值⽽赋值（⼀种情况例外:被 final 修饰的成员变量也必须显式地赋值），⽽局部变量则不会⾃动赋值。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_22-静态变量和实例变量的区别-静态方法、实例方法呢)22.静态变量和实例变量的区别？静态方法、实例方法呢？

> 静态变量和实例变量的区别？

**静态变量:** 是被 static 修饰符修饰的变量，也称为类变量，它属于类，不属于类的任何一个对象，一个类不管创建多少个对象，静态变量在内存中有且仅有一个副本。

**实例变量:** 必须依存于某一实例，需要先创建对象然后通过对象才能访问到它。静态变量可以实现让多个对象共享内存。

> 静态⽅法和实例⽅法有何不同?

类似地。

**静态方法**：static 修饰的方法，也被称为类方法。在外部调⽤静态⽅法时，可以使⽤"**类名.⽅法名**"的⽅式，也可以使⽤"**对象名.⽅法名**"的⽅式。静态方法里不能访问类的非静态成员变量和方法。

**实例⽅法**：依存于类的实例，需要使用"**对象名.⽅法名**"的⽅式调用；可以访问类的所有成员变量和方法。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_24-final-关键字有什么作用)24.final 关键字有什么作用？

final 表示不可变的意思，可用于修饰类、属性和方法：

- 被 final 修饰的类不可以被继承

- 被 final 修饰的方法不可以被重写

- 被 final 修饰的变量不可变，被 final 修饰的变量必须被显式第指定初始值，还得注意的是，这里的不可变指的是变量的引用不可变，不是引用指向的内容的不可变。

  例如：



```java
final StringBuilder sb = new StringBuilder("abc");
sb.append("d");
System.out.println(sb);  //abcd
```

一张图说明：

![final修饰变量](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-13.png)final修饰变量

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_25-final、finally、finalize-的区别)25.final、finally、finalize 的区别？

- final 用于修饰变量、方法和类：final 修饰的类不可被继承；修饰的方法不可被重写；修饰的变量不可变。

- finally 作为异常处理的一部分，它只能在 `try/catch` 语句中，并且附带一个语句块表示这段语句最终一定被执行（无论是否抛出异常），经常被用在需要释放资源的情况下，`System.exit (0)` 可以阻断 finally 执行。

- finalize 是在 `java.lang.Object` 里定义的方法，也就是说每一个对象都有这么个方法，这个方法在 `gc` 启动，该对象被回收的时候被调用。

  一个对象的 finalize 方法只会被调用一次，finalize 被调用不一定会立即回收该对象，所以有可能调用 finalize 后，该对象又不需要被回收了，然后到了真正要被回收的时候，因为前面调用过一次，所以不会再次调用 finalize 了，进而产生问题，因此不推荐使用 finalize 方法。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_26-和-equals-的区别)26.==和 equals 的区别？

**==** : 它的作⽤是判断两个对象的地址是不是相等。即，判断两个对象是不是同⼀个对象(基本数据类型 **==** 比较的是值，引⽤数据类型 **==** 比较的是内存地址)。

**equals()** : 它的作⽤也是判断两个对象是否相等。但是这个“相等”一般也分两种情况：

- 默认情况：类没有覆盖 equals() ⽅法。则通过 equals() 比较该类的两个对象时，等价于通过“ **==** ”比较这两个对象，还是相当于比较内存地址。
- 自定义情况：类覆盖了 equals() ⽅法。我们平时覆盖的 equals()方法一般是比较两个对象的内容是否相同，自定义了一个相等的标准，也就是两个对象的值是否相等。

举个例⼦，Person，我们认为两个人的编号和姓名相同，就是一个人：



```java
public class Person {
    private String no;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return Objects.equals(no, person.no) &&
                Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(no, name);
    }
}
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_27-hashcode-与-equals)27.hashCode 与 equals?

这个也是面试常问——“你重写过 hashcode 和 equals 么，为什么重写 equals 时必须重写 hashCode ⽅法？”

> 什么是 HashCode？

hashCode() 的作⽤是获取哈希码，也称为散列码；它实际上是返回⼀个 int 整数，定义在 Object 类中， 是一个本地⽅法，这个⽅法通常⽤来将对象的内存地址转换为整数之后返回。



```java
public native int hashCode();
```

哈希码主要在哈希表这类集合映射的时候用到，哈希表存储的是键值对(key-value)，它的特点是：能根据“键”快速的映射到对应的“值”。这其中就利⽤到了哈希码！

> 为什么要有 hashCode？

上面已经讲了，主要是在哈希表这种结构中用的到。

例如 HashMap 怎么把 key 映射到对应的 value 上呢？用的就是哈希取余法，也就是拿哈希码和存储元素的数组的长度取余，获取 key 对应的 value 所在的下标位置。

> 为什么重写 quals 时必须重写 hashCode ⽅法？

如果两个对象相等，则 hashcode ⼀定也是相同的。两个对象相等，对两个对象分别调⽤ equals ⽅法都返回 true。反之，两个对象有相同的 hashcode 值，它们也不⼀定是相等的 。因此，**equals** ⽅法被覆盖过，则 **hashCode** ⽅法也必须被覆盖。

hashCode() 的默认⾏为是对堆上的对象产⽣独特值。如果没有重写 hashCode() ，则该 class 的两个对象⽆论如何都不会相等（即使这两个对象指向相同的数据）

> 为什么两个对象有相同的 hashcode 值，它们也不⼀定是相等的？

因为可能会**碰撞**， hashCode() 所使⽤的散列算法也许刚好会让多个对象传回相同的散列值。越糟糕的散列算法越容易碰撞，但这也与数据值域分布的特性有关（所谓碰撞也就是指的是不同的对象得到相同的 hashCode ）。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_28-java-是值传递-还是引用传递)28.Java 是值传递，还是引用传递？

Java 语言是**值传递**。Java 语言的方法调用只支持参数的值传递。当一个对象实例作为一个参数被传递到方法中时，参数的值就是对该对象的引用。对象的属性可以在被调用过程中被改变，但对对象引用的改变是不会影响到调用者的。

JVM 的内存分为堆和栈，其中栈中存储了基本数据类型和引用数据类型实例的地址，也就是对象地址。

而对象所占的空间是在堆中开辟的，所以传递的时候可以理解为把变量存储的对象地址给传递过去，因此引用类型也是值传递。

![Java引用数据值传递示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-14.png)Java引用数据值传递示意图

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_29-深拷贝和浅拷贝)29.深拷贝和浅拷贝?

- **浅拷贝**：仅拷贝被拷贝对象的成员变量的值，也就是基本数据类型变量的值，和引用数据类型变量的地址值，而对于引用类型变量指向的堆中的对象不会拷贝。
- **深拷贝**：完全拷贝一个对象，拷贝被拷贝对象的成员变量的值，堆中的对象也会拷贝一份。

例如现在有一个 order 对象，里面有一个 products 列表，它的浅拷贝和深拷贝的示意图：

![浅拷贝和深拷贝示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-15.png)浅拷贝和深拷贝示意图

因此深拷贝是安全的，浅拷贝的话如果有引用类型，那么拷贝后对象，引用类型变量修改，会影响原对象。

> 浅拷贝如何实现呢？

Object 类提供的 clone()方法可以非常简单地实现对象的浅拷贝。

> 深拷贝如何实现呢？

- 重写克隆方法：重写克隆方法，引用类型变量单独克隆，这里可能会涉及多层递归。
- 序列化：可以先将原对象序列化，再反序列化成拷贝对象。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_30-java-创建对象有哪几种方式)30.Java 创建对象有哪几种方式？

Java 中有以下四种创建对象的方式:

![Java创建对象的四种方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-16.png)Java创建对象的四种方式

- new 创建新对象
- 通过反射机制
- 采用 clone 机制
- 通过序列化机制

前两者都需要显式地调用构造方法。对于 clone 机制,需要注意浅拷贝和深拷贝的区别，对于序列化机制需要明确其实现原理，在 Java 中序列化可以通过实现 Externalizable 或者 Serializable 来实现。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#string)String

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_31-string-是-java-基本数据类型吗-可以被继承吗)31.String 是 Java 基本数据类型吗？可以被继承吗？

> String 是 Java 基本数据类型吗？

不是。Java 中的基本数据类型只有 8 个：byte、short、int、long、float、double、char、boolean；除了基本类型（primitive type），剩下的都是引用类型（reference type）。

String 是一个比较特殊的引用数据类型。

> String 类可以继承吗？

不行。String 类使用 final 修饰，是所谓的不可变类，无法被继承。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_32-string-和-stringbuilder、stringbuffer-的区别)32.String 和 StringBuilder、StringBuffer 的区别？

- String：String 的值被创建后不能修改，任何对 String 的修改都会引发新的 String 对象的生成。
- StringBuffer：跟 String 类似，但是值可以被修改，使用 synchronized 来保证线程安全。
- StringBuilder：StringBuffer 的非线程安全版本，性能上更高一些。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_33-string-str1-new-string-abc-和-string-str2-abc-和-区别)33.String str1 = new String("abc")和 String str2 = "abc" 和 区别？

两个语句都会去字符串常量池中检查是否已经存在 “abc”，如果有则直接使用，如果没有则会在常量池中创建 “abc” 对象。

![堆与常量池中的String](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-17.png)堆与常量池中的String

但是不同的是，String str1 = new String("abc") 还会通过 new String() 在堆里创建一个 "abc" 字符串对象实例。所以后者可以理解为被前者包含。

> String s = new String("abc")创建了几个对象？

很明显，一个或两个。如果字符串常量池已经有“abc”，则是一个；否则，两个。

当字符创常量池没有 “abc”，此时会创建如下两个对象：

- 一个是字符串字面量 "abc" 所对应的、字符串常量池中的实例
- 另一个是通过 new String() 创建并初始化的，内容与"abc"相同的实例，在堆中。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_34-string-不是不可变类吗-字符串拼接是如何实现的)34.String 不是不可变类吗？字符串拼接是如何实现的？

String 的确是不可变的，“**+**”的拼接操作，其实是会生成新的对象。

例如：



```java
String a = "hello ";
String b = "world!";
String ab = a + b;
```

在**jdk1.8 之前**，a 和 b 初始化时位于字符串常量池，ab 拼接后的对象位于堆中。经过拼接新生成了 String 对象。如果拼接多次，那么会生成多个中间对象。

内存如下：

![jdk1.8之前的字符串拼接](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-18.png)jdk1.8之前的字符串拼接

在**Java8 时**JDK 对“+”号拼接进行了优化，上面所写的拼接方式会被优化为基于 StringBuilder 的 append 方法进行处理。Java 会在编译期对“+”号进行处理。

下面是通过 javap -verbose 命令反编译字节码的结果，很显然可以看到 StringBuilder 的创建和 append 方法的调用。



```java
stack=2, locals=4, args_size=1
     0: ldc           #2                  // String hello
     2: astore_1
     3: ldc           #3                  // String world!
     5: astore_2
     6: new           #4                  // class java/lang/StringBuilder
     9: dup
    10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
    13: aload_1
    14: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
    17: aload_2
    18: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
    21: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
    24: astore_3
    25: return
```

也就是说其实上面的代码其实相当于：



```java
String a = "hello ";
String b = "world!";
StringBuilder sb = new StringBuilder();
sb.append(a);
sb.append(b);
String ab = sb.toString();
```

此时，如果再笼统的回答：通过加号拼接字符串会创建多个 String 对象，因此性能比 StringBuilder 差，就是错误的了。因为本质上加号拼接的效果最终经过编译器处理之后和 StringBuilder 是一致的。

当然，循环里拼接还是建议用 StringBuilder，为什么，因为循环一次就会创建一个新的 StringBuilder 对象，大家可以自行实验。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_35-intern-方法有什么作用)35.intern 方法有什么作用？

JDK 源码里已经对这个方法进行了说明：



```java
     * <p>
     * When the intern method is invoked, if the pool already contains a
     * string equal to this {@code String} object as determined by
     * the {@link #equals(Object)} method, then the string from the pool is
     * returned. Otherwise, this {@code String} object is added to the
     * pool and a reference to this {@code String} object is returned.
     * <p>
```

意思也很好懂：

- 如果当前字符串内容存在于字符串常量池（即 equals()方法为 true，也就是内容一样），直接返回字符串常量池中的字符串
- 否则，将此 String 对象添加到池中，并返回 String 对象的引用

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#integer)Integer

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_36-integer-a-127-integer-b-127-integer-c-128-integer-d-128-相等吗)36.Integer a= 127，Integer b = 127；Integer c= 128，Integer d = 128；，相等吗?

答案是 a 和 b 相等，c 和 d 不相等。

- 对于基本数据类型==比较的值
- 对于引用数据类型==比较的是地址

Integer a= 127 这种赋值，是用到了 Integer 自动装箱的机制。自动装箱的时候会去缓存池里取 Integer 对象，没有取到才会创建新的对象。

如果整型字面量的值在-128 到 127 之间，那么自动装箱时不会 new 新的 Integer 对象，而是直接引用缓存池中的 Integer 对象，超过范围 a1==b1 的结果是 false



```java
    public static void main(String[] args) {
        Integer a = 127;
        Integer b = 127;
        Integer b1 = new Integer(127);
        System.out.println(a == b); //true
        System.out.println(b==b1);  //false

        Integer c = 128;
        Integer d = 128;
        System.out.println(c == d);  //false
    }
```

> 什么是 Integer 缓存？

因为根据实践发现大部分的数据操作都集中在值比较小的范围，因此 Integer 搞了个缓存池，默认范围是 -128 到 127，可以根据通过设置`JVM-XX:AutoBoxCacheMax=`来修改缓存的最大值，最小值改不了。

实现的原理是 int 在自动装箱的时候会调用 Integer.valueOf，进而用到了 IntegerCache。

![Integer.valueOf](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-19.png)Integer.valueOf

很简单，就是判断下值是否在缓存范围之内，如果是的话去 IntegerCache 中取，不是的话就创建一个新的 Integer 对象。

IntegerCache 是一个静态内部类， 在静态块中会初始化好缓存值。



```java
 private static class IntegerCache {
     ……
     static {
            //创建Integer对象存储
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
         ……
     }
 }
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_37-string-怎么转成-integer-的-原理)37.String 怎么转成 Integer 的？原理？

PS:这道题印象中在一些面经中出场过几次。

String 转成 Integer，主要有两个方法：

- Integer.parseInt(String s)
- Integer.valueOf(String s)

不管哪一种，最终还是会调用 Integer 类内中的`parseInt(String s, int radix)`方法。

抛去一些边界之类的看看核心代码：



```java
public static int parseInt(String s, int radix)
                throws NumberFormatException
    {

        int result = 0;
        //是否是负数
        boolean negative = false;
        //char字符数组下标和长度
        int i = 0, len = s.length();
        ……
        int digit;
        //判断字符长度是否大于0，否则抛出异常
        if (len > 0) {
            ……
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                //返回指定基数中字符表示的数值。（此处是十进制数值）
                digit = Character.digit(s.charAt(i++),radix);
                //进制位乘以数值
                result *= radix;
                result -= digit;
            }
        }
        //根据上面得到的是否负数，返回相应的值
        return negative ? result : -result;
    }
```

去掉枝枝蔓蔓（当然这些枝枝蔓蔓可以去看看，源码 cover 了很多情况），其实剩下的就是一个简单的字符串遍历计算，不过计算方式有点反常规，是用负的值累减。

![parseInt示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-20.png)parseInt示意图

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#object)Object

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_38-object-类的常见方法)38.Object 类的常见方法?

Object 类是一个特殊的类，是所有类的父类，也就是说所有类都可以调用它的方法。它主要提供了以下 11 个方法，大概可以分为六类：

![Object类的方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-21.png)Object类的方法

**对象比较**：

- public native int hashCode() ：native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的 HashMap。
- public boolean equals(Object obj)：用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写用户比较字符串的值是否相等。

**对象拷贝**：

- protected native Object clone() throws CloneNotSupportedException：naitive 方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为 true，x.clone().getClass() == x.getClass() 为 true。Object 本身没有实现 Cloneable 接口，所以不重写 clone 方法并且进行调用的话会发生 CloneNotSupportedException 异常。

**对象转字符串：**

- public String toString()：返回类的名字@实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。

**多线程调度：**

- public final native void notify()：native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
- public final native void notifyAll()：native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
- public final native void wait(long timeout) throws InterruptedException：native 方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 。timeout 是等待时间。
- public final void wait(long timeout, int nanos) throws InterruptedException：多了 nanos 参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 毫秒。
- public final void wait() throws InterruptedException：跟之前的 2 个 wait 方法一样，只不过该方法一直等待，没有超时时间这个概念

**反射：**

- public final native Class<?> getClass()：native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。

**垃圾回收：**

- protected void finalize() throws Throwable ：通知垃圾收集器回收对象。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#异常处理)异常处理

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_39-java-中异常处理体系)39.Java 中异常处理体系?

Java 的异常体系是分为多层的。

![Java异常体系](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-22.png)Java异常体系

`Throwable`是 Java 语言中所有错误或异常的基类。 Throwable 又分为`Error`和`Exception`，其中 Error 是系统内部错误，比如虚拟机异常，是程序无法处理的。`Exception`是程序问题导致的异常，又分为两种：

- CheckedException 受检异常：编译器会强制检查并要求处理的异常。
- RuntimeException 运行时异常：程序运行中出现异常，比如我们熟悉的空指针、数组下标越界等等

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_40-异常的处理方式)40.异常的处理方式？

针对异常的处理主要有两种方式：

![异常处理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-23.png)异常处理

- **遇到异常不进行具体处理，而是继续抛给调用者 （throw，throws）**

抛出异常有三种形式，一是 throw,一个 throws，还有一种系统自动抛异常。

throws 用在方法上，后面跟的是异常类，可以跟多个；而 throw 用在方法内，后面跟的是异常对象。

- **try catch 捕获异常**

在 catch 语句块中补货发生的异常，并进行处理。



```java
       try {
            //包含可能会出现异常的代码以及声明异常的方法
        }catch(Exception e) {
            //捕获异常并进行处理
        }finally {                                                       }
            //可选，必执行的代码
        }
```

try-catch 捕获异常的时候还可以选择加上 finally 语句块，finally 语句块不管程序是否正常执行，最终它都会必然执行。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_41-三道经典异常处理代码题)41.三道经典异常处理代码题

> 题目 1



```java
public class TryDemo {
    public static void main(String[] args) {
        System.out.println(test());
    }
    public static int test() {
        try {
            return 1;
        } catch (Exception e) {
            return 2;
        } finally {
            System.out.print("3");
        }
    }
}
```

执行结果：31。

try、catch。finally 的基础用法，在 return 前会先执行 finally 语句块，所以是先输出 finally 里的 3，再输出 return 的 1。

> 题目 2



```java
public class TryDemo {
    public static void main(String[] args) {
        System.out.println(test1());
    }
    public static int test1() {
        try {
            return 2;
        } finally {
            return 3;
        }
    }
}
```

执行结果：3。

try 返回前先执行 finally，结果 finally 里不按套路出牌，直接 return 了，自然也就走不到 try 里面的 return 了。

finally 里面使用 return 仅存在于面试题中，实际开发这么写要挨吊的。

> 题目 3



```java
public class TryDemo {
    public static void main(String[] args) {
        System.out.println(test1());
    }
    public static int test1() {
        int i = 0;
        try {
            i = 2;
            return i;
        } finally {
            i = 3;
        }
    }
}
```

执行结果：2。

大家可能会以为结果应该是 3，因为在 return 前会执行 finally，而 i 在 finally 中被修改为 3 了，那最终返回 i 不是应该为 3 吗？

但其实，在执行 finally 之前，JVM 会先将 i 的结果暂存起来，然后 finally 执行完毕后，会返回之前暂存的结果，而不是返回 i，所以即使 i 已经被修改为 3，最终返回的还是之前暂存起来的结果 2。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#i-o)I/O

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_42-java-中-io-流分为几种)42.Java 中 IO 流分为几种?

流按照不同的特点，有很多种划分方式。

- 按照流的流向分，可以分为**输入流**和**输出流**；
- 按照操作单元划分，可以划分为**字节流**和**字符流**；
- 按照流的角色划分为**节点流**和**处理流**

Java Io 流共涉及 40 多个类，看上去杂乱，其实都存在一定的关联， Java I0 流的 40 多个类都是从如下 4 个抽象类基类中派生出来的。

- **InputStream**/**Reader**: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- **OutputStream**/**Writer**: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

![IO-操作方式分类-图片来源参考[2]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-24.jpeg)IO-操作方式分类-图片来源参考[2]

> IO 流用到了什么设计模式？

其实，Java 的 IO 流体系还用到了一个设计模式——**装饰器模式**。

InputStream 相关的部分类图如下，篇幅有限，装饰器模式就不展开说了。

![Java IO流用到装饰器模式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-25.png)Java IO流用到装饰器模式

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_43-既然有了字节流-为什么还要有字符流)43.既然有了字节流,为什么还要有字符流?

其实字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还比较耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。

所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_44-bio、nio、aio)44.BIO、NIO、AIO？

![BIO、NIO、AIO](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-26.png)BIO、NIO、AIO

**BIO**(blocking I/O) ： 就是传统的 IO，同步阻塞，服务器实现模式为一个连接一个线程，即**客户端有连接请求时服务器端就需要启动一个线程进行处理**，如果这个连接不做任何事情会造成不必要的线程开销，可以通过连接池机制改善(实现多个客户连接服务器)。

![BIO、NIO、AIO](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-27.png)BIO、NIO、AIO

BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，程序简单易理解。

**NIO** ：全称 java non-blocking IO，是指 JDK 提供的新 API。从 JDK1.4 开始，Java 提供了一系列改进的输入/输出的新特性，被统称为 NIO(即 New IO)。

NIO 是**同步非阻塞**的，服务器端用一个线程处理多个连接，客户端发送的连接请求会注册到多路复用器上，多路复用器轮询到连接有 IO 请求就进行处理：

![NIO线程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-28.png)NIO线程

NIO 的数据是面向**缓冲区 Buffer**的，必须从 Buffer 中读取或写入。

所以完整的 NIO 示意图：

![NIO完整示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-29.png)NIO完整示意图

可以看出，NIO 的运行机制：

- 每个 Channel 对应一个 Buffer。
- Selector 对应一个线程，一个线程对应多个 Channel。
- Selector 会根据不同的事件，在各个通道上切换。
- Buffer 是内存块，底层是数据。

**AIO**：JDK 7 引入了 Asynchronous I/O，是**异步不阻塞**的 IO。在进行 I/O 编程中，常用到两种模式：Reactor 和 Proactor。Java 的 NIO 就是 Reactor，当有事件触发时，服务器端得到通知，进行相应的处理，完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#序列化)序列化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_45-什么是序列化-什么是反序列化)45.什么是序列化？什么是反序列化？

什么是序列化，序列化就是**把 Java 对象转为二进制流**，方便存储和传输。

所以**反序列化就是把二进制流恢复成对象**。

![序列化和反序列化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-30.png)序列化和反序列化

类比我们生活中一些大件物品的运输，运输的时候把它拆了打包，用的时候再拆包组装。

> Serializable 接口有什么用？

这个接口只是一个标记，没有具体的作用，但是如果不实现这个接口，在有些序列化场景会报错，所以一般建议，创建的 JavaBean 类都实现 Serializable。

> serialVersionUID 又有什么用？

serialVersionUID 就是起验证作用。



```java
private static final long serialVersionUID = 1L;
```

我们经常会看到这样的代码，这个 ID 其实就是用来验证序列化的对象和反序列化对应的对象 ID 是否一致。

这个 ID 的数字其实不重要，无论是 1L 还是 IDE 自动生成的，只要序列化时候对象的 serialVersionUID 和反序列化时候对象的 serialVersionUID 一致的话就行。

如果没有显示指定 serialVersionUID ，则编译器会根据类的相关信息自动生成一个，可以认为是一个指纹。

所以如果你没有定义一个 serialVersionUID， 结果序列化一个对象之后，在反序列化之前把对象的类的结构改了，比如增加了一个成员变量，则此时的反序列化会失败。

因为类的结构变了，所以 serialVersionUID 就不一致。

> Java 序列化不包含静态变量？

序列化的时候是不包含静态变量的。

> 如果有些变量不想序列化，怎么办？

对于不想进行序列化的变量，使用`transient`关键字修饰。

`transient` 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 `transient` 修饰的变量值不会被持久化和恢复。`transient` 只能修饰变量，不能修饰类和方法。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_46-说说有几种序列化方式)46.说说有几种序列化方式？

Java 序列化方式有很多，常见的有三种：

![Java常见序列化方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-31.png)Java常见序列化方式

- Java 对象序列化 ：Java 原生序列化方法即通过 Java 原生流(InputStream 和 OutputStream 之间的转化)的方式进行转化，一般是对象输出流 `ObjectOutputStream`和对象输入流`ObjectInputStream`。
- Json 序列化：这个可能是我们最常用的序列化方式，Json 序列化的选择很多，一般会使用 jackson 包，通过 ObjectMapper 类来进行一些操作，比如将对象转化为 byte 数组或者将 json 串转化为对象。
- ProtoBuff 序列化：ProtocolBuffer 是一种轻便高效的结构化数据存储格式，ProtoBuff 序列化对象可以很大程度上将其压缩，可以大大减少数据传输大小，提高系统性能。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#泛型)泛型

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_47-java-泛型了解么-什么是类型擦除-介绍一下常用的通配符)47.Java 泛型了解么？什么是类型擦除？介绍一下常用的通配符？

> 什么是泛型？

Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。



```java
List<Integer> list = new ArrayList<>();

list.add(12);
//这里直接添加会报错
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
//但是通过反射添加，是可以的
add.invoke(list, "kl");

System.out.println(list);
```

泛型一般有三种使用方式:**泛型类**、**泛型接口**、**泛型方法**。

![泛型类、泛型接口、泛型方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-32.png)泛型类、泛型接口、泛型方法

**1.泛型类**：



```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

如何实例化泛型类：



```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

**2.泛型接口** ：



```java
public interface Generator<T> {
    public T method();
}
```

实现泛型接口，指定类型：



```java
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

**3.泛型方法** ：



```java
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

使用：



```java
// 创建不同类型数组： Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

> 泛型常用的通配符有哪些？

**常用的通配符为： T，E，K，V，？**

- ？ 表示不确定的 java 类型
- T (type) 表示具体的一个 java 类型
- K V (key value) 分别代表 java 键值中的 Key Value
- E (element) 代表 Element

> 什么是泛型擦除？

所谓的泛型擦除，官方名叫“类型擦除”。

Java 的泛型是伪泛型，这是因为 Java 在编译期间，所有的类型信息都会被擦掉。

也就是说，在运行的时候是没有泛型的。

例如这段代码，往一群猫里放条狗：



```java
LinkedList<Cat> cats = new LinkedList<Cat>();
LinkedList list = cats;  // 注意我在这里把范型去掉了，但是list和cats是同一个链表！
list.add(new Dog());  // 完全没问题！
```

因为 Java 的范型只存在于源码里，编译的时候给你静态地检查一下范型类型是否正确，而到了运行时就不检查了。上面这段代码在 JRE（Java**运行**环境）看来和下面这段没区别：



```java
LinkedList cats = new LinkedList();  // 注意：没有范型！
LinkedList list = cats;
list.add(new Dog());
```

为什么要类型擦除呢？

主要是为了向下兼容，因为 JDK5 之前是没有泛型的，为了让 JVM 保持向下兼容，就出了类型擦除这个策略。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#注解)注解

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_48-说一下你对注解的理解)48.说一下你对注解的理解？

**Java 注解本质上是一个标记**，可以理解成生活中的一个人的一些小装扮，比如戴什么什么帽子，戴什么眼镜。

![Java注解和帽子](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-33.png)Java注解和帽子

注解可以标记在类上、方法上、属性上等，标记自身也可以设置一些值，比如帽子颜色是绿色。

有了标记之后，我们就可以在编译或者运行阶段去识别这些标记，然后搞一些事情，这就是注解的用处。

例如我们常见的 AOP，使用注解作为切点就是运行期注解的应用；比如 lombok，就是注解在编译期的运行。

注解生命周期有三大类，分别是：

- RetentionPolicy.SOURCE：给编译器用的，不会写入 class 文件
- RetentionPolicy.CLASS：会写入 class 文件，在类加载阶段丢弃，也就是运行的时候就没这个信息了
- RetentionPolicy.RUNTIME：会写入 class 文件，永久保存，可以通过反射获取注解信息

所以我上文写的是解析的时候，没写具体是解析啥，因为不同的生命周期的解析动作是不同的。

像常见的：

![Override注解](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-34.png)Override注解

就是给编译器用的，编译器编译的时候检查没问题就 over 了，class 文件里面不会有 Override 这个标记。

再比如 Spring 常见的 Autowired ，就是 RUNTIME 的，所以**在运行的时候可以通过反射得到注解的信息**，还能拿到标记的值 required 。

![Autowired注解](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-35.png)Autowired注解

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#反射)反射

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_49-什么是反射-应用-原理)49.什么是反射？应用？原理？

> 什么是反射？

我们通常都是利用`new`方式来创建对象实例，这可以说就是一种“正射”，这种方式在编译时候就确定了类型信息。

而如果，我们想在时候动态地获取类信息、创建类实例、调用类方法这时候就要用到**反射**。

通过反射你可以获取任意一个类的所有属性和方法，你还可以调用这些方法和属性。

反射最核心的四个类：

![Java反射相关类](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-36.png)Java反射相关类

> 反射的应用场景？

一般我们平时都是在在写业务代码，很少会接触到直接使用反射机制的场景。

但是，这并不代表反射没有用。相反，正是因为反射，你才能这么轻松地使用各种框架。像 Spring/Spring Boot、MyBatis 等等框架中都大量使用了反射机制。

像 Spring 里的很多 **注解** ，它真正的功能实现就是利用反射。

就像为什么我们使用 Spring 的时候 ，一个`@Component`注解就声明了一个类为 Spring Bean 呢？为什么通过一个 `@Value`注解就读取到配置文件中的值呢？究竟是怎么起作用的呢？

这些都是因为我们可以基于反射操作类，然后获取到类/属性/方法/方法的参数上的注解，注解这里就有两个作用，一是标记，我们对注解标记的类/属性/方法进行对应的处理；二是注解本身有一些信息，可以参与到处理的逻辑中。

> 反射的原理？

我们都知道 Java 程序的执行分为编译和运行两步，编译之后会生成字节码(.class)文件，JVM 进行类加载的时候，会加载字节码文件，将类型相关的所有信息加载进方法区，反射就是去获取这些信息，然后进行各种操作。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#jdk1-8-新特性)JDK1.8 新特性

JDK 已经出到 17 了，但是你迭代你的版本，我用我的 8。JDK1.8 的一些新特性，当然现在也不新了，其实在工作中已经很常用了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_50-jdk1-8-都有哪些新特性)50.JDK1.8 都有哪些新特性？

JDK1.8 有不少新特性，我们经常接触到的新特性如下：

![JDK1.8主要新特性](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-37.png)JDK1.8主要新特性

- 接口默认方法：Java 8 允许我们给接口添加一个非抽象的方法实现，只需要使用 default 关键字修饰即可

- Lambda 表达式和函数式接口：Lambda 表达式本质上是一段匿名内部类，也可以是一段可以传递的代码。Lambda 允许把函数作为一个方法的参数（函数作为参数传递到方法中），使用 Lambda 表达式使代码更加简洁，但是也不要滥用，否则会有可读性等问题，《Effective Java》作者 Josh Bloch 建议使用 Lambda 表达式最好不要超过 3 行。

- Stream API：用函数式编程方式在集合类上进行复杂操作的工具，配合 Lambda 表达式可以方便的对集合进行处理。

  Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用 Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。

  简而言之，Stream API 提供了一种高效且易于使用的处理数据的方式。

- 日期时间 API：Java 8 引入了新的日期时间 API 改进了日期时间的管理。

- Optional 类：用来解决空指针异常的问题。很久以前 Google Guava 项目引入了 Optional 作为解决空指针异常的一种方式，不赞成代码被 null 检查的代码污染，期望程序员写整洁的代码。受 Google Guava 的鼓励，Optional 现在是 Java 8 库的一部分。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_51-lambda-表达式了解多少)51.Lambda 表达式了解多少？

Lambda 表达式本质上是一段匿名内部类，也可以是一段可以传递的代码。

比如我们以前使用 Runnable 创建并运行线程：



```java
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread is running before Java8!");
            }
        }).start();
```

这是通过内部类的方式来重写 run 方法，使用 Lambda 表达式，还可以更加简洁：



```java
new Thread( () -> System.out.println("Thread is running since Java8!") ).start();
```

当然不是每个接口都可以缩写成 Lambda 表达式。只有那些函数式接口（Functional Interface）才能缩写成 Lambda 表示式。

所谓函数式接口（Functional Interface）就是只包含一个抽象方法的声明。针对该接口类型的所有 Lambda 表达式都会与这个抽象方法匹配。

> Java8 有哪些内置函数式接口？

JDK 1.8 API 包含了很多内置的函数式接口。其中就包括我们在老版本中经常见到的 **Comparator** 和 **Runnable**，Java 8 为他们都添加了 @FunctionalInterface 注解，以用来支持 Lambda 表达式。

除了这两个之外，还有 Callable、Predicate、Function、Supplier、Consumer 等等。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_52-optional-了解吗)52.Optional 了解吗？

`Optional`是用于防范`NullPointerException`。

可以将 `Optional` 看做是包装对象（可能是 `null`, 也有可能非 `null`）的容器。当我们定义了 一个方法，这个方法返回的对象可能是空，也有可能非空的时候，我们就可以考虑用 `Optional` 来包装它，这也是在 Java 8 被推荐使用的做法。



```java
Optional<String> optional = Optional.of("bam");

optional.isPresent();           // true
optional.get();                 // "bam"
optional.orElse("fallback");    // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javase.html#_53-stream-流用过吗)53.Stream 流用过吗？

`Stream` 流，简单来说，使用 `java.util.Stream` 对一个包含一个或多个元素的集合做各种操作。这些操作可能是 *中间操作* 亦或是 *终端操作*。 终端操作会返回一个结果，而中间操作会返回一个 `Stream` 流。

Stream 流一般用于集合，我们对一个集合做几个常见操作：



```java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```

- **Filter 过滤**



```java
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa2", "aaa1"
```

- **Sorted 排序**



```java
stringCollection
    .stream()
    .sorted()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa1", "aaa2"
```

- **Map 转换**



```java
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
```

- **Match 匹配**



```java
// 验证 list 中 string 是否有以 a 开头的, 匹配到第一个，即返回 true
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true

// 验证 list 中 string 是否都是以 a 开头的
boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false

// 验证 list 中 string 是否都不是以 z 开头的,
boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true
```

- **Count 计数**

`count` 是一个终端操作，它能够统计 `stream` 流中的元素总数，返回值是 `long` 类型。



```java
// 先对 list 中字符串开头为 b 进行过滤，让后统计数量
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();

System.out.println(startsWithB);    // 3
```

- **Reduce**

`Reduce` 中文翻译为：*减少、缩小*。通过入参的 `Function`，我们能够将 `list` 归约成一个值。它的返回类型是 `Optional` 类型。



```java
Optional<String> reduced =
    stringCollection
        .stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);
// "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
```

以上是常见的几种流式操作，还有其它的一些流式操作，可以帮助我们更便捷地处理集合数据。

![Java Stream流](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javase-38.png)Java Stream流

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。





# Java  Collection  Interview

## 引言

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_1-说说有哪些常见集合)1.说说有哪些常见集合？

集合相关类和接口都在java.util中，主要分为3种：List（列表）、Map（映射）、Set(集)。

![Java集合主要关系](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-1.png)Java集合主要关系

其中`Collection`是集合`List`、`Set`的父接口，它主要有两个子接口：

- `List`：存储的元素有序，可重复。
- `Set`：存储的元素不无序，不可重复。

`Map`是另外的接口，是键值对映射结构的集合。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#list)List

List，也没啥好问的，但不排除面试官剑走偏锋，比如面试官也看了我这篇文章。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_2-arraylist和linkedlist有什么区别)2.ArrayList和LinkedList有什么区别？

**（1）**数据结构不同

- ArrayList基于数组实现
- LinkedList基于双向链表实现

![ArrayList和LinkedList的数据结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-2.png)ArrayList和LinkedList的数据结构

**（2）** 多数情况下，ArrayList更利于查找，LinkedList更利于增删

- ArrayList基于数组实现，get(int index)可以直接通过数组下标获取，时间复杂度是O(1)；LinkedList基于链表实现，get(int index)需要遍历链表，时间复杂度是O(n)；当然，get(E element)这种查找，两种集合都需要遍历，时间复杂度都是O(n)。
- ArrayList增删如果是数组末尾的位置，直接插入或者删除就可以了，但是如果插入中间的位置，就需要把插入位置后的元素都向前或者向后移动，甚至还有可能触发扩容；双向链表的插入和删除只需要改变前驱节点、后继节点和插入节点的指向就行了，不需要移动元素。

![ArrayList和LinkedList中间插入](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-3.png)ArrayList和LinkedList中间插入

![ArrayList和LinkedList中间删除](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-4.png)ArrayList和LinkedList中间删除

> 注意，这个地方可能会出陷阱，LinkedList更利于增删更多是体现在平均步长上，不是体现在时间复杂度上，二者增删的时间复杂度都是O(n)

**（3）**是否支持随机访问

- ArrayList基于数组，所以它可以根据下标查找，支持随机访问，当然，它也实现了RandmoAccess 接口，这个接口只是用来标识是否支持随机访问。
- LinkedList基于链表，所以它没法根据序号直接获取元素，它没有实现RandmoAccess 接口，标记不支持随机访问。

**（4）**内存占用，ArrayList基于数组，是一块连续的内存空间，LinkedList基于链表，内存空间不连续，它们在空间占用上都有一些额外的消耗：

- ArrayList是预先定义好的数组，可能会有空的内存空间，存在一定空间浪费
- LinkedList每个节点，需要存储前驱和后继，所以每个节点会占用更多的空间

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_3-arraylist的扩容机制了解吗)3.ArrayList的扩容机制了解吗？

ArrayList是基于数组的集合，数组的容量是在定义的时候确定的，如果数组满了，再插入，就会数组溢出。所以在插入时候，会先检查是否需要扩容，如果当前容量+1超过数组长度，就会进行扩容。

ArrayList的扩容是创建一个**1.5倍**的新数组，然后把原数组的值拷贝过去。

![ArrayList扩容](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-5.png)ArrayList扩容

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_4-arraylist怎么序列化的知道吗-为什么用transient修饰数组)4.ArrayList怎么序列化的知道吗？ 为什么用transient修饰数组？

ArrayList的序列化不太一样，它使用`transient`修饰存储元素的`elementData`的数组，`transient`关键字的作用是让被修饰的成员属性不被序列化。

**为什么最ArrayList不直接序列化元素数组呢？**

出于效率的考虑，数组可能长度100，但实际只用了50，剩下的50不用其实不用序列化，这样可以提高序列化和反序列化的效率，还可以节省内存空间。

**那ArrayList怎么序列化呢？**

ArrayList通过两个方法**readObject、writeObject**自定义序列化和反序列化策略，实际直接使用两个流`ObjectOutputStream`和`ObjectInputStream`来进行序列化和反序列化。

![ArrayList自定义序列化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-6.png)ArrayList自定义序列化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_5-快速失败-fail-fast-和安全失败-fail-safe-了解吗)5.快速失败(fail-fast)和安全失败(fail-safe)了解吗？

**快速失败（fail—fast）**：快速失败是Java集合的一种错误检测机制

- 在用迭代器遍历一个集合对象时，如果线程A遍历过程中，线程B对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。
- 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 `modCount` 变量。集合在被遍历期间如果内容发生变化，就会改变`modCount`的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。
- 注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。
- 场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改），比如ArrayList 类。

**安全失败（fail—safe）**

- 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。
- 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。
- 缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。
- 场景：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改，比如CopyOnWriteArrayList类。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_6-有哪几种实现arraylist线程安全的方法)6.有哪几种实现ArrayList线程安全的方法？

fail-fast是一种可能触发的机制，实际上，ArrayList的线程安全仍然没有保证，一般，保证ArrayList的线程安全可以通过这些方案：

- 使用 Vector 代替 ArrayList。（不推荐，Vector是一个历史遗留类）
- 使用 Collections.synchronizedList 包装 ArrayList，然后操作包装后的 list。
- 使用 CopyOnWriteArrayList 代替 ArrayList。
- 在使用 ArrayList 时，应用程序通过同步机制去控制 ArrayList 的读写。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_7-copyonwritearraylist了解多少)7.CopyOnWriteArrayList了解多少？

CopyOnWriteArrayList就是线程安全版本的ArrayList。

它的名字叫`CopyOnWrite`——写时复制，已经明示了它的原理。

CopyOnWriteArrayList采用了一种读写分离的并发策略。CopyOnWriteArrayList容器允许并发读，读操作是无锁的，性能较高。至于写操作，比如向容器中添加一个元素，则首先将当前容器复制一份，然后在新副本上执行写操作，结束之后再将原容器的引用指向新容器。

![CopyOnWriteArrayList原理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-7.png)CopyOnWriteArrayList原理

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#map)Map

Map中，毫无疑问，最重要的就是HashMap，面试基本被盘出包浆了，各种问法，一定要好好准备。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_8-能说一下hashmap的数据结构吗)8.能说一下HashMap的数据结构吗？

JDK1.7的数据结构是`数组`+`链表`，JDK1.7还有人在用？不会吧……

说一下JDK1.8的数据结构吧：

JDK1.8的数据结构是`数组`+`链表`+`红黑树`。

数据结构示意图如下：

![jdk1.8 hashmap数据结构示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-8.png)jdk1.8 hashmap数据结构示意图

其中，桶数组是用来存储数据元素，链表是用来解决冲突，红黑树是为了提高查询的效率。

- 数据元素通过映射关系，也就是散列函数，映射到桶数组对应索引的位置
- 如果发生冲突，从冲突的位置拉一个链表，插入冲突的元素
- 如果链表长度>8&数组大小>=64，链表转为红黑树
- 如果红黑树节点个数<6 ，转为链表

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_9-你对红黑树了解多少-为什么不用二叉树-平衡树呢)9.你对红黑树了解多少？为什么不用二叉树/平衡树呢？

红黑树本质上是一种二叉查找树，为了保持平衡，它又在二叉查找树的基础上增加了一些规则：

1. 每个节点要么是红色，要么是黑色；
2. 根节点永远是黑色的；
3. 所有的叶子节点都是是黑色的（注意这里说叶子节点其实是图中的 NULL 节点）；
4. 每个红色节点的两个子节点一定都是黑色；
5. 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；

![红黑树](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-9.png)红黑树

> 之所以不用二叉树：

红黑树是一种平衡的二叉树，插入、删除、查找的最坏时间复杂度都为 O(logn)，避免了二叉树最坏情况下的O(n)时间复杂度。

> 之所以不用平衡二叉树：

平衡二叉树是比红黑树更严格的平衡树，为了保持保持平衡，需要旋转的次数更多，也就是说平衡二叉树保持平衡的效率更低，所以平衡二叉树插入和删除的效率比红黑树要低。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_10-红黑树怎么保持平衡的知道吗)10.红黑树怎么保持平衡的知道吗？

红黑树有两种方式保持平衡：`旋转`和`染色`。

- 旋转：旋转分为两种，左旋和右旋

![左旋](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-10.png)左旋

![右旋](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-11.png)右旋

- 染⾊：

![染色](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-12.png)染色

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_11-hashmap的put流程知道吗)11.HashMap的put流程知道吗？

先上个流程图吧:

![HashMap插入数据流程图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-13.jpg)HashMap插入数据流程图

1. 首先进行哈希值的扰动，获取一个新的哈希值。`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);`

2. 判断tab是否位空或者长度为0，如果是则进行扩容操作。

   

   ```java
   if ((tab = table) == null || (n = tab.length) == 0)
       n = (tab = resize()).length;
   ```

3. 根据哈希值计算下标，如果对应小标正好没有存放数据，则直接插入即可否则需要覆盖。`tab[i = (n - 1) & hash])`

4. 判断tab[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点。

5. 如果链表中插入节点的时候，链表长度大于等于8，则需要把链表转换为红黑树。`treeifyBin(tab, hash);`

6. 最后所有元素处理完成后，判断是否超过阈值；`threshold`，超过则扩容。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_12-hashmap怎么查找元素的呢)12.HashMap怎么查找元素的呢？

先看流程图：

![HashMap查找流程图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-14.png)HashMap查找流程图

HashMap的查找就简单很多：

1. 使用扰动函数，获取新的哈希值
2. 计算数组下标，获取节点
3. 当前节点和key匹配，直接返回
4. 否则，当前节点是否为树节点，查找红黑树
5. 否则，遍历链表查找

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_13-hashmap的哈希-扰动函数是怎么设计的)13.HashMap的哈希/扰动函数是怎么设计的?

HashMap的哈希函数是先拿到 key 的hashcode，是一个32位的int类型的数值，然后让hashcode的高16位和低16位进行异或操作。



```java
    static final int hash(Object key) {
        int h;
        // key的hashCode和key的hashCode右移16位做异或运算
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

这么设计是为了降低哈希碰撞的概率。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_14-为什么哈希-扰动函数能降hash碰撞)14.为什么哈希/扰动函数能降hash碰撞？

因为 key.hashCode() 函数调用的是 key 键值类型自带的哈希函数，返回 int 型散列值。int 值范围为 **-2147483648~2147483647**，加起来大概 40 亿的映射空间。

只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。

假如 HashMap 数组的初始大小才 16，就需要用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标。

源码中模运算就是把散列值和数组长度 - 1 做一个 "`与&`" 操作，位运算比取余 % 运算要快。



```java
bucketIndex = indexFor(hash, table.length);

static int indexFor(int h, int length) {
     return h & (length-1);
}
```

顺便说一下，这也正好解释了为什么 HashMap 的数组长度要取 2 的整数幂。因为这样（数组长度 - 1）正好相当于一个 “低位掩码”。`与` 操作的结果就是散列值的高位全部归零，只保留低位值，用来做数组下标访问。以初始长度 16 为例，16-1=15。2 进制表示是` 0000 0000 0000 0000 0000 0000 0000 1111`。和某个散列值做 `与` 操作如下，结果就是截取了最低的四位值。

![哈希&运算](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-15.png)哈希&运算

这样是要快捷一些，但是新的问题来了，就算散列值分布再松散，要是只取最后几位的话，碰撞也会很严重。如果散列本身做得不好，分布上成等差数列的漏洞，如果正好让最后几个低位呈现规律性重复，那就更难搞了。

这时候 `扰动函数` 的价值就体现出来了，看一下扰动函数的示意图：

![扰动函数示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-16.jpg)扰动函数示意图

右移 16 位，正好是 32bit 的一半，自己的高半区和低半区做异或，就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性。而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_15-为什么hashmap的容量是2的倍数呢)15.为什么HashMap的容量是2的倍数呢？

- 第一个原因是为了方便哈希取余：

将元素放在table数组上面，是用hash值%数组大小定位位置，而HashMap是用hash值&(数组大小-1)，却能和前面达到一样的效果，这就得益于HashMap的大小是2的倍数，2的倍数意味着该数的二进制位只有一位为1，而该数-1就可以得到二进制位上1变成0，后面的0变成1，再通过&运算，就可以得到和%一样的效果，并且位运算比%的效率高得多

HashMap的容量是2的n次幂时，(n-1)的2进制也就是1111111***111这样形式的，这样与添加元素的hash值进行位运算时，能够充分的散列，使得添加的元素均匀分布在HashMap的每个位置上，减少hash碰撞。

- 第二个方面是在扩容时，利用扩容后的大小也是2的倍数，将已经产生hash碰撞的元素完美的转移到新的table中去

我们可以简单看看HashMap的扩容机制，HashMap中的元素在超过`负载因子*HashMap`大小时就会产生扩容。

![put中的扩容](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-17.png)put中的扩容

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_16-如果初始化hashmap-传一个17的值new-hashmap-它会怎么处理)16.如果初始化HashMap，传一个17的值`new HashMap<>`，它会怎么处理？

简单来说，就是初始化时，传的不是2的倍数时，HashMap会向上寻找`离得最近的2的倍数`，所以传入17，但HashMap的实际容量是32。

我们来看看详情，在HashMap的初始化中，有这样⼀段⽅法；



```java
public HashMap(int initialCapacity, float loadFactor) {
 ...
 this.loadFactor = loadFactor;
 this.threshold = tableSizeFor(initialCapacity);
}
```

- 阀值 threshold ，通过⽅法` tableSizeFor` 进⾏计算，是根据初始化传的参数来计算的。
- 同时，这个⽅法也要要寻找⽐初始值⼤的，最⼩的那个2进制数值。⽐如传了17，我应该找到的是32。



```java
static final int tableSizeFor(int cap) {
 int n = cap - 1;
 n |= n >>> 1;
 n |= n >>> 2;
 n |= n >>> 4;
 n |= n >>> 8;
 n |= n >>> 16;
 return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1; }
```

- MAXIMUM_CAPACITY = 1 << 30，这个是临界范围，也就是最⼤的Map集合。
- 计算过程是向右移位1、2、4、8、16，和原来的数做`|`运算，这主要是为了把⼆进制的各个位置都填上1，当⼆进制的各个位置都是1以后，就是⼀个标准的2的倍数减1了，最后把结果加1再返回即可。

以17为例，看一下初始化计算table容量的过程：

![容量计算](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-18.png)容量计算

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_17-你还知道哪些哈希函数的构造方法呢)17.你还知道哪些哈希函数的构造方法呢？

HashMap里哈希构造函数的方法叫：

- **除留取余法**：H（key)=key%p（p<=N）,关键字除以一个不大于哈希表长度的正整数p，所得余数为地址，当然HashMap里进行了优化改造，效率更高，散列也更均衡。

除此之外，还有这几种常见的哈希函数构造方法：

- **直接定址法**

  直接根据`key`来映射到对应的数组位置，例如1232放到下标1232的位置。

- **数字分析法**

  取`key`的某些数字（例如十位和百位）作为映射的位置

- **平方取中法**

  取`key`平方的中间几位作为映射的位置

- **折叠法**

  将`key`分割成位数相同的几段，然后把它们的叠加和作为映射的位置

![散列函数构造](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-19.png)散列函数构造

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_18-解决哈希冲突有哪些方法呢)18.解决哈希冲突有哪些方法呢？

我们到现在已经知道，HashMap使用链表的原因为了处理哈希冲突，这种方法就是所谓的：

- **链地址法**：在冲突的位置拉一个链表，把冲突的元素放进去。

除此之外，还有一些常见的解决冲突的办法：

- **开放定址法**：开放定址法就是从冲突的位置再接着往下找，给冲突元素找个空位。

  找到空闲位置的方法也有很多种：

  - 线行探查法: 从冲突的位置开始，依次判断下一个位置是否空闲，直至找到空闲位置
  - 平方探查法: 从冲突的位置x开始，第一次增加`1^2`个位置，第二次增加`2^2`…，直至找到空闲的位置
  - ……

![开放定址法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-20.png)开放定址法

- **再哈希法**：换种哈希函数，重新计算冲突元素的地址。
- **建立公共溢出区**：再建一个数组，把冲突的元素放进去。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_19-为什么hashmap链表转红黑树的阈值为8呢)19.为什么HashMap链表转红黑树的阈值为8呢？

树化发生在table数组的长度大于64，且链表的长度大于8的时候。

为什么是8呢？源码的注释也给出了答案。

![源码注释](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-21.png)源码注释

红黑树节点的大小大概是普通节点大小的两倍，所以转红黑树，牺牲了空间换时间，更多的是一种兜底的策略，保证极端情况下的查找效率。

阈值为什么要选8呢？和统计学有关。理想情况下，使用随机哈希码，链表里的节点符合泊松分布，出现节点个数的概率是递减的，节点个数为8的情况，发生概率仅为`0.00000006`。

至于红黑树转回链表的阈值为什么是6，而不是8？是因为如果这个阈值也设置成8，假如发生碰撞，节点增减刚好在8附近，会发生链表和红黑树的不断转换，导致资源浪费。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_20-扩容在什么时候呢-为什么扩容因子是0-75)20.扩容在什么时候呢？为什么扩容因子是0.75？

为了减少哈希冲突发生的概率，当当前HashMap的元素个数达到一个临界值的时候，就会触发扩容，把所有元素rehash之后再放在扩容后的容器中，这是一个相当耗时的操作。

![put时，扩容](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-22.png)put时，扩容

而这个`临界值threshold`就是由加载因子和当前容器的容量大小来确定的，假如采用默认的构造方法：

> 临界值（threshold ）= 默认容量（DEFAULT_INITIAL_CAPACITY） * 默认扩容因子（DEFAULT_LOAD_FACTOR）

![threshold计算](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-23.png)threshold计算

那就是大于`16x0.75=12`时，就会触发扩容操作。

> 那么为什么选择了0.75作为HashMap的默认加载因子呢？

简单来说，这是对`空间`成本和`时间`成本平衡的考虑。

在HashMap中有这样一段注释：

![关于默认负载因子的注释](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-24.png)关于默认负载因子的注释

我们都知道，HashMap的散列构造方式是Hash取余，负载因子决定元素个数达到多少时候扩容。

假如我们设的比较大，元素比较多，空位比较少的时候才扩容，那么发生哈希冲突的概率就增加了，查找的时间成本就增加了。

我们设的比较小的话，元素比较少，空位比较多的时候就扩容了，发生哈希碰撞的概率就降低了，查找时间成本降低，但是就需要更多的空间去存储元素，空间成本就增加了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_21-那扩容机制了解吗)21.那扩容机制了解吗？

HashMap是基于数组+链表和红黑树实现的，但用于存放key值的桶数组的长度是固定的，由初始化参数确定。

那么，随着数据的插入数量增加以及负载因子的作用下，就需要扩容来存放更多的数据。而扩容中有一个非常重要的点，就是jdk1.8中的优化操作，可以不需要再重新计算每一个元素的哈希值。

因为HashMap的初始容量是2的次幂，扩容之后的长度是原来的二倍，新的容量也是2的次幂，所以，元素，要么在原位置，要么在原位置再移动2的次幂。

看下这张图，n为table的长度，图`a`表示扩容前的key1和key2两种key确定索引的位置，图`b`表示扩容后key1和key2两种key确定索引位置。

![扩容之后的索引计算](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-25.png)扩容之后的索引计算

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![扩容位置变化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-26.png)扩容位置变化

所以在扩容时，只需要看原来的hash值新增的那一位是0还是1就行了，是0的话索引没变，是1的化变成`原索引+oldCap`，看看如16扩容为32的示意图：

![扩容节点迁移示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-27.png)扩容节点迁移示意图

扩容节点迁移主要逻辑：

![扩容主要逻辑](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-28.png)扩容主要逻辑

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_22-jdk1-8对hashmap主要做了哪些优化呢-为什么)22.jdk1.8对HashMap主要做了哪些优化呢？为什么？

jdk1.8 的HashMap主要有五点优化：

1. **数据结构**：数组 + 链表改成了数组 + 链表或红黑树

   `原因`：发生 hash 冲突，元素会存入链表，链表过长转为红黑树，将时间复杂度由`O(n)`降为`O(logn)`

2. **链表插入方式**：链表的插入方式从头插法改成了尾插法

   简单说就是插入时，如果数组位置上已经有元素，1.7 将新元素放到数组中，原始节点作为新节点的后继节点，1.8 遍历链表，将元素放置到链表的最后。

   `原因`：因为 1.7 头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环。

3. **扩容rehash**：扩容的时候 1.7 需要对原数组中的元素进行重新 hash 定位在新数组的位置，1.8 采用更简单的判断逻辑，不需要重新通过哈希函数计算位置，新的位置不变或索引 + 新增容量大小。

   `原因：`提高扩容的效率，更快地扩容。

4. **扩容时机**：在插入时，1.7 先判断是否需要扩容，再插入，1.8 先进行插入，插入完成再判断是否需要扩容；

5. **散列函数**：1.7 做了四次移位和四次异或，jdk1.8只做一次。

   `原因`：做 4 次的话，边际效用也不大，改为一次，提升效率。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_23-你能自己设计实现一个hashmap吗)23.你能自己设计实现一个HashMap吗？

这道题**快手**常考。

不要慌，红黑树版咱们多半是写不出来，但是数组+链表版还是问题不大的，详细可见： [手写HashMap，快手面试官直呼内行！open in new window](https://mp.weixin.qq.com/s/Z9yoRZW5itrtgbS-cj0bUg)。

整体的设计：

- 散列函数：hashCode()+除留余数法
- 冲突解决：链地址法
- 扩容：节点重新hash获取位置

![自定义HashMap整体结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-29.png)自定义HashMap整体结构

完整代码：

![完整代码](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-30.png)完整代码

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_24-hashmap-是线程安全的吗-多线程下会有什么问题)24.HashMap 是线程安全的吗？多线程下会有什么问题？

HashMap不是线程安全的，可能会发生这些问题：

- 多线程下扩容死循环。JDK1.7 中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。因此，JDK1.8 使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。
- 多线程的 put 可能导致元素的丢失。多线程同时执行 put 操作，如果计算出来的索引位置是相同的，那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。此问题在 JDK 1.7 和 JDK 1.8 中都存在。
- put 和 get 并发时，可能导致 get 为 null。线程 1 执行 put 时，因为元素个数超出 threshold 而导致 rehash，线程 2 此时执行 get，有可能导致这个问题。这个问题在 JDK 1.7 和 JDK 1.8 中都存在。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_25-有什么办法能解决hashmap线程不安全的问题呢)25.有什么办法能解决HashMap线程不安全的问题呢？

Java 中有 HashTable、Collections.synchronizedMap、以及 ConcurrentHashMap 可以实现线程安全的 Map。

- HashTable 是直接在操作方法上加 synchronized 关键字，锁住整个table数组，粒度比较大；
- Collections.synchronizedMap 是使用 Collections 集合工具的内部类，通过传入 Map 封装出一个 SynchronizedMap 对象，内部定义了一个对象锁，方法内通过对象锁实现；
- ConcurrentHashMap 在jdk1.7中使用分段锁，在jdk1.8中使用CAS+synchronized。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_26-能具体说一下concurrenthashmap的实现吗)26.能具体说一下ConcurrentHashmap的实现吗？

ConcurrentHashmap线程安全在jdk1.7版本是基于`分段锁`实现，在jdk1.8是基于`CAS+synchronized`实现。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_1-7分段锁)1.7分段锁

从结构上说，1.7版本的ConcurrentHashMap采用分段锁机制，里面包含一个Segment数组，Segment继承于ReentrantLock，Segment则包含HashEntry的数组，HashEntry本身就是一个链表的结构，具有保存key、value的能力能指向下一个节点的指针。

实际上就是相当于每个Segment都是一个HashMap，默认的Segment长度是16，也就是支持16个线程的并发写，Segment之间相互不会受到影响。

![1.7ConcurrentHashMap示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-31.png)1.7ConcurrentHashMap示意图

**put流程**

整个流程和HashMap非常类似，只不过是先定位到具体的Segment，然后通过ReentrantLock去操作而已，后面的流程，就和HashMap基本上是一样的。

1. 计算hash，定位到segment，segment如果是空就先初始化
2. 使用ReentrantLock加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定获取锁成功
3. 遍历HashEntry，就是和HashMap一样，数组中key和hash一样就直接替换，不存在就再插入链表，链表同样操作

![jdk1.7 put流程](https://gitee.com/sanfene/picgo3/raw/master/20211128230624.jpg)

**get流程**

get也很简单，key通过hash定位到segment，再遍历链表定位到具体的元素上，需要注意的是value是volatile的，所以get是不需要加锁的。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_1-8-cas-synchronized)**1.8 CAS+synchronized**

jdk1.8实现线程安全不是在数据结构上下功夫，它的数据结构和HashMap是一样的，数组+链表+红黑树。它实现线程安全的关键点在于put流程。

**put流程**

1. 首先计算hash，遍历node数组，如果node是空的话，就通过CAS+自旋的方式初始化



```java
 tab = initTable();
```

node数组初始化：



```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        //如果正在初始化或者扩容
        if ((sc = sizeCtl) < 0)
            //等待
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {   //CAS操作
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

2.如果当前数组位置是空则直接通过CAS自旋写入数据



```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

1. 如果hash==MOVED，说明需要扩容，执行扩容



```java
else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
```



```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    if (tab != null && (f instanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        int rs = resizeStamp(tab.length);
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

1. 如果都不满足，就使用synchronized写入数据，写入数据同样判断链表、红黑树，链表写入和HashMap的方式一样，key hash一样就覆盖，反之就尾插法，链表长度超过8就转换成红黑树



```java
 synchronized (f){
     ……
 }
```

![ConcurrentHashmap jdk1.8put流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-32.jpg)ConcurrentHashmap jdk1.8put流程

**get查询**

get很简单，和HashMap基本相同，通过key计算位置，table该位置key相同就返回，如果是红黑树按照红黑树获取，否则就遍历链表获取。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_27-hashmap-内部节点是有序的吗)27.HashMap 内部节点是有序的吗？

HashMap是无序的，根据 hash 值随机插入。如果想使用有序的Map，可以使用LinkedHashMap 或者 TreeMap。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_28-讲讲-linkedhashmap-怎么实现有序的)28.讲讲 LinkedHashMap 怎么实现有序的？

LinkedHashMap维护了一个双向链表，有头尾节点，同时 LinkedHashMap 节点 Entry 内部除了继承 HashMap 的 Node 属性，还有 before 和 after 用于标识前置节点和后置节点。

![Entry节点](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-33.png)Entry节点

可以实现按插入的顺序或访问顺序排序。

![LinkedHashMap实现原理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-34.png)LinkedHashMap实现原理

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_29-讲讲-treemap-怎么实现有序的)29.讲讲 TreeMap 怎么实现有序的？

TreeMap 是按照 Key 的自然顺序或者 Comprator 的顺序进行排序，内部是通过红黑树来实现。所以要么 key 所属的类实现 Comparable 接口，或者自定义一个实现了 Comparator 接口的比较器，传给 TreeMap 用于 key 的比较。

![TreeMap](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-35.png)TreeMap

## [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#set)Set

Set面试没啥好问的，拿HashSet来凑个数。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/collection.html#_30-讲讲hashset的底层实现)30.讲讲HashSet的底层实现？

HashSet 底层就是基于 HashMap 实现的。（ HashSet 的源码⾮常⾮常少，因为除了 clone() 、 writeObject() 、 readObject() 是 HashSet⾃⼰不得不实现之外，其他⽅法都是直接调⽤ HashMap 中的⽅法。

HashSet的add方法，直接调用HashMap的put方法，将添加的元素作为key，new一个Object作为value，直接调用HashMap的put方法，它会根据返回值是否为空来判断是否插入元素成功。



```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

![HashSet套娃](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-36.png)HashSet套娃

而在HashMap的putVal方法中，进行了一系列判断，最后的结果是，只有在key在table数组中不存在的时候，才会返回插入的值。



```java
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。



# Java   Courrent  Interview

## 基础

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_1-并行跟并发有什么区别)1.并行跟并发有什么区别？

从操作系统的角度来看，线程是CPU分配的最小单位。

- 并行就是同一时刻，两个线程都在执行。这就要求有两个CPU去分别执行两个线程。
- 并发就是同一时刻，只有一个执行，但是一个时间段内，两个线程都执行了。并发的实现依赖于CPU切换线程，因为切换的时间特别短，所以基本对于用户是无感知的。

![并行和并发](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-1.png)并行和并发

就好像我们去食堂打饭，并行就是我们在多个窗口排队，几个阿姨同时打菜；并发就是我们挤在一个窗口，阿姨给这个打一勺，又手忙脚乱地给那个打一勺。

![并行并发和食堂打饭](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-2.png)并行并发和食堂打饭

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_2-说说什么是进程和线程)2.说说什么是进程和线程？

要说线程，必须得先说说进程。

- 进程：进程是代码在数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位。
- 线程：线程是进程的一个执行路径，一个进程中至少有一个线程，进程中的多个线程共享进程的资源。

操作系统在分配资源时是把资源分配给进程的， 但是 CPU 资源比较特殊，它是被分配到线程的，因为真正要占用CPU运行的是线程，所以也说线程是 CPU分配的基本单位。

比如在Java中，当我们启动 main 函数其实就启动了一个JVM进程，而 main 函数在的线程就是这个进程中的一个线程，也称主线程。

![程序进程线程关系](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-3.png)程序进程线程关系

一个进程中有多个线程，多个线程共用进程的堆和方法区资源，但是每个线程有自己的程序计数器和栈。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_3-说说线程有几种创建方式)3.说说线程有几种创建方式？

Java中创建线程主要有三种方式，分别为继承Thread类、实现Runnable接口、实现Callable接口。

![线程创建三种方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-4.png)线程创建三种方式

- 继承Thread类，重写run()方法，调用start()方法启动线程



```java
public class ThreadTest {

    /**
     * 继承Thread类
     */
    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("This is child thread");
        }
    }

    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
    }
}
```

- 实现 Runnable 接口，重写run()方法



```java
public class RunnableTask implements Runnable {
    public void run() {
        System.out.println("Runnable!");
    }

    public static void main(String[] args) {
        RunnableTask task = new RunnableTask();
        new Thread(task).start();
    }
}
```

上面两种都是没有返回值的，但是如果我们需要获取线程的执行结果，该怎么办呢？

- 实现Callable接口，重写call()方法，这种方式可以通过FutureTask获取任务执行的返回值



```java
public class CallerTask implements Callable<String> {
    public String call() throws Exception {
        return "Hello,i am running!";
    }

    public static void main(String[] args) {
        //创建异步任务
        FutureTask<String> task=new FutureTask<String>(new CallerTask());
        //启动线程
        new Thread(task).start();
        try {
            //等待执行完成，并获取返回结果
            String result=task.get();
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_4-为什么调用start-方法时会执行run-方法-那怎么不直接调用run-方法)4.为什么调用start()方法时会执行run()方法，那怎么不直接调用run()方法？

JVM执行start方法，会先创建一条线程，由创建出来的新线程去执行thread的run方法，这才起到多线程的效果。

![start方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-5.png)start方法

**为什么我们不能直接调用run()方法？**也很清楚， 如果直接调用Thread的run()方法，那么run方法还是运行在主线程中，相当于顺序执行，就起不到多线程的效果。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_5-线程有哪些常用的调度方法)5.线程有哪些常用的调度方法？

![线程常用调度方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-6.png)线程常用调度方法

**线程等待与通知**

在Object类中有一些函数可以用于线程的等待与通知。

- wait()：当一个线程A调用一个共享变量的 wait()方法时， 线程A会被阻塞挂起， 发生下面几种情况才会返回 ：
  - （1） 线程A调用了共享对象 notify()或者 notifyAll()方法；
  - （2）其他线程调用了线程A的 interrupt() 方法，线程A抛出InterruptedException异常返回。
- wait(long timeout) ：这个方法相比 wait() 方法多了一个超时参数，它的不同之处在于，如果线程A调用共享对象的wait(long timeout)方法后，没有在指定的 timeout ms时间内被其它线程唤醒，那么这个方法还是会因为超时而返回。
- wait(long timeout, int nanos)，其内部调用的是 wait(long timout）函数。

上面是线程等待的方法，而唤醒线程主要是下面两个方法：

- notify() : 一个线程A调用共享对象的 notify() 方法后，会唤醒一个在这个共享变量上调用 wait 系列方法后被挂起的线程。 一个共享变量上可能会有多个线程在等待，具体唤醒哪个等待的线程是随机的。
- notifyAll() ：不同于在共享变量上调用 notify() 函数会唤醒被阻塞到该共享变量上的一个线程，notifyAll()方法则会唤醒所有在该共享变量上由于调用 wait 系列方法而被挂起的线程。

Thread类也提供了一个方法用于等待的方法：

- join()：如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才

  从thread.join()返回。

**线程休眠**

- sleep(long millis) :Thread类中的静态方法，当一个执行中的线程A调用了Thread 的sleep方法后，线程A会暂时让出指定时间的执行权，但是线程A所拥有的监视器资源，比如锁还是持有不让出的。指定的睡眠时间到了后该函数会正常返回，接着参与 CPU 的调度，获取到 CPU 资源后就可以继续运行。

**让出优先权**

- yield() ：Thread类中的静态方法，当一个线程调用 yield 方法时，实际就是在暗示线程调度器当前线程请求让出自己的CPU ，但是线程调度器可以无条件忽略这个暗示。

**线程中断**

Java 中的线程中断是一种线程间的协作模式，通过设置线程的中断标志并不能直接终止该线程的执行，而是被中断的线程根据中断状态自行处理。

- void interrupt() ：中断线程，例如，当线程A运行时，线程B可以调用线程interrupt() 方法来设置线程的中断标志为true 并立即返回。设置标志仅仅是设置标志, 线程A实际并没有被中断， 会继续往下执行。
- boolean isInterrupted() 方法： 检测当前线程是否被中断。
- boolean interrupted() 方法： 检测当前线程是否被中断，与 isInterrupted 不同的是，该方法如果发现当前线程被中断，则会清除中断标志。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_6-线程有几种状态)6.线程有几种状态？

在Java中，线程共有六种状态：

| 状态         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| NEW          | 初始状态：线程被创建，但还没有调用start()方法                |
| RUNNABLE     | 运行状态：Java线程将操作系统中的就绪和运行两种状态笼统的称作“运行” |
| BLOCKED      | 阻塞状态：表示线程阻塞于锁                                   |
| WAITING      | 等待状态：表示线程进入等待状态，进入该状态表示当前线程需要等待其他线程做出一些特定动作（通知或中断） |
| TIME_WAITING | 超时等待状态：该状态不同于 WAITIND，它是可以在指定的时间自行返回的 |
| TERMINATED   | 终止状态：表示当前线程已经执行完毕                           |

线程在自身的生命周期中， 并不是固定地处于某个状态，而是随着代码的执行在不同的状态之间进行切换，Java线程状态变化如图示：

![Java线程状态变化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-7.png)Java线程状态变化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_7-什么是线程上下文切换)7.什么是线程上下文切换？

使用多线程的目的是为了充分利用CPU，但是我们知道，并发其实是一个CPU来应付多个线程。

![线程切换-2020-12-16-2107](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-8.png)线程切换-2020-12-16-2107

为了让用户感觉多个线程是在同时执行的， CPU 资源的分配采用了时间片轮转也就是给每个线程分配一个时间片，线程在时间片内占用 CPU 执行任务。当线程使用完时间片后，就会处于就绪状态并让出 CPU 让其他线程占用，这就是上下文切换。

![上下文切换时机](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-9.png)上下文切换时机

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_8-守护线程了解吗)8.守护线程了解吗？

Java中的线程分为两类，分别为 daemon 线程（守护线程）和 user 线程（用户线程）。

在JVM 启动时会调用 main 函数，main函数所在的线程就是一个用户线程。其实在 JVM 内部同时还启动了很多守护线程， 比如垃圾回收线程。

那么守护线程和用户线程有什么区别呢？区别之一是当最后一个非守护线程束时， JVM会正常退出，而不管当前是否存在守护线程，也就是说守护线程是否结束并不影响 JVM退出。换而言之，只要有一个用户线程还没结束，正常情况下JVM就不会退出。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_9-线程间有哪些通信方式)9.线程间有哪些通信方式？

![线程间通信方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-10.png)线程间通信方式

- **volatile和synchronized关键字**

关键字volatile可以用来修饰字段（成员变量），就是告知程序任何对该变量的访问均需要从共享内存中获取，而对它的改变必须同步刷新回共享内存，它能保证所有线程对变量访问的可见性。

关键字synchronized可以修饰方法或者以同步块的形式来进行使用，它主要确保多个线程在同一个时刻，只能有一个线程处于方法或者同步块中，它保证了线程对变量访问的可见性和排他性。

- **等待/通知机制**

可以通过Java内置的等待/通知机制（wait()/notify()）实现一个线程修改一个对象的值，而另一个线程感知到了变化，然后进行相应的操作。

- **管道输入/输出流**

管道输入/输出流和普通的文件输入/输出流或者网络输入/输出流不同之处在于，它主要用于线程之间的数据传输，而传输的媒介为内存。

管道输入/输出流主要包括了如下4种具体实现：PipedOutputStream、PipedInputStream、 PipedReader和PipedWriter，前两种面向字节，而后两种面向字符。

- **使用Thread.join()**

如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才从thread.join()返回。。线程Thread除了提供join()方法之外，还提供了join(long millis)和join(long millis,int nanos)两个具备超时特性的方法。

- **使用ThreadLocal**

ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的存储结构。这个结构被附带在线程上，也就是说一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值。

可以通过set(T)方法来设置一个值，在当前线程下再通过get()方法获取到原先设置的值。

> 关于多线程，其实很大概率还会出一些笔试题，比如交替打印、银行转账、生产消费模型等等，后面老三会单独出一期来盘点一下常见的多线程笔试题。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#threadlocal)ThreadLocal

ThreadLocal其实应用场景不是很多，但却是被炸了千百遍的面试老油条，涉及到多线程、数据结构、JVM，可问的点比较多，一定要拿下。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_10-threadlocal是什么)10.ThreadLocal是什么？

ThreadLocal，也就是线程本地变量。如果你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个本地拷贝，多个线程操作这个变量的时候，实际是操作自己本地内存里面的变量，从而起到线程隔离的作用，避免了线程安全问题。

![ThreadLocal线程副本](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-11.png)ThreadLocal线程副本

- 创建

创建了一个ThreadLoca变量localVariable，任何一个线程都能并发访问localVariable。



```java
//创建一个ThreadLocal变量
public static ThreadLocal<String> localVariable = new ThreadLocal<>();
```

- 写入

线程可以在任何地方使用localVariable，写入变量。



```java
localVariable.set("鄙人三某”);
```

- 读取

线程在任何地方读取的都是它写入的变量。



```java
localVariable.get();
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_11-你在工作中用到过threadlocal吗)11.你在工作中用到过ThreadLocal吗？

有用到过的，用来做用户信息上下文的存储。

我们的系统应用是一个典型的MVC架构，登录后的用户每次访问接口，都会在请求头中携带一个token，在控制层可以根据这个token，解析出用户的基本信息。那么问题来了，假如在服务层和持久层都要用到用户信息，比如rpc调用、更新用户获取等等，那应该怎么办呢？

一种办法是显式定义用户相关的参数，比如账号、用户名……这样一来，我们可能需要大面积地修改代码，多少有点瓜皮，那该怎么办呢？

这时候我们就可以用到ThreadLocal，在控制层拦截请求把用户信息存入ThreadLocal，这样我们在任何一个地方，都可以取出ThreadLocal中存的用户数据。

![ThreadLoca存放用户上下文](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-12.png)ThreadLoca存放用户上下文

很多其它场景的cookie、session等等数据隔离也都可以通过ThreadLocal去实现。

我们常用的数据库连接池也用到了ThreadLocal：

- 数据库连接池的连接交给ThreadLoca进行管理，保证当前线程的操作都是同一个Connnection。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_12-threadlocal怎么实现的呢)12.ThreadLocal怎么实现的呢？

我们看一下ThreadLocal的set(T)方法，发现先获取到当前线程，再获取`ThreadLocalMap`，然后把元素存到这个map中。



```java
    public void set(T value) {
        //获取当前线程
        Thread t = Thread.currentThread();
        //获取ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        //讲当前元素存入map
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

ThreadLocal实现的秘密都在这个`ThreadLocalMap`了，可以Thread类中定义了一个类型为`ThreadLocal.ThreadLocalMap`的成员变量`threadLocals`。



```java
public class Thread implements Runnable {
   //ThreadLocal.ThreadLocalMap是Thread的属性
   ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

ThreadLocalMap既然被称为Map，那么毫无疑问它是<key,value>型的数据结构。我们都知道map的本质是一个个<key,value>形式的节点组成的数组，那ThreadLocalMap的节点是什么样的呢？



```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    //节点类
    Entry(ThreadLocal<?> k, Object v) {
        //key赋值
        super(k);
        //value赋值
        value = v;
    }
}
```

这里的节点，key可以简单低视作ThreadLocal，value为代码中放入的值，当然实际上key并不是ThreadLocal本身，而是它的一个**弱引用**，可以看到Entry的key继承了 WeakReference（弱引用），再来看一下key怎么赋值的：



```java
public WeakReference(T referent) {
    super(referent);
}
```

key的赋值，使用的是WeakReference的赋值。

![ThreadLoca结构图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-13.png)ThreadLoca结构图

> 所以，怎么回答ThreadLocal原理？要答出这几个点：

- Thread类有一个类型为ThreadLocal.ThreadLocalMap的实例变量threadLocals，每个线程都有一个属于自己的ThreadLocalMap。
- ThreadLocalMap内部维护着Entry数组，每个Entry代表一个完整的对象，key是ThreadLocal的弱引用，value是ThreadLocal的泛型值。
- 每个线程在往ThreadLocal里设置值的时候，都是往自己的ThreadLocalMap里存，读也是以某个ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离。
- ThreadLocal本身不存储值，它只是作为一个key来让线程往ThreadLocalMap里存取值。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_13-threadlocal-内存泄露是怎么回事)13.ThreadLocal 内存泄露是怎么回事？

我们先来分析一下使用ThreadLocal时的内存，我们都知道，在JVM中，栈内存线程私有，存储了对象的引用，堆内存线程共享，存储了对象实例。

所以呢，栈中存储了ThreadLocal、Thread的引用，堆中存储了它们的具体实例。

![ThreadLocal内存分配](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-14.png)ThreadLocal内存分配

ThreadLocalMap中使用的 key 为 ThreadLocal 的弱引用。

> “弱引用：只要垃圾回收机制一运行，不管JVM的内存空间是否充足，都会回收该对象占用的内存。”

那么现在问题就来了，弱引用很容易被回收，如果ThreadLocal（ThreadLocalMap的Key）被垃圾回收器回收了，但是ThreadLocalMap生命周期和Thread是一样的，它这时候如果不被回收，就会出现这种情况：ThreadLocalMap的key没了，value还在，这就会**造成了内存泄漏问题**。

> 那怎么解决内存泄漏问题呢？

很简单，使用完ThreadLocal后，及时调用remove()方法释放内存空间。



```java
ThreadLocal<String> localVariable = new ThreadLocal();
try {
    localVariable.set("鄙人三某”);
    ……
} finally {
    localVariable.remove();
}
```

> 那为什么key还要设计成弱引用？

key设计成弱引用同样是为了防止内存泄漏。

假如key被设计成强引用，如果ThreadLocal Reference被销毁，此时它指向ThreadLoca的强引用就没有了，但是此时key还强引用指向ThreadLoca，就会导致ThreadLocal不能被回收，这时候就发生了内存泄漏的问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_14-threadlocalmap的结构了解吗)14.ThreadLocalMap的结构了解吗？

ThreadLocalMap虽然被叫做Map，其实它是没有实现Map接口的，但是结构还是和HashMap比较类似的，主要关注的是两个要素：`元素数组`和`散列方法`。

![ThreadLocalMap结构示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-15.png)ThreadLocalMap结构示意图

- 元素数组

  一个table数组，存储Entry类型的元素，Entry是ThreaLocal弱引用作为key，Object作为value的结构。



```java
 private Entry[] table;
```

- 散列方法

  散列方法就是怎么把对应的key映射到table数组的相应下标，ThreadLocalMap用的是哈希取余法，取出key的threadLocalHashCode，然后和table数组长度减一&运算（相当于取余）。



```java
int i = key.threadLocalHashCode & (table.length - 1);
```

这里的threadLocalHashCode计算有点东西，每创建一个ThreadLocal对象，它就会新增`0x61c88647`，这个值很特殊，它是**斐波那契数** 也叫 **黄金分割数**。`hash`增量为 这个数字，带来的好处就是 `hash` **分布非常均匀**。



```java
    private static final int HASH_INCREMENT = 0x61c88647;
    
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_15-threadlocalmap怎么解决hash冲突的)15.ThreadLocalMap怎么解决Hash冲突的？

我们可能都知道HashMap使用了链表来解决冲突，也就是所谓的链地址法。

ThreadLocalMap没有使用链表，自然也不是用链地址法来解决冲突了，它用的是另外一种方式——**开放定址法**。开放定址法是什么意思呢？简单来说，就是这个坑被人占了，那就接着去找空着的坑。

![ThreadLocalMap解决冲突](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-16.png)ThreadLocalMap解决冲突

如上图所示，如果我们插入一个value=27的数据，通过 hash计算后应该落入第 4 个槽位中，而槽位 4 已经有了 Entry数据，而且Entry数据的key和当前不相等。此时就会线性向后查找，一直找到 Entry为 null的槽位才会停止查找，把元素放到空的槽中。

在get的时候，也会根据ThreadLocal对象的hash值，定位到table中的位置，然后判断该槽位Entry对象中的key是否和get的key一致，如果不一致，就判断下一个位置。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_16-threadlocalmap扩容机制了解吗)16.ThreadLocalMap扩容机制了解吗？

在ThreadLocalMap.set()方法的最后，如果执行完启发式清理工作后，未清理到任何数据，且当前散列数组中`Entry`的数量已经达到了列表的扩容阈值`(len*2/3)`，就开始执行`rehash()`逻辑：



```java
if (!cleanSomeSlots(i, sz) && sz >= threshold)
    rehash();
```

再着看rehash()具体实现：这里会先去清理过期的Entry，然后还要根据条件判断`size >= threshold - threshold / 4` 也就是`size >= threshold* 3/4`来决定是否需要扩容。



```java
private void rehash() {
    //清理过期Entry
    expungeStaleEntries();

    //扩容
    if (size >= threshold - threshold / 4)
        resize();
}

//清理过期Entry
private void expungeStaleEntries() {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}
```

接着看看具体的`resize()`方法，扩容后的`newTab`的大小为老数组的两倍，然后遍历老的table数组，散列方法重新计算位置，开放地址解决冲突，然后放到新的`newTab`，遍历完成之后，`oldTab`中所有的`entry`数据都已经放入到`newTab`中了，然后table引用指向`newTab`

![ThreadLocalMap扩容](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-17.png)ThreadLocalMap扩容

具体代码：

![ThreadLocalMap resize](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-18.png)ThreadLocalMap resize

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_17-父子线程怎么共享数据)17.父子线程怎么共享数据？

父线程能用ThreadLocal来给子线程传值吗？毫无疑问，不能。那该怎么办？

这时候可以用到另外一个类——`InheritableThreadLocal `。

使用起来很简单，在主线程的InheritableThreadLocal实例设置值，在子线程中就可以拿到了。



```java
public class InheritableThreadLocalTest {
    
    public static void main(String[] args) {
        final ThreadLocal threadLocal = new InheritableThreadLocal();
        // 主线程
        threadLocal.set("不擅技术");
        //子线程
        Thread t = new Thread() {
            @Override
            public void run() {
                super.run();
                System.out.println("鄙人三某 ，" + threadLocal.get());
            }
        };
        t.start();
    }
}
```

> 那原理是什么呢？

原理很简单，在Thread类里还有另外一个变量：



```java
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

在Thread.init的时候，如果父线程的`inheritableThreadLocals`不为空，就把它赋给当前线程（子线程）的`inheritableThreadLocals `。



```java
if (inheritThreadLocals && parent.inheritableThreadLocals != null)
    this.inheritableThreadLocals =
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
```

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#java内存模型)Java内存模型

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_18-说一下你对java内存模型-jmm-的理解)18.说一下你对Java内存模型（JMM）的理解？

Java内存模型（Java Memory Model，JMM），是一种抽象的模型，被定义出来屏蔽各种硬件和操作系统的内存访问差异。

JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在`主内存`（Main Memory）中，每个线程都有一个私有的`本地内存`（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。

Java内存模型的抽象图：

![Java内存模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-19.png)Java内存模型

本地内存是JMM的 一个抽象概念，并不真实存在。它其实涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

![实际线程工作模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-20.png)实际线程工作模型

图里面的是一个双核 CPU 系统架构 ，每个核有自己的控制器和运算器，其中控制器包含一组寄存器和操作控制器，运算器执行算术逻辅运算。每个核都有自己的一级缓存，在有些架构里面还有一个所有 CPU 共享的二级缓存。 那么 Java 内存模型里面的工作内存，就对应这里的 Ll 缓存或者 L2 缓存或者 CPU 寄存器。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_19-说说你对原子性、可见性、有序性的理解)19.说说你对原子性、可见性、有序性的理解？

原子性、有序性、可见性是并发编程中非常重要的基础概念，JMM的很多技术都是围绕着这三大特性展开。

- **原子性**：原子性指的是一个操作是不可分割、不可中断的，要么全部执行并且执行的过程不会被任何因素打断，要么就全不执行。
- **可见性**：可见性指的是一个线程修改了某一个共享变量的值时，其它线程能够立即知道这个修改。
- **有序性**：有序性指的是对于一个线程的执行代码，从前往后依次执行，单线程下可以认为程序是有序的，但是并发时有可能会发生指令重排。

> 分析下面几行代码的原子性？



```java
int i = 2;
int j = i;
i++;
i = i + 1;
```

- 第1句是基本类型赋值，是原子性操作。
- 第2句先读i的值，再赋值到j，两步操作，不能保证原子性。
- 第3和第4句其实是等效的，先读取i的值，再+1，最后赋值到i，三步操作了，不能保证原子性。

> 原子性、可见性、有序性都应该怎么保证呢？

- 原子性：JMM只能保证基本的原子性，如果要保证一个代码块的原子性，需要使用`synchronized `。
- 可见性：Java是利用`volatile`关键字来保证可见性的，除此之外，`final`和`synchronized`也能保证可见性。
- 有序性：`synchronized`或者`volatile`都可以保证多线程之间操作的有序性。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_20-那说说什么是指令重排)20.那说说什么是指令重排？

在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分3种类型。

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应 机器指令的执行顺序。
3. 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从Java源代码到最终实际执行的指令序列，会分别经历下面3种重排序，如图：

![多级指令重排](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-21.png)多级指令重排

我们比较熟悉的双重校验单例模式就是一个经典的指令重排的例子，`Singleton instance=new Singleton()；`对应的JVM指令分为三步：分配内存空间-->初始化对象--->对象指向分配的内存空间，但是经过了编译器的指令重排序，第二步和第三步就可能会重排序。

![双重校验单例模式异常情形](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-22.png)双重校验单例模式异常情形

JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_21-指令重排有限制吗-happens-before了解吗)21.指令重排有限制吗？happens-before了解吗？

指令重排也是有一些限制的，有两个规则`happens-before`和`as-if-serial`来约束。

happens-before的定义：

- 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
- 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照 happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序并不非法

happens-before和我们息息相关的有六大规则：

![happens-before六大规则](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-23.png)happens-before六大规则

- **程序顺序规则**：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- **监视器锁规则**：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- **volatile变量规则**：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- **传递性**：如果A happens-before B，且B happens-before C，那么A happens-before C。
- **start()规则**：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的 ThreadB.start()操作happens-before于线程B中的任意操作。
- **join()规则**：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作 happens-before于线程A从ThreadB.join()操作成功返回。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_22-as-if-serial又是什么-单线程的程序一定是顺序的吗)22.as-if-serial又是什么？单线程的程序一定是顺序的吗？

as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），**单线程程序的执行结果不能被改变**。编译器、runtime和处理器都必须遵守as-if-serial语义。

为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。为了具体说明，请看下面计算圆面积的代码示例。



```java
double pi = 3.14;   // A
double r = 1.0;   // B 
double area = pi * r * r;   // C
```

上面3个操作的数据依赖关系：

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-24.png)

A和C之间存在数据依赖关系，同时B和C之间也存在数据依赖关系。因此在最终执行的指令序列中，C不能被重排序到A和B的前面（C排到A和B的前面，程序的结果将会被改变）。但A和B之间没有数据依赖关系，编译器和处理器可以重排序A和B之间的执行顺序。

所以最终，程序可能会有两种执行顺序：

![两种执行结果](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-25.png)两种执行结果

as-if-serial语义把单线程程序保护了起来，遵守as-if-serial语义的编译器、runtime和处理器共同编织了这么一个“楚门的世界”：单线程程序是按程序的“顺序”来执行的。as- if-serial语义使单线程情况下，我们不需要担心重排序的问题，可见性的问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_23-volatile实现原理了解吗)23.volatile实现原理了解吗？

volatile有两个作用，保证**可见性**和**有序性**。

> volatile怎么保证可见性的呢？

相比synchronized的加锁方式来解决共享变量的内存可见性问题，volatile就是更轻量的选择，它没有上下文切换的额外开销成本。

volatile可以确保对某个变量的更新对其他线程马上可见，一个变量被声明为volatile 时，线程在写入变量时不会把值缓存在寄存器或者其他地方，而是会把值刷新回主内存 当其它线程读取该共享变量 ，会从主内存重新获取最新值，而不是使用当前线程的本地内存中的值。

例如，我们声明一个 volatile 变量 volatile int x = 0，线程A修改x=1，修改完之后就会把新的值刷新回主内存，线程B读取x的时候，就会清空本地内存变量，然后再从主内存获取最新值。

![volatile内存可见性](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-26.png)volatile内存可见性

> volatile怎么保证有序性的呢？

重排序可以分为编译器重排序和处理器重排序，valatile保证有序性，就是通过分别限制这两种类型的重排序。

![volatile重排序规则表](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-27.png)volatile重排序规则表

为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

1. 在每个volatile写操作的前面插入一个`StoreStore`屏障
2. 在每个volatile写操作的后面插入一个`StoreLoad`屏障
3. 在每个volatile读操作的后面插入一个`LoadLoad`屏障
4. 在每个volatile读操作的后面插入一个`LoadStore`屏障

![volatile写插入内存屏障后生成的指令序列示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-28.png)volatile写插入内存屏障后生成的指令序列示意图

![volatile写插入内存屏障后生成的指令序列示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-29.png)volatile写插入内存屏障后生成的指令序列示意图

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#锁)锁

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_24-synchronized用过吗-怎么使用)24.synchronized用过吗？怎么使用？

synchronized经常用的，用来保证代码的原子性。

synchronized主要有三种用法：

- **修饰实例方法:** 作用于当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁**



```java
synchronized void method() {
  //业务代码
}
```

- **修饰静态方法**：也就是给当前类加锁，会作⽤于类的所有对象实例 ，进⼊同步代码前要获得当前 class 的锁。因为静态成员不属于任何⼀个实例对象，是类成员（ static 表明这是该类的⼀个静态资源，不管 new 了多少个对象，只有⼀份）。

  如果⼀个线程 A 调⽤⼀个实例对象的⾮静态 synchronized ⽅法，⽽线程 B 需要调⽤这个实例对象所属类的静态 synchronized ⽅法，是允许的，不会发⽣互斥现象，因为访问静态 synchronized ⽅法占⽤的锁是当前类的锁，⽽访问⾮静态 synchronized ⽅法占⽤的锁是当前实例对象锁。



```java
synchronized void staic method() {
 //业务代码
}
```

- **修饰代码块** ：指定加锁对象，对给定对象/类加锁。 synchronized(this|object) 表示进⼊同步代码库前要获得给定对象的锁。 synchronized(类.class) 表示进⼊同步代码前要获得 当前 **class** 的锁



```java
synchronized(this) {
 //业务代码
}
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_25-synchronized的实现原理)25.synchronized的实现原理？

> synchronized是怎么加锁的呢？

我们使用synchronized的时候，发现不用自己去lock和unlock，是因为JVM帮我们把这个事情做了。

1. synchronized修饰代码块时，JVM采用`monitorenter`、`monitorexit`两个指令来实现同步，`monitorenter` 指令指向同步代码块的开始位置， `monitorexit` 指令则指向同步代码块的结束位置。

   反编译一段synchronized修饰代码块代码，`javap -c -s -v -l SynchronizedDemo.class`，可以看到相应的字节码指令。

![monitorenter和monitorexit](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-30.png)monitorenter和monitorexit

1. synchronized修饰同步方法时，JVM采用`ACC_SYNCHRONIZED`标记符来实现同步，这个标识指明了该方法是一个同步方法。

同样可以写段代码反编译看一下。

![synchronized修饰同步方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-31.png)synchronized修饰同步方法

> synchronized锁住的是什么呢？

monitorenter、monitorexit或者ACC_SYNCHRONIZED都是**基于Monitor实现**的。

实例对象结构里有对象头，对象头里面有一块结构叫Mark Word，Mark Word指针指向了**monitor**。

所谓的Monitor其实是一种**同步工具**，也可以说是一种**同步机制**。在Java虚拟机（HotSpot）中，Monitor是由**ObjectMonitor实现**的，可以叫做内部锁，或者Monitor锁。

ObjectMonitor的工作原理：

- ObjectMonitor有两个队列：_WaitSet、_EntryList，用来保存ObjectWaiter 对象列表。
- _owner，获取 Monitor 对象的线程进入 _owner 区时， _count + 1。如果线程调用了 wait() 方法，此时会释放 Monitor 对象， _owner 恢复为空， _count - 1。同时该等待线程进入 _WaitSet 中，等待被唤醒。



```java
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录线程获取锁的次数
    _waiters      = 0,
    _recursions   = 0;  //锁的重入次数
    _object       = NULL;
    _owner        = NULL;  // 指向持有ObjectMonitor对象的线程
    _WaitSet      = NULL;  // 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;  // 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

可以类比一个去医院就诊的例子[18]：

- 首先，患者在**门诊大厅**前台或自助挂号机**进行挂号**；
- 随后，挂号结束后患者找到对应的**诊室就诊**：
  - 诊室每次只能有一个患者就诊；
  - 如果此时诊室空闲，直接进入就诊；
  - 如果此时诊室内有其它患者就诊，那么当前患者进入**候诊室**，等待叫号；
- 就诊结束后，**走出就诊室**，候诊室的**下一位候诊患者**进入就诊室。

![就诊-图片来源参考[18]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-32.png)就诊-图片来源参考[18]

这个过程就和Monitor机制比较相似：

- **门诊大厅**：所有待进入的线程都必须先在**入口Entry Set**挂号才有资格；
- **就诊室**：就诊室**_Owner**里里只能有一个线程就诊，就诊完线程就自行离开
- **候诊室**：就诊室繁忙时，进入**等待区（Wait Set）**，就诊室空闲的时候就从**等待区（Wait Set）**叫新的线程

![Java Montior机制](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-33.png)Java Montior机制

所以我们就知道了，同步是锁住的什么东西：

- monitorenter，在判断拥有同步标识 ACC_SYNCHRONIZED 抢先进入此方法的线程会优先拥有 Monitor 的 owner ，此时计数器 +1。
- monitorexit，当执行完退出后，计数器 -1，归 0 后被其他进入的线程获得。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_26-除了原子性-synchronized可见性-有序性-可重入性怎么实现)26.除了原子性，synchronized可见性，有序性，可重入性怎么实现？

> synchronized怎么保证可见性？

- 线程加锁前，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值。
- 线程加锁后，其它线程无法获取主内存中的共享变量。
- 线程解锁前，必须把共享变量的最新值刷新到主内存中。

> synchronized怎么保证有序性？

synchronized同步的代码块，具有排他性，一次只能被一个线程拥有，所以synchronized保证同一时刻，代码是单线程执行的。

因为as-if-serial语义的存在，单线程的程序能保证最终结果是有序的，但是不保证不会指令重排。

所以synchronized保证的有序是执行结果的有序性，而不是防止指令重排的有序性。

> synchronized怎么实现可重入的呢？

synchronized 是可重入锁，也就是说，允许一个线程二次请求自己持有对象锁的临界资源，这种情况称为可重入锁。

synchronized 锁对象的时候有个计数器，他会记录下线程获取锁的次数，在执行完对应的代码块之后，计数器就会-1，直到计数器清零，就释放锁了。

之所以，是可重入的。是因为 synchronized 锁对象有个计数器，会随着线程获取锁后 +1 计数，当线程执行完毕后 -1，直到清零释放锁。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_27-锁升级-synchronized优化了解吗)27.锁升级？synchronized优化了解吗？

了解锁升级，得先知道，不同锁的状态是什么样的。这个状态指的是什么呢？

Java对象头里，有一块结构，叫`Mark Word`标记字段，这块结构会随着锁的状态变化而变化。

64 位虚拟机 Mark Word 是 64bit，我们来看看它的状态变化：

![Mark Word变化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-34.png)Mark Word变化

Mark Word存储对象自身的运行数据，如**哈希码、GC分代年龄、锁状态标志、偏向时间戳（Epoch）** 等。

> synchronized做了哪些优化？

在JDK1.6之前，synchronized的实现直接调用ObjectMonitor的enter和exit，这种锁被称之为**重量级锁**。从JDK6开始，HotSpot虚拟机开发团队对Java中的锁进行优化，如增加了适应性自旋、锁消除、锁粗化、轻量级锁和偏向锁等优化策略，提升了synchronized的性能。

- 偏向锁：在无竞争的情况下，只是在Mark Word里存储当前线程指针，CAS操作都不做。
- 轻量级锁：在没有多线程竞争时，相对重量级锁，减少操作系统互斥量带来的性能消耗。但是，如果存在锁竞争，除了互斥量本身开销，还额外有CAS操作的开销。
- 自旋锁：减少不必要的CPU上下文切换。在轻量级锁升级为重量级锁时，就使用了自旋加锁的方式
- 锁粗化：将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。
- 锁消除：虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。

> 锁升级的过程是什么样的？

锁升级方向：无锁-->偏向锁---> 轻量级锁---->重量级锁，这个方向基本上是不可逆的。

![锁升级方向](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-35.png)锁升级方向

我们看一下升级的过程：

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#偏向锁)偏向锁：

**偏向锁的获取：**

1. 判断是否为可偏向状态--MarkWord中锁标志是否为‘01’，是否偏向锁是否为‘1’
2. 如果是可偏向状态，则查看线程ID是否为当前线程，如果是，则进入步骤'5'，否则进入步骤‘3’
3. 通过CAS操作竞争锁，如果竞争成功，则将MarkWord中线程ID设置为当前线程ID，然后执行‘5’；竞争失败，则执行‘4’
4. CAS获取偏向锁失败表示有竞争。当达到safepoint时获得偏向锁的线程被挂起，**偏向锁升级为轻量级锁**，然后被阻塞在安全点的线程继续往下执行同步代码块
5. 执行同步代码

**偏向锁的撤销：**

1. 偏向锁不会主动释放(撤销)，只有遇到其他线程竞争时才会执行撤销，由于撤销需要知道当前持有该偏向锁的线程栈状态，因此要等到safepoint时执行，此时持有该偏向锁的线程（T）有‘2’，‘3’两种情况；
2. 撤销----T线程已经退出同步代码块，或者已经不再存活，则直接撤销偏向锁，变成无锁状态----该状态达到阈值20则执行批量重偏向
3. 升级----T线程还在同步代码块中，则将T线程的偏向锁**升级为轻量级锁**，当前线程执行轻量级锁状态下的锁获取步骤----该状态达到阈值40则执行批量撤销

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#轻量级锁)轻量级锁：

**轻量级锁的获取：**

1. 进行加锁操作时，jvm会判断是否已经时重量级锁，如果不是，则会在当前线程栈帧中划出一块空间，作为该锁的锁记录，并且将锁对象MarkWord复制到该锁记录中
2. 复制成功之后，jvm使用CAS操作将对象头MarkWord更新为指向锁记录的指针，并将锁记录里的owner指针指向对象头的MarkWord。如果成功，则执行‘3’，否则执行‘4’
3. 更新成功，则当前线程持有该对象锁，并且对象MarkWord锁标志设置为‘00’，即表示此对象处于轻量级锁状态
4. 更新失败，jvm先检查对象MarkWord是否指向当前线程栈帧中的锁记录，如果是则执行‘5’，否则执行‘4’
5. 表示锁重入；然后当前线程栈帧中增加一个锁记录第一部分（Displaced Mark Word）为null，并指向Mark Word的锁对象，起到一个重入计数器的作用。
6. 表示该锁对象已经被其他线程抢占，则进行**自旋等待**（默认10次），等待次数达到阈值仍未获取到锁，则**升级为重量级锁**

大体上省简的升级过程：

![锁升级简略过程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-36.png)锁升级简略过程

完整的升级过程：

![synchronized 锁升级过程-来源参考[14]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-37.png)synchronized 锁升级过程-来源参考[14]

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_28-说说synchronized和reentrantlock的区别)28.说说synchronized和ReentrantLock的区别？

可以从锁的实现、功能特点、性能等几个维度去回答这个问题：

- **锁的实现：** synchronized是Java语言的关键字，基于JVM实现。而ReentrantLock是基于JDK的API层面实现的（一般是lock()和unlock()方法配合try/finally 语句块来完成。）

- **性能：** 在JDK1.6锁优化以前，synchronized的性能比ReenTrantLock差很多。但是JDK6开始，增加了适应性自旋、锁消除等，两者性能就差不多了。

- 功能特点：

   

  ReentrantLock 比 synchronized 增加了一些高级功能，如等待可中断、可实现公平锁、可实现选择性通知。

  - ReentrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制
  - ReentrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。
  - synchronized与wait()和notify()/notifyAll()方法结合实现等待/通知机制，ReentrantLock类借助Condition接口与newCondition()方法实现。
  - ReentrantLock需要手工声明来加锁和释放锁，一般跟finally配合释放锁。而synchronized不用手动释放锁。

下面的表格列出出了两种锁之间的区别：

![synchronized和ReentrantLock的区别](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-38.png)synchronized和ReentrantLock的区别

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_29-aqs了解多少)29.AQS了解多少？

AbstractQueuedSynchronizer 抽象同步队列，简称 AQS ，它是Java并发包的根基，并发包中的锁就是基于AQS实现的。

- AQS是基于一个FIFO的双向队列，其内部定义了一个节点类Node，Node 节点内部的 SHARED 用来标记该线程是获取共享资源时被阻挂起后放入AQS 队列的， EXCLUSIVE 用来标记线程是 取独占资源时被挂起后放入AQS 队列
- AQS 使用一个 volatile 修饰的 int 类型的成员变量 state 来表示同步状态，修改同步状态成功即为获得锁，volatile 保证了变量在多线程之间的可见性，修改 State 值时通过 CAS 机制来保证修改的原子性
- 获取state的方式分为两种，独占方式和共享方式，一个线程使用独占方式获取了资源，其它线程就会在获取失败后被阻塞。一个线程使用共享方式获取了资源，另外一个线程还可以通过CAS的方式进行获取。
- 如果共享资源被占用，需要一定的阻塞等待唤醒机制来保证锁的分配，AQS 中会将竞争共享资源失败的线程添加到一个变体的 CLH 队列中。

![AQS抽象队列同步器](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-39.png)AQS抽象队列同步器

![CLH队列](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-40.png)CLH队列

AQS 中的队列是 CLH 变体的虚拟双向队列，通过将每条请求共享资源的线程封装成一个节点来实现锁的分配：

![AQS变种CLH队列](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-41.png)AQS变种CLH队列

AQS 中的 CLH 变体等待队列拥有以下特性：

- AQS 中队列是个双向链表，也是 FIFO 先进先出的特性
- 通过 Head、Tail 头尾两个节点来组成队列结构，通过 volatile 修饰保证可见性
- Head 指向节点为已获得锁的节点，是一个虚拟节点，节点本身不持有具体线程
- 获取不到同步状态，会将节点进行自旋获取锁，自旋一定次数失败后会将线程阻塞，相对于 CLH 队列性能较好

ps:AQS源码里面有很多细节可问，建议有时间好好看看AQS源码。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_30-reentrantlock实现原理)30.**ReentrantLock**实现原理？

ReentrantLock 是可重入的独占锁，只能有一个线程可以获取该锁，其它获取该锁的线程会被阻塞而被放入该锁的阻塞队列里面。

看看ReentrantLock的加锁操作：



```java
// 创建非公平锁
ReentrantLock lock = new ReentrantLock();
// 获取锁操作
lock.lock();
try {
    // 执行代码逻辑
} catch (Exception ex) {
    // ...
} finally {
    // 解锁操作
    lock.unlock();
}
```

`new ReentrantLock() `构造函数默认创建的是非公平锁 NonfairSync。

**公平锁 FairSync**

1. 公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁
2. 公平锁的优点是等待锁的线程不会饿死。缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU 唤醒阻塞线程的开销比非公平锁大

**非公平锁 NonfairSync**

- 非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁
- 非公平锁的优点是可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU 不必唤醒所有线程。缺点是处于等待队列中的线程可能会饿死，或者等很久才会获得锁

默认创建的对象lock()的时候：

- 如果锁当前没有被其它线程占用，并且当前线程之前没有获取过该锁，则当前线程会获取到该锁，然后设置当前锁的拥有者为当前线程，并设置 AQS 的状态值为1 ，然后直接返回。如果当前线程之前己经获取过该锁，则这次只是简单地把 AQS 的状态值加1后返回。
- 如果该锁己经被其他线程持有，非公平锁会尝试去获取锁，获取失败的话，则调用该方法线程会被放入 AQS 队列阻塞挂起。

![ReentrantLock 非公平锁加锁流程简图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-42.png)ReentrantLock 非公平锁加锁流程简图

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_31-reentrantlock怎么实现公平锁的)31.ReentrantLock怎么实现公平锁的？

`new ReentrantLock() `构造函数默认创建的是非公平锁 NonfairSync



```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

同时也可以在创建锁构造函数中传入具体参数创建公平锁 FairSync



```java
ReentrantLock lock = new ReentrantLock(true);
--- ReentrantLock
// true 代表公平锁，false 代表非公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

FairSync、NonfairSync 代表公平锁和非公平锁，两者都是 ReentrantLock 静态内部类，只不过实现不同锁语义。

**非公平锁和公平锁的两处不同：**

1. 非公平锁在调用 lock 后，首先就会调用 CAS 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。
2. 非公平锁在 CAS 失败后，和公平锁一样都会进入到 tryAcquire 方法，在 tryAcquire 方法中，如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，乖乖排到后面。

![公平锁tryAcquire](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-43.png)公平锁tryAcquire

相对来说，非公平锁会有更好的性能，因为它的吞吐量比较大。当然，非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_32-cas呢-cas了解多少)32.CAS呢？CAS了解多少？

CAS叫做CompareAndSwap，⽐较并交换，主要是通过处理器的指令来保证操作的原⼦性的。

CAS 指令包含 3 个参数：共享变量的内存地址 A、预期的值 B 和共享变量的新值 C。

只有当内存中地址 A 处的值等于 B 时，才能将内存中地址 A 处的值更新为新值 C。作为一条 CPU 指令，CAS 指令本身是能够保证原子性的 。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_33-cas-有什么问题-如何解决)33.CAS 有什么问题？如何解决？

CAS的经典三大问题：

![CAS三大问题](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-44.png)CAS三大问题

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#aba-问题)ABA 问题

并发环境下，假设初始条件是A，去修改数据时，发现是A就会执行修改。但是看到的虽然是A，中间可能发生了A变B，B又变回A的情况。此时A已经非彼A，数据即使成功修改，也可能有问题。

> 怎么解决ABA问题？

- 加版本号

每次修改变量，都在这个变量的版本号上加1，这样，刚刚A->B->A，虽然A的值没变，但是它的版本号已经变了，再判断版本号就会发现此时的A已经被改过了。参考乐观锁的版本号，这种做法可以给数据带上了一种实效性的检验。

Java提供了AtomicStampReference类，它的compareAndSet方法首先检查当前的对象引用值是否等于预期引用，并且当前印戳（Stamp）标志是否等于预期标志，如果全部相等，则以原子方式将引用值和印戳标志的值更新为给定的更新值。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#循环性能开销)循环性能开销

自旋CAS，如果一直循环执行，一直不成功，会给CPU带来非常大的执行开销。

> 怎么解决循环性能开销问题？

在Java中，很多使用自旋CAS的地方，会有一个自旋次数的限制，超过一定次数，就停止自旋。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#只能保证一个变量的原子操作)只能保证一个变量的原子操作

CAS 保证的是对一个变量执行操作的原子性，如果对多个变量操作时，CAS 目前无法直接保证操作的原子性的。

> 怎么解决只能保证一个变量的原子操作问题？

- 可以考虑改用锁来保证操作的原子性
- 可以考虑合并多个变量，将多个变量封装成一个对象，通过AtomicReference来保证原子性。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_34-java有哪些保证原子性的方法-如何保证多线程下i-结果正确)34.Java有哪些保证原子性的方法？如何保证多线程下i++ 结果正确？

![Java保证原子性方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-45.png)Java保证原子性方法

- 使用循环原子类，例如AtomicInteger，实现i++原子操作
- 使用juc包下的锁，如ReentrantLock ，对i++操作加锁lock.lock()来实现原子性
- 使用synchronized，对i++操作加锁

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_35-原子操作类了解多少)35.原子操作类了解多少？

当程序更新一个变量时，如果多线程同时更新这个变量，可能得到期望之外的值，比如变量i=1，A线程更新i+1，B线程也更新i+1，经过两个线程操作之后可能i不等于3，而是等于2。因为A和B线程在更新变量i的时候拿到的i都是1，这就是线程不安全的更新操作，一般我们会使用synchronized来解决这个问题，synchronized会保证多线程不会同时更新变量i。

其实除此之外，还有更轻量级的选择，Java从JDK 1.5开始提供了java.util.concurrent.atomic包，这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。

因为变量的类型有很多种，所以在Atomic包里一共提供了13个类，属于4种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段）。

![原子操作类](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-46.png)原子操作类

Atomic包里的类基本都是使用Unsafe实现的包装类。

使用原子的方式更新基本类型，Atomic包提供了以下3个类：

- AtomicBoolean：原子更新布尔类型。
- AtomicInteger：原子更新整型。
- AtomicLong：原子更新长整型。

通过原子的方式更新数组里的某个元素，Atomic包提供了以下4个类：

- AtomicIntegerArray：原子更新整型数组里的元素。
- AtomicLongArray：原子更新长整型数组里的元素。
- AtomicReferenceArray：原子更新引用类型数组里的元素。
- AtomicIntegerArray类主要是提供原子的方式更新数组里的整型

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类：

- AtomicReference：原子更新引用类型。
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段。
- AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference（V initialRef，boolean initialMark）。

如果需原子地更新某个类里的某个字段时，就需要使用原子更新字段类，Atomic包提供了以下3个类进行原子字段更新：

- AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。
- AtomicLongFieldUpdater：原子更新长整型字段的更新器。
- AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的 ABA问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_36-atomicinteger-的原理)36.AtomicInteger 的原理？

一句话概括：**使用CAS实现**。

以AtomicInteger的添加方法为例：



```java
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
```

通过`Unsafe`类的实例来进行添加操作，来看看具体的CAS操作：



```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

compareAndSwapInt 是一个native方法，基于CAS来操作int类型变量。其它的原子操作类基本都是大同小异。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_37-线程死锁了解吗-该如何避免)37.线程死锁了解吗？该如何避免？

死锁是指两个或两个以上的线程在执行过程中，因争夺资源而造成的互相等待的现象，在无外力作用的情况下，这些线程会一直相互等待而无法继续运行下去。

![死锁示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-47.png)死锁示意图

那么为什么会产生死锁呢？ 死锁的产生必须具备以下四个条件：

![死锁产生必备四条件](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-48.png)死锁产生必备四条件

- 互斥条件：指线程对己经获取到的资源进行它性使用，即该资源同时只由一个线程占用。如果此时还有其它线程请求获取获取该资源，则请求者只能等待，直至占有资源的线程释放该资源。
- 请求并持有条件：指一个 线程己经持有了至少一个资源，但又提出了新的资源请求，而新资源己被其它线程占有，所以当前线程会被阻塞，但阻塞 的同时并不释放自己已经获取的资源。
- 不可剥夺条件：指线程获取到的资源在自己使用完之前不能被其它线程抢占，只有在自己使用完毕后才由自己释放该资源。
- 环路等待条件：指在发生死锁时，必然存在一个线程——资源的环形链，即线程集合 {T0，T1，T2,…… ，Tn} 中 T0 正在等待一 T1 占用的资源，Tl1正在等待 T2用的资源，…… Tn 在等待己被 T0占用的资源。

该如何避免死锁呢？答案是**至少破坏死锁发生的一个条件**。

- 其中，互斥这个条件我们没有办法破坏，因为用锁为的就是互斥。不过其他三个条件都是有办法破坏掉的，到底如何做呢？
- 对于“请求并持有”这个条件，可以一次性请求所有的资源。
- 对于“不可剥夺”这个条件，占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源，这样不可抢占这个条件就破坏掉了。
- 对于“环路等待”这个条件，可以靠按序申请资源来预防。所谓按序申请，是指资源是有线性顺序的，申请的时候可以先申请资源序号小的，再申请资源序号大的，这样线性化后就不存在环路了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_38-那死锁问题怎么排查呢)38.那死锁问题怎么排查呢？

可以使用jdk自带的命令行工具排查：

1. 使用jps查找运行的Java进程：jps -l
2. 使用jstack查看线程堆栈信息：jstack -l 进程id

基本就可以看到死锁的信息。

还可以利用图形化工具，比如JConsole。出现线程死锁以后，点击JConsole线程面板的`检测到死锁`按钮，将会看到线程的死锁信息。

![线程死锁检测](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-49.png)线程死锁检测

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#并发工具类)并发工具类

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_39-countdownlatch-倒计数器-了解吗)39.CountDownLatch（倒计数器）了解吗？

CountDownLatch，倒计数器，有两个常见的应用场景[18]：

**场景1：协调子线程结束动作：等待所有子线程运行结束**

CountDownLatch允许一个或多个线程等待其他线程完成操作。

例如，我们很多人喜欢玩的王者荣耀，开黑的时候，得等所有人都上线之后，才能开打。

![王者荣耀等待玩家确认-来源参考[18]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-50.jpeg)王者荣耀等待玩家确认-来源参考[18]

CountDownLatch模仿这个场景(参考[18])：

创建大乔、兰陵王、安其拉、哪吒和铠等五个玩家，主线程必须在他们都完成确认后，才可以继续运行。

在这段代码中，`new CountDownLatch(5)`用户创建初始的latch数量，各玩家通过`countDownLatch.countDown()`完成状态确认，主线程通过`countDownLatch.await()`等待。



```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch countDownLatch = new CountDownLatch(5);

    Thread 大乔 = new Thread(countDownLatch::countDown);
    Thread 兰陵王 = new Thread(countDownLatch::countDown);
    Thread 安其拉 = new Thread(countDownLatch::countDown);
    Thread 哪吒 = new Thread(countDownLatch::countDown);
    Thread 铠 = new Thread(() -> {
        try {
            // 稍等，上个卫生间，马上到...
            Thread.sleep(1500);
            countDownLatch.countDown();
        } catch (InterruptedException ignored) {}
    });

    大乔.start();
    兰陵王.start();
    安其拉.start();
    哪吒.start();
    铠.start();
    countDownLatch.await();
    System.out.println("所有玩家已经就位！");
}
```

**场景2. 协调子线程开始动作：统一各线程动作开始的时机**

王者游戏中也有类似的场景，游戏开始时，各玩家的初始状态必须一致。不能有的玩家都出完装了，有的才降生。

所以大家得一块出生，在

![王者荣耀-来源参考[18]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-51.jpeg)王者荣耀-来源参考[18]

在这个场景中，仍然用五个线程代表大乔、兰陵王、安其拉、哪吒和铠等五个玩家。需要注意的是，各玩家虽然都调用了`start()`线程，但是它们在运行时都在等待`countDownLatch`的信号，在信号未收到前，它们不会往下执行。



```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch countDownLatch = new CountDownLatch(1);

    Thread 大乔 = new Thread(() -> waitToFight(countDownLatch));
    Thread 兰陵王 = new Thread(() -> waitToFight(countDownLatch));
    Thread 安其拉 = new Thread(() -> waitToFight(countDownLatch));
    Thread 哪吒 = new Thread(() -> waitToFight(countDownLatch));
    Thread 铠 = new Thread(() -> waitToFight(countDownLatch));

    大乔.start();
    兰陵王.start();
    安其拉.start();
    哪吒.start();
    铠.start();
    Thread.sleep(1000);
    countDownLatch.countDown();
    System.out.println("敌方还有5秒达到战场，全军出击！");
}

private static void waitToFight(CountDownLatch countDownLatch) {
    try {
        countDownLatch.await(); // 在此等待信号再继续
        System.out.println("收到，发起进攻！");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

CountDownLatch的**核心方法**也不多：

- `await()`：等待latch降为0；
- `boolean await(long timeout, TimeUnit unit)`：等待latch降为0，但是可以设置超时时间。比如有玩家超时未确认，那就重新匹配，总不能为了某个玩家等到天荒地老。
- `countDown()`：latch数量减1；
- `getCount()`：获取当前的latch数量。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_40-cyclicbarrier-同步屏障-了解吗)40.CyclicBarrier（同步屏障）了解吗？

CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一 组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

它和CountDownLatch类似，都可以协调多线程的结束动作，在它们结束后都可以执行特定动作，但是为什么要有CyclicBarrier，自然是它有和CountDownLatch不同的地方。

不知道你听没听过一个新人UP主小约翰可汗，小约翰生平有两大恨——“想结衣结衣不依,迷爱理爱理不理。”我们来还原一下事情的经过：小约翰在亲政后认识了新垣结衣，于是决定第一次选妃，向结衣表白，等待回应。然而新垣结衣回应嫁给了星野源，小约翰伤心欲绝，发誓生平不娶，突然发现了铃木爱理，于是小约翰决定第二次选妃，求爱理搭理，等待回应。

![想结衣结衣不依,迷爱理爱理不理。](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-52.png)想结衣结衣不依,迷爱理爱理不理。

我们拿代码模拟这一场景，发现CountDownLatch无能为力了，因为CountDownLatch的使用是一次性的，无法重复利用，而这里等待了两次。此时，我们用CyclicBarrier就可以实现，因为它可以重复利用。

![小约翰可汗选妃模拟代码](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-53.png)小约翰可汗选妃模拟代码

运行结果：

![运行结果](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-54.png)运行结果

CyclicBarrier最最核心的方法，仍然是await()：

- 如果当前线程不是第一个到达屏障的话，它将会进入等待，直到其他线程都到达，除非发生**被中断**、**屏障被拆除**、**屏障被重设**等情况；

上面的例子抽象一下，本质上它的流程就是这样就是这样：

![CyclicBarrier工作流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-55.png)CyclicBarrier工作流程

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_41-cyclicbarrier和countdownlatch有什么区别)41.CyclicBarrier和CountDownLatch有什么区别？

两者最核心的区别[18]：

- CountDownLatch是一次性的，而CyclicBarrier则可以多次设置屏障，实现重复利用；
- CountDownLatch中的各个子线程不可以等待其他线程，只能完成自己的任务；而CyclicBarrier中的各个线程可以等待其他线程

它们区别用一个表格整理：

| CyclicBarrier                                                | CountDownLatch                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| CyclicBarrier是可重用的，其中的线程会等待所有的线程完成任务。届时，屏障将被拆除，并可以选择性地做一些特定的动作。 | CountDownLatch是一次性的，不同的线程在同一个计数器上工作，直到计数器为0. |
| CyclicBarrier面向的是线程数                                  | CountDownLatch面向的是任务数                                 |
| 在使用CyclicBarrier时，你必须在构造中指定参与协作的线程数，这些线程必须调用await()方法 | 使用CountDownLatch时，则必须要指定任务数，至于这些任务由哪些线程完成无关紧要 |
| CyclicBarrier可以在所有的线程释放后重新使用                  | CountDownLatch在计数器为0时不能再使用                        |
| 在CyclicBarrier中，如果某个线程遇到了中断、超时等问题时，则处于await的线程都会出现问题 | 在CountDownLatch中，如果某个线程出现问题，其他线程不受影响   |

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_42-semaphore-信号量-了解吗)42.Semaphore（信号量）了解吗？

Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。

听起来似乎很抽象，现在汽车多了，开车出门在外的一个老大难问题就是停车 。停车场的车位是有限的，只能允许若干车辆停泊，如果停车场还有空位，那么显示牌显示的就是绿灯和剩余的车位，车辆就可以驶入；如果停车场没位了，那么显示牌显示的就是绿灯和数字0，车辆就得等待。如果满了的停车场有车离开，那么显示牌就又变绿，显示空车位数量，等待的车辆就能进停车场。

![停车场空闲车位提示-图片来源网络](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-56.jpeg)停车场空闲车位提示-图片来源网络

我们把这个例子类比一下，车辆就是线程，进入停车场就是线程在执行，离开停车场就是线程执行完毕，看见红灯就表示线程被阻塞，不能执行，Semaphore的本质就是**协调多个线程对共享资源的获取**。

![Semaphore许可获取-来源参考[18]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-57.jpeg)Semaphore许可获取-来源参考[18]

我们再来看一个Semaphore的用途：它可以用于做流量控制，特别是公用资源有限的应用场景，比如数据库连接。

假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发地读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有10个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，就可以使用Semaphore来做流量控制，如下：



```java
public class SemaphoreTest {
    private static final int THREAD_COUNT = 30;
    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);
    private static Semaphore s = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        s.acquire();
                        System.out.println("save data");
                        s.release();
                    } catch (InterruptedException e) {
                    }
                }
            });
        }
        threadPool.shutdown();
    }
}
```

在代码中，虽然有30个线程在执行，但是只允许10个并发执行。Semaphore的构造方法` Semaphore（int permits`）接受一个整型的数字，表示可用的许可证数量。`Semaphore（10）`表示允许10个线程获取许可证，也就是最大并发数是10。Semaphore的用法也很简单，首先线程使用 Semaphore的acquire()方法获取一个许可证，使用完之后调用release()方法归还许可证。还可以用tryAcquire()方法尝试获取许可证。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_43-exchanger-了解吗)43.Exchanger 了解吗？

Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。

![英雄交换猎物-来源参考[18]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-58.jpeg)英雄交换猎物-来源参考[18]

这两个线程通过 exchange方法交换数据，如果第一个线程先执行exchange()方法，它会一直等待第二个线程也执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

Exchanger可以用于遗传算法，遗传算法里需要选出两个人作为交配对象，这时候会交换两人的数据，并使用交叉规则得出2个交配结果。Exchanger也可以用于校对工作，比如我们需要将纸制银行流水通过人工的方式录入成电子银行流水，为了避免错误，采用AB岗两人进行录入，录入到Excel之后，系统需要加载这两个Excel，并对两个Excel数据进行校对，看看是否录入一致。



```java
public class ExchangerTest {
    private static final Exchanger<String> exgr = new Exchanger<String>();
    private static ExecutorService threadPool = Executors.newFixedThreadPool(2);

    public static void main(String[] args) {
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String A = "银行流水A"; // A录入银行流水数据 
                    exgr.exchange(A);
                } catch (InterruptedException e) {
                }
            }
        });
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String B = "银行流水B"; // B录入银行流水数据 
                    String A = exgr.exchange("B");
                    System.out.println("A和B数据是否一致：" + A.equals(B) + "，A录入的是："
                            + A + "，B录入是：" + B);
                } catch (InterruptedException e) {
                }
            }
        });
        threadPool.shutdown();
    }
}
```

假如两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用`exchange(V x, long timeOut, TimeUnit unit) `设置最大等待时长。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#线程池)线程池

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_44-什么是线程池)44.什么是线程池？

**线程池：** 简单理解，它就是一个管理线程的池子。

![管理线程的池子](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-59.png)管理线程的池子

- **它帮我们管理线程，避免增加创建线程和销毁线程的资源损耗**。因为线程其实也是一个对象，创建一个对象，需要经过类加载过程，销毁一个对象，需要走GC垃圾回收流程，都是需要资源开销的。
- **提高响应速度。** 如果任务到达了，相对于从线程池拿线程，重新去创建一条线程执行，速度肯定慢很多。
- **重复利用。** 线程用完，再放回池子，可以达到重复利用的效果，节省资源。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_45-能说说工作中线程池的应用吗)45.能说说工作中线程池的应用吗？

之前我们有一个和第三方对接的需求，需要向第三方推送数据，引入了多线程来提升数据推送的效率，其中用到了线程池来管理线程。

![业务示例](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-60.png)业务示例

主要代码如下：

![主要代码](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-61.png)主要代码

完整可运行代码地址：[https://gitee.com/fighter3/thread-demo.gitopen in new window](https://gitee.com/fighter3/thread-demo.git)

线程池的参数如下：

- corePoolSize：线程核心参数选择了CPU数×2
- maximumPoolSize：最大线程数选择了和核心线程数相同
- keepAliveTime：非核心闲置线程存活时间直接置为0
- unit：非核心线程保持存活的时间选择了 TimeUnit.SECONDS 秒
- workQueue：线程池等待队列，使用 LinkedBlockingQueue阻塞队列

同时还用了synchronized 来加锁，保证数据不会被重复推送：



```java
  synchronized (PushProcessServiceImpl.class) {}
```

ps:这个例子只是简单地进行了数据推送，实际上还可以结合其他的业务，像什么数据清洗啊、数据统计啊，都可以套用。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_46-能简单说一下线程池的工作流程吗)46.能简单说一下线程池的工作流程吗？

用一个通俗的比喻：

有一个营业厅，总共有六个窗口，现在开放了三个窗口，现在有三个窗口坐着三个营业员小姐姐在营业。

老三去办业务，可能会遇到什么情况呢？

1. 老三发现有空间的在营业的窗口，直接去找小姐姐办理业务。

![直接办理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-62.png)直接办理

1. 老三发现没有空闲的窗口，就在排队区排队等。

![排队等待](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-63.png)排队等待

1. 老三发现没有空闲的窗口，等待区也满了，蚌埠住了，经理一看，就让休息的小姐姐赶紧回来上班，等待区号靠前的赶紧去新窗口办，老三去排队区排队。小姐姐比较辛苦，假如一段时间发现他们可以不用接着营业，经理就让她们接着休息。

![排队区满](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-64.png)排队区满

1. 老三一看，六个窗口都满了，等待区也没位置了。老三急了，要闹，经理赶紧出来了，经理该怎么办呢？

![等待区，排队区都满](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-65.png)等待区，排队区都满

> 1. 我们银行系统已经瘫痪
> 2. 谁叫你来办的你找谁去
> 3. 看你比较急，去队里加个塞
> 4. 今天没办法，不行你看改一天

上面的这个流程几乎就跟 JDK 线程池的大致流程类似，

> 1. 营业中的 3个窗口对应核心线程池数：corePoolSize
> 2. 总的营业窗口数6对应：maximumPoolSize
> 3. 打开的临时窗口在多少时间内无人办理则关闭对应：unit
> 4. 排队区就是等待队列：workQueue
> 5. 无法办理的时候银行给出的解决方法对应：RejectedExecutionHandler
> 6. threadFactory 该参数在 JDK 中是 线程工厂，用来创建线程对象，一般不会动。

所以我们线程池的工作流程也比较好理解了：

1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。
2. 当调用 execute() 方法添加一个任务时，线程池会做如下判断：

- 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；
- 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；
- 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
- 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会根据拒绝策略来对应处理。

![线程池执行流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-66.png)线程池执行流程

1. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
2. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_47-线程池主要参数有哪些)47.线程池主要参数有哪些？

![线程池参数](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-67.png)线程池参数

线程池有七大参数，需要重点关注`corePoolSize`、`maximumPoolSize`、`workQueue`、`handler`这四个。

1. corePoolSize

此值是用来初始化线程池中核心线程数，当线程池中线程池数< `corePoolSize`时，系统默认是添加一个任务才创建一个线程池。当线程数 = corePoolSize时，新任务会追加到workQueue中。

1. maximumPoolSize

`maximumPoolSize`表示允许的最大线程数 = (非核心线程数+核心线程数)，当`BlockingQueue`也满了，但线程池中总线程数 < `maximumPoolSize`时候就会再次创建新的线程。

1. keepAliveTime

非核心线程 =(maximumPoolSize - corePoolSize ) ,非核心线程闲置下来不干活最多存活时间。

1. unit

线程池中非核心线程保持存活的时间的单位

- TimeUnit.DAYS; 天
- TimeUnit.HOURS; 小时
- TimeUnit.MINUTES; 分钟
- TimeUnit.SECONDS; 秒
- TimeUnit.MILLISECONDS; 毫秒
- TimeUnit.MICROSECONDS; 微秒
- TimeUnit.NANOSECONDS; 纳秒

1. workQueue

线程池等待队列，维护着等待执行的`Runnable`对象。当运行当线程数= corePoolSize时，新的任务会被添加到`workQueue`中，如果`workQueue`也满了则尝试用非核心线程执行任务，等待队列应该尽量用有界的。

1. threadFactory

创建一个新线程时使用的工厂，可以用来设定线程名、是否为daemon线程等等。

1. handler

`corePoolSize`、`workQueue`、`maximumPoolSize`都不可用的时候执行的饱和策略。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_48-线程池的拒绝策略有哪些)48.线程池的拒绝策略有哪些？

类比前面的例子，无法办理业务时的处理方式，帮助记忆：

![四种策略](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-68.png)四种策略

- AbortPolicy ：直接抛出异常，默认使用此策略
- CallerRunsPolicy：用调用者所在的线程来执行任务
- DiscardOldestPolicy：丢弃阻塞队列里最老的任务，也就是队列里靠前的任务
- DiscardPolicy ：当前任务直接丢弃

想实现自己的拒绝策略，实现RejectedExecutionHandler接口即可。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_49-线程池有哪几种工作队列)49.线程池有哪几种工作队列？

常用的阻塞队列主要有以下几种：

![线程池常用阻塞队列](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-69.png)线程池常用阻塞队列

- ArrayBlockingQueue：ArrayBlockingQueue（有界队列）是一个用数组实现的有界阻塞队列，按FIFO排序量。
- LinkedBlockingQueue：LinkedBlockingQueue（可设置容量队列）是基于链表结构的阻塞队列，按FIFO排序任务，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为Integer.MAX_VALUE，吞吐量通常要高于ArrayBlockingQuene；newFixedThreadPool线程池使用了这个队列
- DelayQueue：DelayQueue（延迟队列）是一个任务定时周期的延迟执行的队列。根据指定的执行时间从小到大排序，否则根据插入到队列的先后排序。newScheduledThreadPool线程池使用了这个队列。
- PriorityBlockingQueue：PriorityBlockingQueue（优先级队列）是具有优先级的无界阻塞队列
- SynchronousQueue：SynchronousQueue（同步队列）是一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene，newCachedThreadPool线程池使用了这个队列。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_50-线程池提交execute和submit有什么区别)50.线程池提交execute和submit有什么区别？

1. execute 用于提交不需要返回值的任务



```java
threadsPool.execute(new Runnable() { 
    @Override public void run() { 
        //  Auto-generated method stub } 
    });
```

1. submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个 future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值



```java
Future<Object> future = executor.submit(harReturnValuetask); 
try { Object s = future.get(); } catch (InterruptedException e) { 
    // 处理中断异常 
} catch (ExecutionException e) { 
    // 处理无法执行任务异常 
} finally { 
    // 关闭线程池 executor.shutdown();
}
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_51-线程池怎么关闭知道吗)51.线程池怎么关闭知道吗？

可以通过调用线程池的`shutdown`或`shutdownNow`方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。

**shutdown() 将线程池状态置为shutdown,并不会立即停止**：

1. 停止接收外部submit的任务
2. 内部正在跑的任务和队列里等待的任务，会执行完
3. 等到第二步完成后，才真正停止

**shutdownNow() 将线程池状态置为stop。一般会立即停止，事实上不一定**：

1. 和shutdown()一样，先停止接收外部提交的任务
2. 忽略队列里等待的任务
3. 尝试将正在跑的任务interrupt中断
4. 返回未执行的任务列表

shutdown 和shutdownnow简单来说区别如下：

- shutdownNow()能立即停止线程池，正在跑的和正在等待的任务都停下了。这样做立即生效，但是风险也比较大。
- shutdown()只是关闭了提交通道，用submit()是无效的；而内部的任务该怎么跑还是怎么跑，跑完再彻底停止线程池。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_52-线程池的线程数应该怎么配置)52.线程池的线程数应该怎么配置？

线程在Java中属于稀缺资源，线程池不是越大越好也不是越小越好。任务分为计算密集型、IO密集型、混合型。

1. 计算密集型：大部分都在用CPU跟内存，加密，逻辑操作业务处理等。
2. IO密集型：数据库链接，网络通讯传输等。

![常见线程池参数配置方案-来源美团技术博客](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-70.png)常见线程池参数配置方案-来源美团技术博客

一般的经验，不同类型线程池的参数配置：

1. 计算密集型一般推荐线程池不要过大，一般是CPU数 + 1，+1是因为可能存在**页缺失**(就是可能存在有些数据在硬盘中需要多来一个线程将数据读入内存)。如果线程池数太大，可能会频繁的 进行线程上下文切换跟任务调度。获得当前CPU核心数代码如下：



```java
Runtime.getRuntime().availableProcessors();
```

1. IO密集型：线程数适当大一点，机器的Cpu核心数*2。
2. 混合型：可以考虑根绝情况将它拆分成CPU密集型和IO密集型任务，如果执行时间相差不大，拆分可以提升吞吐量，反之没有必要。

当然，实际应用中没有固定的公式，需要结合测试和监控来进行调整。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_53-有哪几种常见的线程池)53.有哪几种常见的线程池？

面试常问，主要有四种，都是通过工具类Excutors创建出来的，需要注意，阿里巴巴《Java开发手册》里禁止使用这种方式来创建线程池。

![四大线程池](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-71.png)四大线程池

- newFixedThreadPool (固定数目线程的线程池)
- newCachedThreadPool (可缓存线程的线程池)
- newSingleThreadExecutor (单线程的线程池)
- newScheduledThreadPool (定时及周期执行的线程池)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_54-能说一下四种常见线程池的原理吗)54.能说一下四种常见线程池的原理吗？

前三种线程池的构造直接调用ThreadPoolExecutor的构造方法。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#newsinglethreadexecutor)newSingleThreadExecutor



```java
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```

**线程池特点**

- 核心线程数为1
- 最大线程数也为1
- 阻塞队列是无界队列LinkedBlockingQueue，可能会导致OOM
- keepAliveTime为0

![SingleThreadExecutor运行流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-72.png)SingleThreadExecutor运行流程

工作流程：

- 提交任务
- 线程池是否有一条线程在，如果没有，新建线程执行任务
- 如果有，将任务加到阻塞队列
- 当前的唯一线程，从队列取任务，执行完一个，再继续取，一个线程执行任务。

**适用场景**

适用于串行执行任务的场景，一个任务一个任务地执行。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#newfixedthreadpool)newFixedThreadPool



```java
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

**线程池特点：**

- 核心线程数和最大线程数大小一样
- 没有所谓的非空闲时间，即keepAliveTime为0
- 阻塞队列为无界队列LinkedBlockingQueue，可能会导致OOM

![FixedThreadPool](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-73.png)FixedThreadPool

工作流程：

- 提交任务
- 如果线程数少于核心线程，创建核心线程执行任务
- 如果线程数等于核心线程，把任务添加到LinkedBlockingQueue阻塞队列
- 如果线程执行完任务，去阻塞队列取任务，继续执行。

**使用场景**

FixedThreadPool 适用于处理CPU密集型的任务，确保CPU在长期被工作线程使用的情况下，尽可能的少的分配线程，即适用执行长期的任务。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#newcachedthreadpool)newCachedThreadPool



```java
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

**线程池特点：**

- 核心线程数为0
- 最大线程数为Integer.MAX_VALUE，即无限大，可能会因为无限创建线程，导致OOM
- 阻塞队列是SynchronousQueue
- 非核心线程空闲存活时间为60秒

当提交任务的速度大于处理任务的速度时，每次提交一个任务，就必然会创建一个线程。极端情况下会创建过多的线程，耗尽 CPU 和内存资源。由于空闲 60 秒的线程会被终止，长时间保持空闲的 CachedThreadPool 不会占用任何资源。

![CachedThreadPool执行流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-74.png)CachedThreadPool执行流程

工作流程：

- 提交任务
- 因为没有核心线程，所以任务直接加到SynchronousQueue队列。
- 判断是否有空闲线程，如果有，就去取出任务执行。
- 如果没有空闲线程，就新建一个线程执行。
- 执行完任务的线程，还可以存活60秒，如果在这期间，接到任务，可以继续活下去；否则，被销毁。

**适用场景**

用于并发执行大量短期的小任务。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#newscheduledthreadpool)newScheduledThreadPool



```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

**线程池特点**

- 最大线程数为Integer.MAX_VALUE，也有OOM的风险
- 阻塞队列是DelayedWorkQueue
- keepAliveTime为0
- scheduleAtFixedRate() ：按某种速率周期执行
- scheduleWithFixedDelay()：在某个延迟后执行

![ScheduledThreadPool执行流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-75.png)ScheduledThreadPool执行流程

**工作机制**

- 线程从DelayQueue中获取已到期的ScheduledFutureTask（DelayQueue.take()）。到期任务是指ScheduledFutureTask的time大于等于当前时间。
- 线程执行这个ScheduledFutureTask。
- 线程修改ScheduledFutureTask的time变量为下次将要被执行的时间。
- 线程把这个修改time之后的ScheduledFutureTask放回DelayQueue中（DelayQueue.add()）。

![ScheduledThreadPoolExecutor执行流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-76.png)ScheduledThreadPoolExecutor执行流程

**使用场景**

周期性执行任务的场景，需要限制线程数量的场景

> 使用无界队列的线程池会导致什么问题吗？

例如newFixedThreadPool使用了无界的阻塞队列LinkedBlockingQueue，如果线程获取一个任务后，任务的执行时间比较长，会导致队列的任务越积越多，导致机器内存使用不停飙升，最终导致OOM。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_55-线程池异常怎么处理知道吗)55.线程池异常怎么处理知道吗？

在使用线程池处理任务的时候，任务代码可能抛出RuntimeException，抛出异常后，线程池可能捕获它，也可能创建一个新的线程来代替异常的线程，我们可能无法感知任务出现了异常，因此我们需要考虑线程池异常情况。

常见的异常处理方式：

![线程池异常处理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-77.png)线程池异常处理

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_56-能说一下线程池有几种状态吗)56.能说一下线程池有几种状态吗？

线程池有这几个状态：RUNNING,SHUTDOWN,STOP,TIDYING,TERMINATED。



```java
//线程池状态
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

线程池各个状态切换图：

![线程池状态切换图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-78.png)线程池状态切换图

**RUNNING**

- 该状态的线程池会接收新任务，并处理阻塞队列中的任务;
- 调用线程池的shutdown()方法，可以切换到SHUTDOWN状态;
- 调用线程池的shutdownNow()方法，可以切换到STOP状态;

**SHUTDOWN**

- 该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；
- 队列为空，并且线程池中执行的任务也为空,进入TIDYING状态;

**STOP**

- 该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；
- 线程池中执行的任务为空,进入TIDYING状态;

**TIDYING**

- 该状态表明所有的任务已经运行终止，记录的任务数量为0。
- terminated()执行完毕，进入TERMINATED状态

**TERMINATED**

- 该状态表示线程池彻底终止

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_57-线程池如何实现参数的动态修改)57.线程池如何实现参数的动态修改？

线程池提供了几个 setter方法来设置线程池的参数。

![JDK 线程池参数设置接口来源参考[7]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-79.png)JDK 线程池参数设置接口来源参考[7]

这里主要有两个思路：

![动态修改线程池参数](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-80.png)动态修改线程池参数

- 在我们微服务的架构下，可以利用配置中心如Nacos、Apollo等等，也可以自己开发配置中心。业务服务读取线程池配置，获取相应的线程池实例来修改线程池的参数。
- 如果限制了配置中心的使用，也可以自己去扩展**ThreadPoolExecutor**，重写方法，监听线程池参数变化，来动态修改线程池参数。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#线程池调优了解吗)线程池调优了解吗？

线程池配置没有固定的公式，通常事前会对线程池进行一定评估，常见的评估方案如下：

![线程池评估方案 来源参考[7]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-81.png)线程池评估方案 来源参考[7]

上线之前也要进行充分的测试，上线之后要建立完善的线程池监控机制。

事中结合监控告警机制，分析线程池的问题，或者可优化点，结合线程池动态参数配置机制来调整配置。

事后要注意仔细观察，随时调整。

![线程池调优](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-82.png)线程池调优

具体的调优案例可以查看参考[7]美团技术博客。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_58-你能设计实现一个线程池吗)58.你能设计实现一个线程池吗？

⭐这道题在阿里的面试中出现频率比较高

线程池实现原理可以查看 [要是以前有人这么讲线程池，我早就该明白了！open in new window](https://mp.weixin.qq.com/s/Exy7pRGND9TCjRd9TZK4jg) ，当然，我们自己实现， 只需要抓住线程池的核心流程-参考[6]：

![线程池主要实现流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-83.png)线程池主要实现流程

我们自己的实现就是完成这个核心流程：

- 线程池中有N个工作线程
- 把任务提交给线程池运行
- 如果线程池已满，把任务放入队列
- 最后当有空闲时，获取队列中任务来执行

实现代码[6]：

![自定义线程池](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-84.png)自定义线程池

这样，一个实现了线程池主要流程的类就完成了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_59-单机线程池执行断电了应该怎么处理)59.单机线程池执行断电了应该怎么处理？


我们可以对正在处理和阻塞队列的任务做事务管理或者对阻塞队列中的任务持久化处理，并且当断电或者系统崩溃，操作无法继续下去的时候，可以通过回溯日志的方式来撤销`正在处理`的已经执行成功的操作。然后重新执行整个阻塞队列。

也就是说，对阻塞队列持久化；正在处理任务事务控制；断电之后正在处理任务的回滚，通过日志恢复该次操作；服务器重启后阻塞队列中的数据再加载。

## [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#并发容器和框架)并发容器和框架

关于一些并发容器，可以去看看 [面渣逆袭：Java集合连环三十问 open in new window](https://mp.weixin.qq.com/s/SHkQ7LEOT0itt4bXMoDBPw)，里面有`CopyOnWriteList`和`ConcurrentHashMap`这两种线程安全容器类的问答。。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/javathread.html#_60-fork-join框架了解吗)60.Fork/Join框架了解吗？

Fork/Join框架是Java7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

要想掌握Fork/Join框架，首先需要理解两个点，**分而治之**和**工作窃取算法**。

**分而治之**

Fork/Join框架的定义，其实就体现了分治思想：将一个规模为N的问题分解为K个规模较小的子问题，这些子问题相互独立且与原问题性质相同。求出子问题的解，就可得到原问题的解。

![Fork/Join分治算法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-85.png)Fork/Join分治算法

**工作窃取算法**

大任务拆成了若干个小任务，把这些小任务放到不同的队列里，各自创建单独线程来执行队列里的任务。

那么问题来了，有的线程干活块，有的线程干活慢。干完活的线程不能让它空下来，得让它去帮没干完活的线程干活。它去其它线程的队列里窃取一个任务来执行，这就是所谓的**工作窃取**。

工作窃取发生的时候，它们会访问同一个队列，为了减少窃取任务线程和被窃取任务线程之间的竞争，通常任务会使用双端队列，被窃取任务线程永远从双端队列的头部拿，而窃取任务的线程永远从双端队列的尾部拿任务执行。

![工作窃取](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-86.png)工作窃取

看一个Fork/Join框架应用的例子，计算1~n之间的和：1+2+3+…+n

- 设置一个分割阈值，任务大于阈值就拆分任务
- 任务有结果，所以需要继承RecursiveTask



```java
public class CountTask extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 16; // 阈值
    private int start;
    private int end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        // 如果任务足够小就计算任务
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            // 执行子任务
            leftTask.fork();
            rightTask.fork(); // 等待子任务执行完，并得到其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join(); // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool(); // 生成一个计算任务，负责计算1+2+3+4
        CountTask task = new CountTask(1, 100); // 执行一个任务
        Future<Integer> result = forkJoinPool.submit(task);
        try {
            System.out.println(result.get());
        } catch (InterruptedException e) {
        } catch (ExecutionException e) {
        }
    }
    
}
```

ForkJoinTask与一般Task的主要区别在于它需要实现compute方法，在这个方法里，首先需要判断任务是否足够小，如果足够小就直接执行任务。如果比较大，就必须分割成两个子任务，每个子任务在调用fork方法时，又会进compute方法，看看当前子任务是否需要继续分割成子任务，如果不需要继续分割，则执行当前子任务并返回结果。使用join方法会等待子任务执行完并得到其结果。

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。



# JVM  Interview

## 一、引言

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_1-什么是-jvm)1.什么是 JVM?

JVM——Java 虚拟机，它是 Java 实现平台无关性的基石。

Java 程序运行的时候，编译器将 Java 文件编译成平台无关的 Java 字节码文件（.class）,接下来对应平台 JVM 对字节码文件进行解释，翻译成对应平台匹配的机器指令并运行。

![Java语言编译运行](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-1.png)Java语言编译运行

同时 JVM 也是一个跨语言的平台，和语言无关，只和 class 的文件格式关联，任何语言，只要能翻译成符合规范的字节码文件，都能被 JVM 运行。

![JVM跨语言](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-2.png)JVM跨语言

## [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#二、内存管理)二、内存管理

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_2-能说一下-jvm-的内存区域吗)2.能说一下 JVM 的内存区域吗？

JVM 内存区域最粗略的划分可以分为`堆`和`栈`，当然，按照虚拟机规范，可以划分为以下几个区域：

![Java虚拟机运行时数据区](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-3.png)Java虚拟机运行时数据区

JVM 内存分为线程私有区和线程共享区，其中`方法区`和`堆`是线程共享区，`虚拟机栈`、`本地方法栈`和`程序计数器`是线程隔离的数据区。

**1）程序计数器**

程序计数器（Program Counter Register）也被称为 PC 寄存器，是一块较小的内存空间。

它可以看作是当前线程所执行的字节码的行号指示器。

**2）Java 虚拟机栈**

Java 虚拟机栈（Java Virtual Machine Stack）也是线程私有的，它的生命周期与线程相同。

Java 虚拟机栈描述的是 Java 方法执行的线程内存模型：方法执行时，JVM 会同步创建一个栈帧，用来存储局部变量表、操作数栈、动态连接等。

![Java虚拟机栈](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-4.png)Java虚拟机栈

**3）本地方法栈**

本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别只是虚拟机栈为虚拟机执行 Java 方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的本地（Native）方法服务。

Java 虚拟机规范允许本地方法栈被实现成固定大小的或者是根据计算动态扩展和收缩的。

**4）Java 堆**

对于 Java 应用程序来说，Java 堆（Java Heap）是虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，Java 里“**几乎**”所有的对象实例都在这里分配内存。

Java 堆是垃圾收集器管理的内存区域，因此一些资料中它也被称作“GC 堆”（Garbage Collected Heap，）。从回收内存的角度看，由于现代垃圾收集器大部分都是基于分代收集理论设计的，所以 Java 堆中经常会出现`新生代`、`老年代`、`Eden空间`、`From Survivor空间`、`To Survivor空间`等名词，需要注意的是这种划分只是根据垃圾回收机制来进行的划分，不是 Java 虚拟机规范本身制定的。

![Java 堆内存结构](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-5.png)Java 堆内存结构

**5）方法区**

方法区是比较特别的一块区域，和堆类似，它也是各个线程共享的内存区域，用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

它特别在 Java 虚拟机规范对它的约束非常宽松，所以方法区的具体实现历经了许多变迁，例如 jdk1.7 之前使用永久代作为方法区的实现。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_3-说一下-jdk1-6、1-7、1-8-内存区域的变化)3.说一下 JDK1.6、1.7、1.8 内存区域的变化？

JDK1.6、1.7/1.8 内存区域发生了变化，主要体现在方法区的实现：

- JDK1.6 使用永久代实现方法区：

![JDK 1.6内存区域](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-6.png)JDK 1.6内存区域

- JDK1.7 时发生了一些变化，将字符串常量池、静态变量，存放在堆上

![JDK 1.7内存区域](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-7.png)JDK 1.7内存区域

- 在 JDK1.8 时彻底干掉了永久代，而在直接内存中划出一块区域作为**元空间**，运行时常量池、类常量池都移动到元空间。

![JDK 1.8内存区域](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-8.png)JDK 1.8内存区域

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_4-为什么使用元空间替代永久代作为方法区的实现)4.为什么使用元空间替代永久代作为方法区的实现？

Java 虚拟机规范规定的方法区只是换种方式实现。有客观和主观两个原因。

- 客观上使用永久代来实现方法区的决定的设计导致了 Java 应用更容易遇到内存溢出的问题（永久代有-XX：MaxPermSize 的上限，即使不设置也有默认大小，而 J9 和 JRockit 只要没有触碰到进程可用内存的上限，例如 32 位系统中的 4GB 限制，就不会出问题），而且有极少数方法 （例如 String::intern()）会因永久代的原因而导致不同虚拟机下有不同的表现。
- 主观上当 Oracle 收购 BEA 获得了 JRockit 的所有权后，准备把 JRockit 中的优秀功能，譬如 Java Mission Control 管理工具，移植到 HotSpot 虚拟机时，但因为两者对方法区实现的差异而面临诸多困难。考虑到 HotSpot 未来的发展，在 JDK 6 的 时候 HotSpot 开发团队就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区的计划了，到了 JDK 7 的 HotSpot，已经把原本放在永久代的字符串常量池、静态变量等移出，而到了 JDK 8，终于完全废弃了永久代的概念，改用与 JRockit、J9 一样在本地内存中实现的元空间（Meta-space）来代替，把 JDK 7 中永久代还剩余的内容（主要是类型信息）全部移到元空间中。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_5-对象创建的过程了解吗)5.对象创建的过程了解吗？

在 JVM 中对象的创建，我们从一个 new 指令开始：

- 首先检查这个指令的参数是否能在常量池中定位到一个类的符号引用
- 检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，就先执行相应的类加载过程
- 类加载检查通过后，接下来虚拟机将为新生对象分配内存。
- 内存分配完成之后，虚拟机将分配到的内存空间（但不包括对象头）都初始化为零值。
- 接下来设置对象头，请求头里包含了对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。

这个过程大概图示如下：

![对象创建过程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-9.png)对象创建过程

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_6-什么是指针碰撞-什么是空闲列表)6.什么是指针碰撞？什么是空闲列表？

内存分配有两种方式，**指针碰撞**（Bump The Pointer）、**空闲列表**（Free List）。

![指针碰撞和空闲列表](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-10.png)指针碰撞和空闲列表

- 指针碰撞：假设 Java 堆中内存是绝对规整的，所有被使用过的内存都被放在一边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间方向挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”。
- 空闲列表：如果 Java 堆中的内存并不是规整的，已被使用的内存和空闲的内存相互交错在一起，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为“空闲列表”。
- 两种方式的选择由 Java 堆是否规整决定，Java 堆是否规整是由选择的垃圾收集器是否具有压缩整理能力决定的。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_7-jvm-里-new-对象时-堆会发生抢占吗-jvm-是怎么设计来保证线程安全的)7.JVM 里 new 对象时，堆会发生抢占吗？JVM 是怎么设计来保证线程安全的？

会，假设 JVM 虚拟机上，每一次 new 对象时，指针就会向右移动一个对象 size 的距离，一个线程正在给 A 对象分配内存，指针还没有来的及修改，另一个为 B 对象分配内存的线程，又引用了这个指针来分配内存，这就发生了抢占。

有两种可选方案来解决这个问题：

![堆抢占和解决方案](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-11.png)堆抢占和解决方案

- 采用 CAS 分配重试的方式来保证更新操作的原子性

- 每个线程在 Java 堆中预先分配一小块内存，也就是本地线程分配缓冲（Thread Local Allocation

  Buffer，TLAB），要分配内存的线程，先在本地缓冲区中分配，只有本地缓冲区用完了，分配新的缓存区时才需要同步锁定。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_8-能说一下对象的内存布局吗)8.能说一下对象的内存布局吗？

在 HotSpot 虚拟机里，对象在堆内存中的存储布局可以划分为三个部分：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。

![对象的存储布局](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-12.png)对象的存储布局

**对象头**主要由两部分组成：

- 第一部分存储对象自身的运行时数据：哈希码、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等，官方称它为 Mark Word，它是个动态的结构，随着对象状态变化。
- 第二部分是类型指针，指向对象的类元数据类型（即对象代表哪个类）。
- 此外，如果对象是一个 Java 数组，那还应该有一块用于记录数组长度的数据

**实例数据**用来存储对象真正的有效信息，也就是我们在程序代码里所定义的各种类型的字段内容，无论是从父类继承的，还是自己定义的。

**对齐填充**不是必须的，没有特别含义，仅仅起着占位符的作用。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_9-对象怎么访问定位)9.对象怎么访问定位？

Java 程序会通过栈上的 reference 数据来操作堆上的具体对象。由于 reference 类型在《Java 虚拟机规范》里面只规定了它是一个指向对象的引用，并没有定义这个引用应该通过什么方式去定位、访问到堆中对象的具体位置，所以对象访问方式也是由虚拟机实现而定的，主流的访问方式主要有使用句柄和直接指针两种：

- 如果使用句柄访问的话，Java 堆中将可能会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自具体的地址信息，其结构如图所示：

![通过句柄访问对象](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-13.png)通过句柄访问对象

- 如果使用直接指针访问的话，Java 堆中对象的内存布局就必须考虑如何放置访问类型数据的相关信息，reference 中存储的直接就是对象地址，如果只是访问对象本身的话，就不需要多一次间接访问的开销，如图所示：

![通过直接指针访问对象](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-14.png)通过直接指针访问对象

这两种对象访问方式各有优势，使用句柄来访问的最大好处就是 reference 中存储的是稳定句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而 reference 本身不需要被修改。

使用直接指针来访问最大的好处就是速度更快，它节省了一次指针定位的时间开销，由于对象访问在 Java 中非常频繁，因此这类开销积少成多也是一项极为可观的执行成本。

HotSpot 虚拟机主要使用直接指针来进行对象访问。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_10-内存溢出和内存泄漏是什么意思)10.内存溢出和内存泄漏是什么意思？

内存泄露就是申请的内存空间没有被正确释放，导致内存被白白占用。

内存溢出就是申请的内存超过了可用内存，内存不够了。

两者关系：内存泄露可能会导致内存溢出。

用一个有味道的比喻，内存溢出就是排队去蹲坑，发现没坑位了，内存泄漏，就是有人占着茅坑不拉屎，占着茅坑不拉屎的多了可能会导致坑位不够用。

![内存泄漏、内存溢出](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-15.png)内存泄漏、内存溢出

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_11-能手写内存溢出的例子吗)11.能手写内存溢出的例子吗？

在 JVM 的几个内存区域中，除了程序计数器外，其他几个运行时区域都有发生内存溢出（OOM）异常的可能，重点关注堆和栈。

- Java 堆溢出

Java 堆用于储存对象实例，只要不断创建不可被回收的对象，比如静态对象，那么随着对象数量的增加，总容量触及最大堆的容量限制后就会产生内存溢出异常（OutOfMemoryError）。

这就相当于一个房子里，不断堆积不能被收走的杂物，那么房子很快就会被堆满了。



```java
/**
 * VM参数： -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 */
public class HeapOOM {
    static class OOMObject {
    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();
        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

- 虚拟机栈.OutOfMemoryError

JDK 使用的 HotSpot 虚拟机的栈内存大小是固定的，我们可以把栈的内存设大一点，然后不断地去创建线程，因为操作系统给每个进程分配的内存是有限的，所以到最后，也会发生 OutOfMemoryError 异常。



```java
/**
 * vm参数：-Xss2M
 */
public class JavaVMStackOOM {
    private void dontStop() {
        while (true) {
        }
    }

    public void stackLeakByThread() {
        while (true) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    dontStop();
                }
            });
            thread.start();
        }
    }

    public static void main(String[] args) throws Throwable {
        JavaVMStackOOM oom = new JavaVMStackOOM();
        oom.stackLeakByThread();
    }
}
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_12-内存泄漏可能由哪些原因导致呢)12.内存泄漏可能由哪些原因导致呢？

内存泄漏可能的原因有很多种：

![内存泄漏可能原因](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-16.png)内存泄漏可能原因

**静态集合类引起内存泄漏**

静态集合的生命周期和 JVM 一致，所以静态集合引用的对象不能被释放。



```java
public class OOM {
 static List list = new ArrayList();

 public void oomTests(){
   Object obj = new Object();

   list.add(obj);
  }
}
```

**单例模式**

和上面的例子原理类似，单例对象在初始化后会以静态变量的方式在 JVM 的整个生命周期中存在。如果单例对象持有外部的引用，那么这个外部对象将不能被 GC 回收，导致内存泄漏。

**数据连接、IO、Socket 等连接**

创建的连接不再使用时，需要调用 **close** 方法关闭连接，只有连接被关闭后，GC 才会回收对应的对象（Connection，Statement，ResultSet，Session）。忘记关闭这些资源会导致持续占有内存，无法被 GC 回收。



```java
try {
    Connection conn = null;
    Class.forName("com.mysql.jdbc.Driver");
    conn = DriverManager.getConnection("url", "", "");
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery("....");
  } catch (Exception e) {

  }finally {
    //不关闭连接
  }
```

**变量不合理的作用域**

一个变量的定义作用域大于其使用范围，很可能存在内存泄漏；或不再使用对象没有及时将对象设置为 null，很可能导致内存泄漏的发生。



```java
public class Simple {
    Object object;
    public void method1(){
        object = new Object();
        //...其他代码
        //由于作用域原因，method1执行完成之后，object 对象所分配的内存不会马上释放
        object = null;
    }
}
```

**hash 值发生变化**

对象 Hash 值改变，使用 HashMap、HashSet 等容器中时候，由于对象修改之后的 Hah 值和存储进容器时的 Hash 值不同，所以无法找到存入的对象，自然也无法单独删除了，这也会造成内存泄漏。说句题外话，这也是为什么 String 类型被设置成了不可变类型。

**ThreadLocal 使用不当**

ThreadLocal 的弱引用导致内存泄漏也是个老生常谈的话题了，使用完 ThreadLocal 一定要记得使用 remove 方法来进行清除。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_13-如何判断对象仍然存活)13.如何判断对象仍然存活？

有两种方式，**引用计数算法（reference counting）**和可达性分析算法。

- **引用计数算法**

引用计数器的算法是这样的：在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一；当引用失效时，计数器值就减一；任何时刻计数器为零的对象就是不可能再被使用的。

![引用计数算法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-17.png)引用计数算法

- **可达性分析算法**

目前 Java 虚拟机的主流垃圾回收器采取的是可达性分析算法。这个算法的实质在于将一系列 GC Roots 作为初始的存活对象合集（Gc Root Set），然后从该合集出发，探索所有能够被该集合引用到的对象，并将其加入到该集合中，这个过程我们也称之为标记（mark）。最终，未被探索到的对象便是死亡的，是可以回收的。 ![GC Root](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-18.png)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_14-java-中可作为-gc-roots-的对象有哪几种)14.Java 中可作为 GC Roots 的对象有哪几种？

可以作为 GC Roots 的主要有四种对象：

- 虚拟机栈(栈帧中的本地变量表)中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中 JNI 引用的对象

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_15-说一下对象有哪几种引用)15.说一下对象有哪几种引用？

Java 中的引用有四种，分为强引用（Strongly Reference）、软引用（Soft Reference）、弱引用（Weak Reference）和虚引用（Phantom Reference）4 种，这 4 种引用强度依次逐渐减弱。

- 强引用是最传统的`引用`的定义，是指在程序代码之中普遍存在的引用赋值，无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。



```java
Object obj =new Object();
```

- 软引用是用来描述一些还有用，但非必须的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存， 才会抛出内存溢出异常。在 JDK 1.2 版之后提供了 SoftReference 类来实现软引用。



```java
Object obj = new Object();
ReferenceQueue queue = new ReferenceQueue();
SoftReference reference = new SoftReference(obj, queue);
//强引用对象滞空，保留软引用
obj = null;
```

- 弱引用也是用来描述那些非必须对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在 JDK 1.2 版之后提供了 WeakReference 类来实现弱引用。



```java
Object obj = new Object();
ReferenceQueue queue = new ReferenceQueue();
WeakReference reference = new WeakReference(obj, queue);
//强引用对象滞空，保留软引用
obj = null;
```

- 虚引用也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。在 JDK 1.2 版之后提供了 PhantomReference 类来实现虚引用。



```java
Object obj = new Object();
ReferenceQueue queue = new ReferenceQueue();
PhantomReference reference = new PhantomReference(obj, queue);
//强引用对象滞空，保留软引用
obj = null;
```

![四种引用总结](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-19.png)四种引用总结

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_16-finalize-方法了解吗-有什么作用)16.finalize()方法了解吗？有什么作用？

用一个不太贴切的比喻，垃圾回收就是古代的秋后问斩，finalize()就是刀下留人，在人犯被处决之前，还要做最后一次审计，青天大老爷看看有没有什么冤情，需不需要刀下留人。

![刀下留人](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-20.png)刀下留人

如果对象在进行可达性分析后发现没有与 GC Roots 相连接的引用链，那它将会被第一次标记，随后进行一次筛选，筛选的条件是此对象是否有必要执行 finalize()方法。如果对象在在 finalize()中成功拯救自己——只要重新与引用链上的任何一个对象建立关联即可，譬如把自己 （this 关键字）赋值给某个类变量或者对象的成员变量，那在第二次标记时它就”逃过一劫“；但是如果没有抓住这个机会，那么对象就真的要被回收了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_17-java-堆的内存分区了解吗)17.Java 堆的内存分区了解吗？

按照垃圾收集，将 Java 堆划分为**新生代 （Young Generation）**和**老年代（Old Generation）**两个区域，新生代存放存活时间短的对象，而每次回收后存活的少量对象，将会逐步晋升到老年代中存放。

而新生代又可以分为三个区域，eden、from、to，比例是 8：1：1，而新生代的内存分区同样是从垃圾收集的角度来分配的。

![Java堆内存划分](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-21.png)Java堆内存划分

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_18-垃圾收集算法了解吗)18.垃圾收集算法了解吗？

垃圾收集算法主要有三种：

1. **标记-清除算法**

见名知义，`标记-清除`（Mark-Sweep）算法分为两个阶段：

- **标记** : 标记出所有需要回收的对象
- **清除**：回收所有被标记的对象

![标记-清除算法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-22.png)标记-清除算法

标记-清除算法比较基础，但是主要存在两个缺点：

- 执行效率不稳定，如果 Java 堆中包含大量对象，而且其中大部分是需要被回收的，这时必须进行大量标记和清除的动作，导致标记和清除两个过程的执行效率都随对象数量增长而降低。
- 内存空间的碎片化问题，标记、清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致当以后在程序运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

1. **标记-复制算法**

标记-复制算法解决了标记-清除算法面对大量可回收对象时执行效率低的问题。

过程也比较简单：将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

![标记-复制算法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-23.png)标记-复制算法

这种算法存在一个明显的缺点：一部分空间没有使用，存在空间的浪费。

新生代垃圾收集主要采用这种算法，因为新生代的存活对象比较少，每次复制的只是少量的存活对象。当然，实际新生代的收集不是按照这个比例。

1. **标记-整理算法**

为了降低内存的消耗，引入一种针对性的算法：`标记-整理`（Mark-Compact）算法。

其中的标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存。

![标记-整理算法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-24.png)标记-整理算法

标记-整理算法主要用于老年代，移动存活对象是个极为负重的操作，而且这种操作需要 Stop The World 才能进行，只是从整体的吞吐量来考量，老年代使用标记-整理算法更加合适。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_19-说一下新生代的区域划分)19.说一下新生代的区域划分？

新生代的垃圾收集主要采用标记-复制算法，因为新生代的存活对象比较少，每次复制少量的存活对象效率比较高。

基于这种算法，虚拟机将内存分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次分配内存只使用 Eden 和其中一块 Survivor。发生垃圾收集时，将 Eden 和 Survivor 中仍然存活的对象一次性复制到另外一块 Survivor 空间上，然后直接清理掉 Eden 和已用过的那块 Survivor 空间。默认 Eden 和 Survivor 的大小比例是 8∶1。

![新生代内存划分](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-25.png)新生代内存划分

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_20-minor-gc-young-gc、major-gc-old-gc、mixed-gc、full-gc-都是什么意思)20.Minor GC/Young GC、Major GC/Old GC、Mixed GC、Full GC 都是什么意思？

**部分收集**（Partial GC）：指目标不是完整收集整个 Java 堆的垃圾收集，其中又分为：

- 新生代收集（Minor GC/Young GC）：指目标只是新生代的垃圾收集。
- 老年代收集（Major GC/Old GC）：指目标只是老年代的垃圾收集。目前**只有**CMS 收集器会有单独收集老年代的行为。
- 混合收集（Mixed GC）：指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有 G1 收集器会有这种行为。

**整堆收集**（Full GC）：收集整个 Java 堆和方法区的垃圾收集。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_21-minor-gc-young-gc-什么时候触发)21.Minor GC/Young GC 什么时候触发？

新创建的对象优先在新生代 Eden 区进行分配，如果 Eden 区没有足够的空间时，就会触发 Young GC 来清理新生代。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_22-什么时候会触发-full-gc)22.什么时候会触发 Full GC？

这个触发条件稍微有点多，往下看：

![Full GC触发条件](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-26.png)Full GC触发条件

- **Young GC 之前检查老年代**：在要进行 Young GC 的时候，发现`老年代可用的连续内存空间` < `新生代历次Young GC后升入老年代的对象总和的平均大小`，说明本次 Young GC 后可能升入老年代的对象大小，可能超过了老年代当前可用内存空间,那就会触发 Full GC。
- **Young GC 之后老年代空间不足**：执行 Young GC 之后有一批对象需要放入老年代，此时老年代就是没有足够的内存空间存放这些对象了，此时必须立即触发一次 Full GC
- **老年代空间不足**，老年代内存使用率过高，达到一定比例，也会触发 Full GC。
- **空间分配担保失败**（ Promotion Failure），新生代的 To 区放不下从 Eden 和 From 拷贝过来对象，或者新生代对象 GC 年龄到达阈值需要晋升这两种情况，老年代如果放不下的话都会触发 Full GC。
- **方法区内存空间不足**：如果方法区由永久代实现，永久代空间不足 Full GC。
- **System.gc()等命令触发**：System.gc()、jmap -dump 等命令会触发 full gc。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_23-对象什么时候会进入老年代)23.对象什么时候会进入老年代？

![对象进入老年代](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-27.png)对象进入老年代

**长期存活的对象将进入老年代**

在对象的对象头信息中存储着对象的迭代年龄,迭代年龄会在每次 YoungGC 之后对象的移区操作中增加,每一次移区年龄加一.当这个年龄达到 15(默认)之后,这个对象将会被移入老年代。

可以通过这个参数设置这个年龄值。



```java
- XX:MaxTenuringThreshold
```

**大对象直接进入老年代**

有一些占用大量连续内存空间的对象在被加载就会直接进入老年代.这样的大对象一般是一些数组,长字符串之类的对。

HotSpot 虚拟机提供了这个参数来设置。



```java
-XX：PretenureSizeThreshold
```

**动态对象年龄判定**

为了能更好地适应不同程序的内存状况，HotSpot 虚拟机并不是永远要求对象的年龄必须达到- XX：MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。

**空间分配担保**

假如在 Young GC 之后，新生代仍然有大量对象存活，就需要老年代进行分配担保，把 Survivor 无法容纳的对象直接送入老年代。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_24-知道有哪些垃圾收集器吗)24.知道有哪些垃圾收集器吗？

主要垃圾收集器如下，图中标出了它们的工作区域、垃圾收集算法，以及配合关系。

![HotSpot虚拟机垃圾收集器](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-28.png)HotSpot虚拟机垃圾收集器

这些收集器里，面试的重点是两个——**CMS**和**G1**。

- Serial 收集器

Serial 收集器是最基础、历史最悠久的收集器。

如同它的名字（串行），它是一个单线程工作的收集器，使用一个处理器或一条收集线程去完成垃圾收集工作。并且进行垃圾收集时，必须暂停其他所有工作线程，直到垃圾收集结束——这就是所谓的“Stop The World”。

Serial/Serial Old 收集器的运行过程如图：

![Serial/Serial Old收集器运行示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-29.png)Serial/Serial Old收集器运行示意图

- ParNew

ParNew 收集器实质上是 Serial 收集器的多线程并行版本，使用多条线程进行垃圾收集。

ParNew/Serial Old 收集器运行示意图如下：

![ParNew/Serial Old收集器运行示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-30.png)ParNew/Serial Old收集器运行示意图

- Parallel Scavenge

Parallel Scavenge 收集器是一款新生代收集器，基于标记-复制算法实现，也能够并行收集。和 ParNew 有些类似，但 Parallel Scavenge 主要关注的是垃圾收集的吞吐量——所谓吞吐量，就是 CPU 用于运行用户代码的时间和总消耗时间的比值，比值越大，说明垃圾收集的占比越小。

![吞吐量](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-31.png)吞吐量

- Serial Old

Serial Old 是 Serial 收集器的老年代版本，它同样是一个单线程收集器，使用标记-整理算法。

- Parallel Old

Parallel Old 是 Parallel Scavenge 收集器的老年代版本，支持多线程并发收集，基于标记-整理算法实现。

![Parallel Scavenge/Parallel Old收集器运行示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-32.png)Parallel Scavenge/Parallel Old收集器运行示意图

- CMS 收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器，同样是老年代的收集器，采用标记-清除算法。

- Garbage First 收集器

Garbage First（简称 G1）收集器是垃圾收集器的一个颠覆性的产物，它开创了局部收集的设计思路和基于 Region 的内存布局形式。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_25-什么是-stop-the-world-什么是-oopmap-什么是安全点)25.什么是 Stop The World ? 什么是 OopMap ？什么是安全点？

进行垃圾回收的过程中，会涉及对象的移动。为了保证对象引用更新的正确性，必须暂停所有的用户线程，像这样的停顿，虚拟机设计者形象描述为`Stop The World`。也简称为 STW。

在 HotSpot 中，有个数据结构（映射表）称为`OopMap`。一旦类加载动作完成的时候，HotSpot 就会把对象内什么偏移量上是什么类型的数据计算出来，记录到 OopMap。在即时编译过程中，也会在`特定的位置`生成 OopMap，记录下栈上和寄存器里哪些位置是引用。

这些特定的位置主要在：

- 1.循环的末尾（非 counted 循环）
- 2.方法临返回前 / 调用方法的 call 指令后
- 3.可能抛异常的位置

这些位置就叫作**安全点(safepoint)。** 用户程序执行时并非在代码指令流的任意位置都能够在停顿下来开始垃圾收集，而是必须是执行到安全点才能够暂停。

用通俗的比喻，假如老王去拉车，车上东西很重，老王累的汗流浃背，但是老王不能在上坡或者下坡休息，只能在平地上停下来擦擦汗，喝口水。

![老王拉车只能在平路休息](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-33.png)老王拉车只能在平路休息

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_26-能详细说一下-cms-收集器的垃圾收集过程吗)26.能详细说一下 CMS 收集器的垃圾收集过程吗？

CMS 收集齐的垃圾收集分为四步：

- **初始标记**（CMS initial mark）：单线程运行，需要 Stop The World，标记 GC Roots 能直达的对象。
- **并发标记**（（CMS concurrent mark）：无停顿，和用户线程同时运行，从 GC Roots 直达对象开始遍历整个对象图。
- **重新标记**（CMS remark）：多线程运行，需要 Stop The World，标记并发标记阶段产生对象。
- **并发清除**（CMS concurrent sweep）：无停顿，和用户线程同时运行，清理掉标记阶段标记的死亡的对象。

Concurrent Mark Sweep 收集器运行示意图如下：

![Concurrent Mark Sweep收集器运行示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-34.png)Concurrent Mark Sweep收集器运行示意图

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_27-g1-垃圾收集器了解吗)27.G1 垃圾收集器了解吗？

Garbage First（简称 G1）收集器是垃圾收集器的一个颠覆性的产物，它开创了局部收集的设计思路和基于 Region 的内存布局形式。

虽然 G1 也仍是遵循分代收集理论设计的，但其堆内存的布局与其他收集器有非常明显的差异。以前的收集器分代是划分新生代、老年代、持久代等。

G1 把连续的 Java 堆划分为多个大小相等的独立区域（Region），每一个 Region 都可以根据需要，扮演新生代的 Eden 空间、Survivor 空间，或者老年代空间。收集器能够对扮演不同角色的 Region 采用不同的策略去处理。

![G1 Heap Regions](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-35.png)G1 Heap Regions

这样就避免了收集整个堆，而是按照若干个 Region 集进行收集，同时维护一个优先级列表，跟踪各个 Region 回收的“价值，优先收集价值高的 Region。

G1 收集器的运行过程大致可划分为以下四个步骤：

- **初始标记**（initial mark），标记了从 GC Root 开始直接关联可达的对象。STW（Stop the World）执行。
- **并发标记**（concurrent marking），和用户线程并发执行，从 GC Root 开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象、
- **最终标记**（Remark），STW，标记再并发标记过程中产生的垃圾。
- **筛选回收**（Live Data Counting And Evacuation），制定回收计划，选择多个 Region 构成回收集，把回收集中 Region 的存活对象复制到空的 Region 中，再清理掉整个旧 Region 的全部空间。需要 STW。

![G1收集器运行示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-36.png)G1收集器运行示意图

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_28-有了-cms-为什么还要引入-g1)28.有了 CMS，为什么还要引入 G1？

优点：CMS 最主要的优点在名字上已经体现出来——并发收集、低停顿。

缺点：CMS 同样有三个明显的缺点。

- Mark Sweep 算法会导致内存碎片比较多
- CMS 的并发能力比较依赖于 CPU 资源，并发回收时垃圾收集线程可能会抢占用户线程的资源，导致用户程序性能下降。
- 并发清除阶段，用户线程依然在运行，会产生所谓的理“浮动垃圾”（Floating Garbage），本次垃圾收集无法处理浮动垃圾，必须到下一次垃圾收集才能处理。如果浮动垃圾太多，会触发新的垃圾回收，导致性能降低。

G1 主要解决了内存碎片过多的问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_29-你们线上用的什么垃圾收集器-为什么要用它)29.你们线上用的什么垃圾收集器？为什么要用它？

怎么说呢，虽然调优说的震天响，但是我们一般都是用默认。管你 Java 怎么升，我用 8，那么 JDK1.8 默认用的是什么呢？

可以使用命令：



```java
java -XX:+PrintCommandLineFlags -version
```

可以看到有这么一行：



```java
-XX:+UseParallelGC
```

`UseParallelGC` = `Parallel Scavenge + Parallel Old`，表示的是新生代用的`Parallel Scavenge`收集器，老年代用的是`Parallel Old` 收集器。

那为什么要用这个呢？默认的呗。

当然面试肯定不能这么答。

Parallel Scavenge 的特点是什么？

高吞吐，我们可以回答：因为我们系统是业务相对复杂，但并发并不是非常高，所以希望尽可能的利用处理器资源，出于提高吞吐量的考虑采用`Parallel Scavenge + Parallel Old`的组合。

当然，这个默认虽然也有说法，但不太讨喜。

还可以说：

采用`Parallel New`+`CMS`的组合，我们比较关注服务的响应速度，所以采用了 CMS 来降低停顿时间。

或者一步到位：

我们线上采用了设计比较优秀的 G1 垃圾收集器，因为它不仅满足我们低停顿的要求，而且解决了 CMS 的浮动垃圾问题、内存碎片问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_30-垃圾收集器应该如何选择)30.垃圾收集器应该如何选择？

垃圾收集器的选择需要权衡的点还是比较多的——例如运行应用的基础设施如何？使用 JDK 的发行商是什么？等等……

这里简单地列一下上面提到的一些收集器的适用场景：

- Serial ：如果应用程序有一个很小的内存空间（大约 100 MB）亦或它在没有停顿时间要求的单线程处理器上运行。
- Parallel：如果优先考虑应用程序的峰值性能，并且没有时间要求要求，或者可以接受 1 秒或更长的停顿时间。
- CMS/G1：如果响应时间比吞吐量优先级高，或者垃圾收集暂停必须保持在大约 1 秒以内。
- ZGC：如果响应时间是高优先级的，或者堆空间比较大。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_31-对象一定分配在堆中吗-有没有了解逃逸分析技术)31.对象一定分配在堆中吗？有没有了解逃逸分析技术？

**对象一定分配在堆中吗？** 不一定的。

随着 JIT 编译期的发展与逃逸分析技术逐渐成熟，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。其实，在编译期间，JIT 会对代码做很多优化。其中有一部分优化的目的就是减少内存堆分配压力，其中一种重要的技术叫做逃逸分析。

**什么是逃逸分析？**

**逃逸分析**是指分析指针动态范围的方法，它同编译器优化原理的指针分析和外形分析相关联。当变量（或者对象）在方法中分配后，其指针有可能被返回或者被全局引用，这样就会被其他方法或者线程所引用，这种现象称作指针（或者引用）的逃逸(Escape)。

通俗点讲，当一个对象被 new 出来之后，它可能被外部所调用，如果是作为参数传递到外部了，就称之为方法逃逸。

![逃逸](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-37.png)逃逸

除此之外，如果对象还有可能被外部线程访问到，例如赋值给可以在其它线程中访问的实例变量，这种就被称为线程逃逸。

![逃逸强度](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-38.png)逃逸强度

**逃逸分析的好处**

- 栈上分配

如果确定一个对象不会逃逸到线程之外，那么久可以考虑将这个对象在栈上分配，对象占用的内存随着栈帧出栈而销毁，这样一来，垃圾收集的压力就降低很多。

- **同步消除**

线程同步本身是一个相对耗时的过程，如果逃逸分析能够确定一个变量不会逃逸出线程，无法被其他线程访问，那么这个变量的读写肯定就不会有竞争， 对这个变量实施的同步措施也就可以安全地消除掉。

- **标量替换**

如果一个数据是基本数据类型，不可拆分，它就被称之为标量。把一个 Java 对象拆散，将其用到的成员变量恢复为原始类型来访问，这个过程就称为标量替换。假如逃逸分析能够证明一个对象不会被方法外部访问，并且这个对象可以被拆散，那么可以不创建对象，直接用创建若干个成员变量代替，可以让对象的成员变量在栈上分配和读写。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#三、jvm-调优)三、JVM 调优

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_32-有哪些常用的命令行性能监控和故障处理工具)32.有哪些常用的命令行性能监控和故障处理工具？

- 操作系统工具
  - top：显示系统整体资源使用情况
  - vmstat：监控内存和 CPU
  - iostat：监控 IO 使用
  - netstat：监控网络使用
- JDK 性能监控工具
  - jps：虚拟机进程查看
  - jstat：虚拟机运行时信息查看
  - jinfo：虚拟机配置查看
  - jmap：内存映像（导出）
  - jhat：堆转储快照分析
  - jstack：Java 堆栈跟踪
  - jcmd：实现上面除了 jstat 外所有命令的功能

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_33-了解哪些可视化的性能监控和故障处理工具)33.了解哪些可视化的性能监控和故障处理工具？

以下是一些 JDK 自带的可视化性能监控和故障处理工具：

- JConsole

![JConsole概览](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-39.png)JConsole概览

- VisualVM

![VisualVM安装插件](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-40.png)VisualVM安装插件

- Java Mission Control

![JMC主要界面](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-41.png)JMC主要界面

除此之外，还有一些第三方的工具：

- **MAT**

Java 堆内存分析工具。

- **GChisto**

GC 日志分析工具。

- **GCViewer**

`GC` 日志分析工具。

- **JProfiler**

商用的性能分析利器。

- **arthas**

阿里开源诊断工具。

- **async-profiler**

Java 应用性能分析工具，开源、火焰图、跨平台。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_34-jvm-的常见参数配置知道哪些)34.JVM 的常见参数配置知道哪些？

一些常见的参数配置：

**堆配置：**

- -Xms:初始堆大小
- -Xms：最大堆大小
- -XX:NewSize=n:设置年轻代大小
- -XX:NewRatio=n:设置年轻代和年老代的比值。如：为 3 表示年轻代和年老代比值为 1：3，年轻代占整个年轻代年老代和的 1/4
- -XX:SurvivorRatio=n:年轻代中 Eden 区与两个 Survivor 区的比值。注意 Survivor 区有两个。如 3 表示 Eden： 3 Survivor：2，一个 Survivor 区占整个年轻代的 1/5
- -XX:MaxPermSize=n:设置持久代大小

**收集器设置：**

- -XX:+UseSerialGC:设置串行收集器
- -XX:+UseParallelGC:设置并行收集器
- -XX:+UseParalledlOldGC:设置并行年老代收集器
- -XX:+UseConcMarkSweepGC:设置并发收集器

**并行收集器设置**

- -XX:ParallelGCThreads=n:设置并行收集器收集时使用的 CPU 数。并行收集线程数
- -XX:MaxGCPauseMillis=n:设置并行收集最大的暂停时间（如果到这个时间了，垃圾回收器依然没有回收完，也会停止回收）
- -XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为：1/(1+n)
- -XX:+CMSIncrementalMode:设置为增量模式。适用于单 CPU 情况
- -XX:ParallelGCThreads=n:设置并发收集器年轻代手机方式为并行收集时，使用的 CPU 数。并行收集线程数

**打印 GC 回收的过程日志信息**

- -XX:+PrintGC
- -XX:+PrintGCDetails
- -XX:+PrintGCTimeStamps
- -Xloggc:filename

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_35-有做过-jvm-调优吗)35.有做过 JVM 调优吗？

JVM 调优是一件很严肃的事情，不是拍脑门就开始调优的，需要有严密的分析和监控机制，大概的一个 JVM 调优流程图：

![JVM调优大致流程图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-42.png)JVM调优大致流程图

实际上，JVM 调优是不得已而为之，有那功夫，好好把烂代码重构一下不比瞎调 JVM 强。

但是，面试官非要问怎么办？可以从处理问题的角度来回答（对应图中事后），这是一个中规中矩的案例：电商公司的运营后台系统，偶发性的引发 OOM 异常，堆内存溢出。

1）因为是偶发性的，所以第一次简单的认为就是堆内存不足导致，单方面的加大了堆内存从 4G 调整到 8G -Xms8g。

2）但是问题依然没有解决，只能从堆内存信息下手，通过开启了-XX:+HeapDumpOnOutOfMemoryError 参数 获得堆内存的 dump 文件。

3）用 JProfiler 对 堆 dump 文件进行分析，通过 JProfiler 查看到占用内存最大的对象是 String 对象，本来想跟踪着 String 对象找到其引用的地方，但 dump 文件太大，跟踪进去的时候总是卡死，而 String 对象占用比较多也比较正常，最开始也没有认定就是这里的问题，于是就从线程信息里面找突破点。

4）通过线程进行分析，先找到了几个正在运行的业务线程，然后逐一跟进业务线程看了下代码，有个方法引起了我的注意，`导出订单信息`。

5）因为订单信息导出这个方法可能会有几万的数据量，首先要从数据库里面查询出来订单信息，然后把订单信息生成 excel，这个过程会产生大量的 String 对象。

6）为了验证自己的猜想，于是准备登录后台去测试下，结果在测试的过程中发现导出订单的按钮前端居然没有做点击后按钮置灰交互事件，后端也没有做防止重复提交，因为导出订单数据本来就非常慢，使用的人员可能发现点击后很久后页面都没反应，然后就一直点，结果就大量的请求进入到后台，堆内存产生了大量的订单对象和 EXCEL 对象，而且方法执行非常慢，导致这一段时间内这些对象都无法被回收，所以最终导致内存溢出。

7）知道了问题就容易解决了，最终没有调整任何 JVM 参数，只是做了两个处理：

- 在前端的导出订单按钮上加上了置灰状态，等后端响应之后按钮才可以进行点击
- 后端代码加分布式锁，做防重处理

这样双管齐下，保证导出的请求不会一直打到服务端，问题解决！

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_36-线上服务-cpu-占用过高怎么排查)36.线上服务 CPU 占用过高怎么排查？

问题分析：CPU 高一定是某个程序长期占用了 CPU 资源。

![CPU飙高](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-43.png)CPU飙高

1）所以先需要找出那个进程占用 CPU 高。

- top 列出系统各个进程的资源占用情况。

2）然后根据找到对应进行里哪个线程占用 CPU 高。

- top -Hp 进程 ID 列出对应进程里面的线程占用资源情况

3）找到对应线程 ID 后，再打印出对应线程的堆栈信息

- printf "%x\n" PID 把线程 ID 转换为 16 进制。
- jstack PID 打印出进程的所有线程信息，从打印出来的线程信息中找到上一步转换为 16 进制的线程 ID 对应的线程信息。

4）最后根据线程的堆栈信息定位到具体业务方法,从代码逻辑中找到问题所在。

查看是否有线程长时间的 watting 或 blocked，如果线程长期处于 watting 状态下， 关注 watting on xxxxxx，说明线程在等待这把锁，然后根据锁的地址找到持有锁的线程。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_37-内存飙高问题怎么排查)37.内存飙高问题怎么排查？

分析： 内存飚高如果是发生在 java 进程上，一般是因为创建了大量对象所导致，持续飚高说明垃圾回收跟不上对象创建的速度，或者内存泄露导致对象无法回收。

1）先观察垃圾回收的情况

- jstat -gc PID 1000 查看 GC 次数，时间等信息，每隔一秒打印一次。
- jmap -histo PID | head -20 查看堆内存占用空间最大的前 20 个对象类型,可初步查看是哪个对象占用了内存。

如果每次 GC 次数频繁，而且每次回收的内存空间也正常，那说明是因为对象创建速度快导致内存一直占用很高；如果每次回收的内存非常少，那么很可能是因为内存泄露导致内存一直无法被回收。

2）导出堆内存文件快照

- jmap -dump:live,format=b,file=/home/myheapdump.hprof PID dump 堆内存信息到文件。

3）使用 visualVM 对 dump 文件进行离线分析，找到占用内存高的对象，再找到创建该对象的业务代码位置，从代码和业务场景中定位具体问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_38-频繁-minor-gc-怎么办)38.频繁 minor gc 怎么办？

优化 Minor GC 频繁问题：通常情况下，由于新生代空间较小，Eden 区很快被填满，就会导致频繁 Minor GC，因此可以通过增大新生代空间`-Xmn`来降低 Minor GC 的频率。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_39-频繁-full-gc-怎么办)39.频繁 Full GC 怎么办？

Full GC 的排查思路大概如下：

1）清楚从程序角度，有哪些原因导致 FGC？

- **大对象**：系统一次性加载了过多数据到内存中（比如 SQL 查询未做分页），导致大对象进入了老年代。
- **内存泄漏**：频繁创建了大量对象，但是无法被回收（比如 IO 对象使用完后未调用 close 方法释放资源），先引发 FGC，最后导致 OOM.
- 程序频繁生成一些**长生命周期的对象**，当这些对象的存活年龄超过分代年龄时便会进入老年代，最后引发 FGC. （即本文中的案例）
- **程序 BUG**
- 代码中**显式调用了 gc**方法，包括自己的代码甚至框架中的代码。
- JVM 参数设置问题：包括总内存大小、新生代和老年代的大小、Eden 区和 S 区的大小、元空间大小、垃圾回收算法等等。

2）清楚排查问题时能使用哪些工具

- 公司的监控系统：大部分公司都会有，可全方位监控 JVM 的各项指标。
- JDK 的自带工具，包括 jmap、jstat 等常用命令：



```bash
# 查看堆内存各区域的使用率以及GC情况
jstat -gcutil -h20 pid 1000
# 查看堆内存中的存活对象，并按空间排序
jmap -histo pid | head -n20
# dump堆内存文件
jmap -dump:format=b,file=heap pid
```

- 可视化的堆内存分析工具：JVisualVM、MAT 等

3）排查指南

- 查看监控，以了解出现问题的时间点以及当前 FGC 的频率（可对比正常情况看频率是否正常）
- 了解该时间点之前有没有程序上线、基础组件升级等情况。
- 了解 JVM 的参数设置，包括：堆空间各个区域的大小设置，新生代和老年代分别采用了哪些垃圾收集器，然后分析 JVM 参数设置是否合理。
- 再对步骤 1 中列出的可能原因做排除法，其中元空间被打满、内存泄漏、代码显式调用 gc 方法比较容易排查。
- 针对大对象或者长生命周期对象导致的 FGC，可通过 jmap -histo 命令并结合 dump 堆内存文件作进一步分析，需要先定位到可疑对象。
- 通过可疑对象定位到具体代码再次分析，这时候要结合 GC 原理和 JVM 参数设置，弄清楚可疑对象是否满足了进入到老年代的条件才能下结论。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_40-有没有处理过内存泄漏问题-是如何定位的)40.有没有处理过内存泄漏问题？是如何定位的？

内存泄漏是内在病源，外在病症表现可能有：

- 应用程序长时间连续运行时性能严重下降
- CPU 使用率飙升，甚至到 100%
- 频繁 Full GC，各种报警，例如接口超时报警等
- 应用程序抛出 `OutOfMemoryError` 错误
- 应用程序偶尔会耗尽连接对象

严重**内存泄漏**往往伴随频繁的 **Full GC**，所以分析排查内存泄漏问题首先还得从查看 Full GC 入手。主要有以下操作步骤：

1）使用 `jps` 查看运行的 Java 进程 ID

2）使用`top -p [pid]` 查看进程使用 CPU 和 MEM 的情况

3）使用 `top -Hp [pid]` 查看进程下的所有线程占 CPU 和 MEM 的情况

4）将线程 ID 转换为 16 进制：`printf "%x\n" [pid]`，输出的值就是线程栈信息中的 **nid**。

例如：`printf "%x\n" 29471`，换行输出 **731f**。

5）抓取线程栈：`jstack 29452 > 29452.txt`，可以多抓几次做个对比。

在线程栈信息中找到对应线程号的 16 进制值，如下是 **731f** 线程的信息。线程栈分析可使用 Visualvm 插件 **TDA**。



```java
"Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007fbe2c164000 nid=0x731f runnable [0x0000000000000000]
  java.lang.Thread.State: RUNNABLE
```

6）使用`jstat -gcutil [pid] 5000 10` 每隔 5 秒输出 GC 信息，输出 10 次，查看 **YGC** 和 **Full GC** 次数。通常会出现 YGC 不增加或增加缓慢，而 Full GC 增加很快。

或使用 `jstat -gccause [pid] 5000` ，同样是输出 GC 摘要信息。

或使用 `jmap -heap [pid]` 查看堆的摘要信息，关注老年代内存使用是否达到阀值，若达到阀值就会执行 Full GC。

7）如果发现 `Full GC` 次数太多，就很大概率存在内存泄漏了

8）使用 `jmap -histo:live [pid]` 输出每个类的对象数量，内存大小(字节单位)及全限定类名。

9）生成 `dump` 文件，借助工具分析哪 个对象非常多，基本就能定位到问题在那了

使用 jmap 生成 dump 文件：



```java
# jmap -dump:live,format=b,file=29471.dump 29471
Dumping heap to /root/dump ...
Heap dump file created
```

10）dump 文件分析

可以使用 **jhat** 命令分析：`jhat -port 8000 29471.dump`，浏览器访问 jhat 服务，端口是 8000。

通常使用图形化工具分析，如 JDK 自带的 **jvisualvm**，从菜单 > 文件 > 装入 dump 文件。

或使用第三方式具分析的，如 **JProfiler** 也是个图形化工具，**GCViewer** 工具。Eclipse 或以使用 MAT 工具查看。或使用在线分析平台 **GCEasy**。

**注意**：如果 dump 文件较大的话，分析会占比较大的内存。

11）在 dump 文析结果中查找存在大量的对象，再查对其的引用。

基本上就可以定位到代码层的逻辑了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_41-有没有处理过内存溢出问题)41.有没有处理过内存溢出问题？

内存泄漏和内存溢出二者关系非常密切，内存溢出可能会有很多原因导致，内存泄漏最可能的罪魁祸首之一。

排查过程和排查内存泄漏过程类似。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#四、虚拟机执行)四、虚拟机执行

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_42-能说一下类的生命周期吗)42.能说一下类的生命周期吗？

一个类从被加载到虚拟机内存中开始，到从内存中卸载，整个生命周期需要经过七个阶段：加载 （Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化 （Initialization）、使用（Using）和卸载（Unloading），其中验证、准备、解析三个部分统称为连接（Linking）。

![类的生命周期](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-44.png)类的生命周期

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_43-类加载的过程知道吗)43.类加载的过程知道吗？

加载是 JVM 加载的起点，具体什么时候开始加载，《Java 虚拟机规范》中并没有进行强制约束，可以交给虚拟机的具体实现来自由把握。

在加载过程，JVM 要做三件事情：

![加载](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-45.png)加载

- 1）通过一个类的全限定名来获取定义此类的二进制字节流。
- 2）将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 3）在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的访问入口。

加载阶段结束后，Java 虚拟机外部的二进制字节流就按照虚拟机所设定的格式存储在方法区之中了，方法区中的数据存储格式完全由虚拟机实现自行定义，《Java 虚拟机规范》未规定此区域的具体数据结构。

类型数据妥善安置在方法区之后，会在 Java 堆内存中实例化一个 java.lang.Class 类的对象， 这个对象将作为程序访问方法区中的类型数据的外部接口。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_44-类加载器有哪些)44.类加载器有哪些？

主要有四种类加载器:

- **启动类加载器**(Bootstrap ClassLoader)用来加载 java 核心类库，无法被 java 程序直接引用。
- **扩展类加载器**(extensions class loader):它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。
- **系统类加载器**（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。
- **用户自定义类加载器** (user class loader)，用户通过继承 java.lang.ClassLoader 类的方式自行实现的类加载器。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_45-什么是双亲委派机制)45.什么是双亲委派机制？

![双亲委派模型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-46.png)双亲委派模型

双亲委派模型的工作过程：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求时，子加载器才会尝试自己去完成加载。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_46-为什么要用双亲委派机制)46.为什么要用双亲委派机制？

答案是为了保证应用程序的稳定有序。

例如类 java.lang.Object，它存放在 rt.jar 之中，通过双亲委派机制，保证最终都是委派给处于模型最顶端的启动类加载器进行加载，保证 Object 的一致。反之，都由各个类加载器自行去加载的话，如果用户自己也编写了一个名为 java.lang.Object 的类，并放在程序的 ClassPath 中，那系统中就会出现多个不同的 Object 类。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_47-如何破坏双亲委派机制)47.如何破坏双亲委派机制？

如果不想打破双亲委派模型，就重写 ClassLoader 类中的 fifindClass()方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。而如果想打破双亲委派模型则需要重写 loadClass()方法。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_48-历史上有哪几次双亲委派机制的破坏)48.历史上有哪几次双亲委派机制的破坏？

双亲委派机制在历史上主要有三次破坏：

![双亲委派模型的三次破坏](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-47.png)双亲委派模型的三次破坏

> **第一次破坏**

双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前——即 JDK 1.2 面世以前的“远古”时代。

由于双亲委派模型在 JDK 1.2 之后才被引入，但是类加载器的概念和抽象类 java.lang.ClassLoader 则在 Java 的第一个版本中就已经存在，为了向下兼容旧代码，所以无法以技术手段避免 loadClass()被子类覆盖的可能性，只能在 JDK 1.2 之后的 java.lang.ClassLoader 中添加一个新的 protected 方法 findClass()，并引导用户编写的类加载逻辑时尽可能去重写这个方法，而不是在 loadClass()中编写代码。

> **第二次破坏**

双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷导致的，如果有基础类型又要调用回用户的代码，那该怎么办呢？

例如我们比较熟悉的 JDBC:

各个厂商各有不同的 JDBC 的实现，Java 在核心包`\lib`里定义了对应的 SPI，那么这个就毫无疑问由`启动类加载器`加载器加载。

但是各个厂商的实现，是没办法放在核心包里的，只能放在`classpath`里，只能被`应用类加载器`加载。那么，问题来了，启动类加载器它就加载不到厂商提供的 SPI 服务代码。

为了解决这个问题，引入了一个不太优雅的设计：线程上下文类加载器 （Thread Context ClassLoader）。这个类加载器可以通过 java.lang.Thread 类的 setContext-ClassLoader()方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

JNDI 服务使用这个线程上下文类加载器去加载所需的 SPI 服务代码，这是一种父类加载器去请求子类加载器完成类加载的行为。

> **第三次破坏**

双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求而导致的，例如代码热替换（Hot Swap）、模块热部署（Hot Deployment）等。

OSGi 实现模块化热部署的关键是它自定义的类加载器机制的实现，每一个程序模块（OSGi 中称为 Bundle）都有一个自己的类加载器，当需要更换一个 Bundle 时，就把 Bundle 连同类加载器一起换掉以实现代码的热替换。在 OSGi 环境下，类加载器不再双亲委派模型推荐的树状结构，而是进一步发展为更加复杂的网状结构。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_49-你觉得应该怎么实现一个热部署功能)49.你觉得应该怎么实现一个热部署功能？

我们已经知道了 Java 类的加载过程。一个 Java 类文件到虚拟机里的对象，要经过如下过程:首先通过 Java 编译器，将 Java 文件编译成 class 字节码，类加载器读取 class 字节码，再将类转化为实例，对实例 newInstance 就可以生成对象。

类加载器 ClassLoader 功能，也就是将 class 字节码转换到类的实例。在 Java 应用中，所有的实例都是由类加载器，加载而来。

一般在系统中，类的加载都是由系统自带的类加载器完成，而且对于同一个全限定名的 java 类（如 com.csiar.soc.HelloWorld），只能被加载一次，而且无法被卸载。

这个时候问题就来了，如果我们希望将 java 类卸载，并且替换更新版本的 java 类，该怎么做呢？

既然在类加载器中，Java 类只能被加载一次，并且无法卸载。那么我们是不是可以直接把 Java 类加载器干掉呢？答案是可以的，我们可以自定义类加载器，并重写 ClassLoader 的 findClass 方法。

想要实现热部署可以分以下三个步骤：

- 1）销毁原来的自定义 ClassLoader
- 2）更新 class 类文件
- 3）创建新的 ClassLoader 去加载更新后的 class 类文件。

到此，一个热部署的功能就这样实现了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/jvm.html#_50-tomcat-的类加载机制了解吗)50.Tomcat 的类加载机制了解吗？

Tomcat 是主流的 Java Web 服务器之一，为了实现一些特殊的功能需求，自定义了一些类加载器。

Tomcat 类加载器如下：

![Tomcat类加载器](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-48.png)Tomcat类加载器

Tomcat 实际上也是破坏了双亲委派模型的。

Tomact 是 web 容器，可能需要部署多个应用程序。不同的应用程序可能会依赖同一个第三方类库的不同版本，但是不同版本的类库中某一个类的全路径名可能是一样的。如多个应用都要依赖 hollis.jar，但是 A 应用需要依赖 1.0.0 版本，但是 B 应用需要依赖 1.0.1 版本。这两个版本中都有一个类是 com.hollis.Test.class。如果采用默认的双亲委派类加载机制，那么无法加载多个相同的类。

所以，Tomcat 破坏了**双亲委派原则**，提供隔离的机制，为每个 web 容器单独提供一个 WebAppClassLoader 加载器。每一个 WebAppClassLoader 负责加载本身的目录下的 class 文件，加载不到时再交 CommonClassLoader 加载，这和双亲委派刚好相反。

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。