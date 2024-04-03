---
layout: post
title: Java并发编程实战
categories: [并发]
description: 极客时间课程笔记
keywords: Java, 并发, 编程, 课程笔记
---

# Java并发编程实战

## 04 | 互斥锁（下）-用一个保护多个资源

### 锁对象的选择

编程中应在保证线程安全的前提下使用最小粒度的锁，如下案例。账户类中有两个资源，余额和密码同时还有分别对这两个资源进行修改和查询的操作。

要实现线程安全，加锁方式有很多，以锁粒度大到小分别为：

- 当前类字节码对象
- 当前对象
- 为余额和密码分别建一个锁对象

以线程安全为标准，以上3中加锁方式都是线程安全的，但粒度不同，应使用粒度最小的方式。

```java

class Account {
  // 锁：保护账户余额
  private final Object balLock = buyiyw Object();
  // 账户余额  
  private Integer balance;
  // 锁：保护账户密码
  private final Object pwLock = new Object();
  // 账户密码
  private String password;
  // 取款
  void withdraw(Integer amt) {
    synchronized(balLock) {
      if (this.balance > amt){
        this.balance -= amt;
      }
    }
  } 
  // 查看余额
  Integer getBalance() {
    synchronized(balLock) {
      return balance;
    }
  }
  // 更改密码
  void updatePassword(String pw){
    synchronized(pwLock) {
      this.password = pw;
    }
  } 
  // 查看密码
  String getPassword() {
    synchronized(pwLock) {
      return password;
    }
  }
}
```

### 死锁

- 死锁是使用细粒度锁锁带来的问题
- 死锁一旦发生无法解决，只能预防

### Coffman 条件

死锁发生需要同时满足的四个条件，破环任何一个条件即可预防死锁

#### 内容

- 互斥：共享资源X和Y（或其锁对象）只能被一个线程占用
- 占有且等待：一个线程在占有X的同时等待Y且不会释放X
- 不可抢占：线程已占有的资源不能被其他线程抢占
- 循环等待：A线程持有X在等待Y，而B线程持有Y在等X

#### 预防死锁

破坏Coffman条件中中任何一个即可预防死锁

- 互斥：作为保证原子性的手段，**不能破坏**
- 占有且等待：一次性获取两个锁，避免持有一个等待另一个
- 不可抢占：J.U.C保重的``Lock``对象提供等待可中断的功能
- 循环等待：按顺序加锁，这样能先加锁成功说明下一个锁必然是未被持有的状态

### 等待通知模式

```java

class Allocator {
  private List<Object> als;
  // 一次性申请所有资源
  synchronized void apply(
    Object from, Object to){
    // 经典写法
    while(als.contains(from) ||
         als.contains(to)){
      try{
        wait();
      }catch(Exception e){
      }   
    } 
    als.add(from);
    als.add(to);  
  }
  // 归还资源
  synchronized void free(
    Object from, Object to){
    als.remove(from);
    als.remove(to);
    notifyAll();
  }
}
```

### wati()和sleep()的区别

- wait()会释放锁，阻塞自己并让出执行权给其它所有线程抢占，sleep(x)不会释放锁，虽然也会阻塞自己，让出执行权，但由于不释放锁所以使用相同锁的其他线程没有执行机会

## 07 | 安全性、活跃性以及性能问题

### 安全性问题

#### 数据竞争

仅存在数据竞争时才需要考虑并发问题

#### 竞态条件 

程序的执行结果依赖程序的执行顺序

### 活跃性问题

#### 死锁

#### 活锁

没有发生阻塞但携程仍然执行不下去，如两个线程在获取同一个锁之前都在等待而导致无法继续执行。解决方案是设置随机等待时间

#### 饥饿

指由于资源有限、无法访问到需要的资源而导致线程无法执行。如线程倒霉一直拿不到执行权，线程执行时间过长等情况。解决方案有分配更多资源，缩短线程执行时间和公平锁。其中公平锁应用最广泛。

### 性能问题

#### 阿姆达尔定律

$$
S=\frac{1}{(1-p)+\frac{p}{n}}
$$

- p表示并行百分比，如10%表示并行部分占全部的10%
- S表示加速比，如10表示相对于完全串行理论最多提高10倍
- n表示CPU核数

#### 性能指标

- 吞吐量
- 延迟
- 并发量

## 08 | 管程：并发编程的万能钥匙

- 管程使用封装共享变量及其对应操作的方式解决互斥问题
- Java中每一个锁都有其对应的管程（监视器）
- 管程模型有入口等待队列、条件变量和条件变量等待队列
- 入口等待队列是管程外的概念，是其他线程的等待队列（在等待这个管程持有的锁的队列）
- 条件变量和条件变量等待队列是管程内概念，用以解决线程间同步的问题
- 条件变量和条件变量等待队列可以有多个，至少一个
- 条件变量等待队列中的线程被唤醒会重新进入入口等待队列排队
- Java的管程实现方案中，等待的线程被重新唤醒将从``wait()``处开始执行，而不是从头开始，所以需要使用``while``循环检测条件，避免唤醒后条件已不满足

## 09 | Java线程（上）：Java线程的生命周期

### 线程状态

- Java中Thread类定义了``NEW`` ``RUNNABLE`` ``BLOCKED`` ``WAITING`` ``TIMED_WAITING`` ``TERMINATED``六个状态
- 操作系统的线程中，分新建、就绪、执行、休眠和终止五个状态，其中``BLOCKED`` ``WAITING`` ``TIMED_WAITING`` 统称为休眠状态，``RUNNABLE`` 细分为就绪和执行

### 线程状态切换

- ``RUNNABLE``->``BLOCKED``：当线程等待锁时，状态切换。对于调用阻塞式API，操作系统层面会转为休眠，但jvm中Thread的状态仍会为``RUNNABLE``
- ``RUNNABLE``->``WAITING``：
  - Object.wait()
  - Thread.join()
  - LockSupport.park()
- ``RUNNABLE``->``TIMED_WAITING`` ：
  - 调用带超时参数的 Thread.sleep(long millis)
  - 调用带超时参数的 Object.wait(long timeout)
  - 调用带超时参数的 Thread.join(long millis)
  - 调用带超时参数的 LockSupport.parkNanos(Object blocker, long deadline) 
  - 调用带超时参数的 LockSupport.parkUntil(long deadline)
- ``NEW`` ->``RUNNABLE`` 
- ``RUNNABLE`` ->``TERMINATED``

### 中断线程

- interrupt()设置中断标志，如何处理有程序逻辑决定，可主动监测标志处理或使用异常处理
- 线程 A 处于 WAITING、TIMED_WAITING 状态时，如果其他线程调用线程 A 的 interrupt() 方法，会使线程 A 返回到 RUNNABLE 状态，同时线程 A 的代码会触发 InterruptedException 异常
- 当线程 A 处于 RUNNABLE 状态时，并且阻塞在 java.nio.channels.InterruptibleChannel 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 会触发 java.nio.channels.ClosedByInterruptException 这个异常
- 而阻塞在 java.nio.channels.Selector 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 的 java.nio.channels.Selector 会立即返回。
- 结合isInterrupted()的主动检测模式手动实现中断
- Thread.stop()方法会强制终止线程，可能导致资源获取了但不释放的问题，如IO流和ReetrantLock

## 10 | Java线程（中）：创建多少线程才是合适的

- 如果 CPU 和 I/O 设备的利用率都很低，那么可以尝试通过增加线程来提高吞吐量
- 单核理论最佳线程数 =1 +（I/O 耗时 / CPU 耗时）（具体问题具体分析，这理想情况）
- 多核最佳线程数 =CPU 核数 * [ 1 +（I/O 耗时 / CPU 耗时）]

## 11 | Java线程（下）：为什么局部变量是线程安全的？

## 12 | 如何用面向对象思想写好并发程序？

### 设计原则

- 封装共享变量：将共享变量作为对象属性封装在内部，对所有公共方法制定并发访问策略
- 识别共享变量间的约束条件
- 制定并发访问策略

### 注意

- 优先使用成熟的工具类
- 迫不得已时才使用低级的同步原语
- 避免过早优化

## 14 | Lock和Condition（上）：隐藏在并发包中的管程

### Lock

- JVM提供设计Lock的目的为了解决死锁问题，由于管程原语不能直接破环Coffman条件，所以新设计的Lock接口提供了3个相关方法支持破环不可抢占条件
- ReentrantLock使用AQS实现同步，其加锁和解锁会自动对内部被volatile修饰的成员变量进行读写，结合先行发生原则中的程序次序原则、volatile的特殊规则和传递性保证可见性
- 可重入锁
- 公平锁和非公平锁（使用ReetrantLock的构造函数构造）

### 加锁的最佳实践

- 永远只在更新对象的成员变量时加锁
- 永远只在访问可变的成员变量时加锁
- 永远不在调用其他对象的方法时加锁

### 活锁分析

```java

class Account {
  private int balance;
  private final Lock lock
          = new ReentrantLock();
  // 转账
  void transfer(Account tar, int amt){
    while (true) {
      if(this.lock.tryLock()) {
        try {
          if (tar.lock.tryLock()) {
            try {
              this.balance -= amt;
              tar.balance += amt;
            } finally {
              tar.lock.unlock();
            }
          }//if
        } finally {
          this.lock.unlock();
        }
      }//if
    }//while
  }//transfer
}
```

- 当两个线程互相向对方转账时，可能都会同时获取了自己的账户的锁，但获取对方账户时失败，然后循环继续一直重复
- 解决方案，给两个tryLock添加随机的时间参数，避免总是同时获取资源

### 书本推荐

- 《Java 并发编程：设计原则与模式》
- 《Java 安全编码标准》

## 15 | Lock和Condition（下）：Dubbo如何用管程实现异步转同步？

- 介绍了Condition的好处
- Dubbo部分不理解怎么做到的接收时还是调这个类的方法

## 16 | Semaphore：如何快速实现一个限流器？

- acquire()时信号量中的初始值 -1 当某线程调用或值小于0时阻塞并放入等待队列
- release()时信号量的初始值+1
- 应用场景：限流器、池化技术

## 17 | ReadWriteLock：如何快速实现一个完备的缓存？

### 读写锁

- 可多个线程同时读，只允许一个线程写，写的时候不能读，读的时候也不能写
- 适合读多写少的场景，如缓存
- 若读的量过大，可能写一直拿不到锁
- 支持条件变量
- 支持阻塞锁、非阻塞锁、超时和不超时版本的可非阻塞锁，可中断锁
- **支持**锁降级，**不支持**锁升级

### 锁升级和锁降级

- 读写锁在持有读锁的情况下再获取写锁叫锁升级，JDK的读写锁不支持锁升级

### 缓存数据一致性解决方案

- 设置数据失效时间
- 监控数据库数据变化
- 数据和缓存双写

## 18 | StampedLock：有没有比读写锁更快的锁？

### StampedLock

- StampedLock由版本（stampe）和模式（写锁、悲观读锁、乐观读）组成
- 支持写锁、悲观读锁、乐观读三种模式
- 相比读写锁，加锁会返回stamp，解锁需要传入stamp
- 乐观读，很容易被破环，仅适合用在很短的只读代码上，场景合适可解决读写锁的写锁饥饿问题
- 支持阻塞锁、非阻塞锁、超时和不超时版本的可非阻塞锁，可中断锁
- 支持锁升级、锁降级
- api支持转读写锁
- 不支持条件变量

## 19 | CountDownLatch和CyclicBarrier：如何让多线程步调一致？

CountDownLatch

- 一个线程等多个线程
- 计数至0寿命终结

CyclicBarrier

- 多个线程间协同
- 计数至0可自动重置

## 20 | 并发容器：都有哪些“坑”需要我们填？

### 同步容器及其注意事项

- 组合操作需要注意竞态条件问题
- 迭代遍历容器时需要对该容器的引用加锁，否则迭代器可绕过安全容器的锁直接操作容器数据

### 并发容器及其注意事项

#### List：CopyOnWriteArrayList

- 无锁实现
- 会有短暂的数据不同步
- 适合读多写少的场景
- 迭代器不支持增删改

#### Map：ConcurrentHashMap和ConcurrentSkipListMap

- ConcurrentHashMap的key是无序的
- ConcurrentSkipListMap的key是有序的
- 都不支持key和value为null
- 跳表插入、删除、查询操作平均的时间复杂度是 O(log n)

#### Set：CopyOnWriteArraySet和ConcurrentSkipListSet

- 同上

#### Queue

##### 分类

- 阻塞队列 / 非阻塞队列
- 单端队列 / 双端队列

##### 单端阻塞队列

- ArrayBlockingQueue
- LinkedBlockingQueue
- SynchronousQueue
- LinkedTransferQueue
- PriorityBlockingQueue
- DelayQueue

##### 双端阻塞队列

- LinkedBlockingDeque

##### 单端非阻塞队列

- ConcurrentLinkedQueue

##### 双端非阻塞队列

- ConcurrentLinkedDeque

#### 其他

- 1.8以前的版本并发执行put会在扩容时形成环形链表，造成死循环，后1.8中优化成红黑树解决了这个问题


## 21 | 原子类：无锁工具类的典范

### 无锁方案的实现

- CAS
- ABA问题：使用版本号解决，带版本的原子类

### 原子类

- get()
- compareAndSet(boolean expect, boolean update)
- weakCompareAndSet(boolean expect, boolean update)
- set(boolean newValue)
- lazySet(boolean newValue)
- getAndSet(boolean newValue)

基本类型原子类

带版本的原子类

基本类型数组原子类

带标记的数组原子类

原子对象属性更新器

- 属性必须带volatile

## 22 | Executor与线程池：如何创建正确的线程池？

### 线程池的构造参数

```java
ThreadPoolExecutor( int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
```

### 拒绝策略

- CallerRunsPolicy：提交任务的线程自己去执行该任务
- AbortPolicy：默认的拒绝策略，会 throws RejectedExecutionException
- DiscardPolicy：直接丢弃任务，没有任何异常抛出
- DiscardOldestPolicy：丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列

### 注意

- 强烈建议使用有界队列
- 默认拒绝策略要慎重使用

### 推荐书目

- 《Java并发编程实战》第七章 取消与关闭 - 7.3处理非正常的线程终止

## 23 | Future：如何用多线程实现最优的“烧水泡茶”程序？

获取任务执行结果的方式

- Callable接口配合线程池
- FutureTask类配合线程池或Thread

## 24 | CompletableFuture：异步编程没那么难

- 串行
- and
- or
- 异常处理

## 26 | Fork/Join：单机版的MapReduce

## 28 | 不变模式和享元模式
## 29 | Copy-on-Write模式：不是延时策略的COW
## 30 | 线程本地存储模式：没有共享，就没有伤害



- 线程池死锁



《图解 Java 多线程设计模式》
