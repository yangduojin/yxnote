# JUC & deadlock

- [JUC & deadlock](#juc--deadlock)
  - [线程 进程](#线程-进程)
  - [Volatile](#volatile)
  - [JMM](#jmm)
    - [三大特性](#三大特性)
  - [锁分类](#锁分类)
  - [AQS](#aqs)
  - [CAS算法](#cas算法)
  - [synchronized & Lock](#synchronized--lock)
  - [Threadlocal](#threadlocal)
  - [DeadLock概念](#deadlock概念)
  - [死锁产生的四个必要条件](#死锁产生的四个必要条件)
  - [排错](#排错)

## 线程 进程

进程:操作系统动态执行的基本单元,在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元。进程彼此之间不能共享内存，线程可以

线程:通常都是把进程作为分配资源的基本单位,线程作为独立运行和独立调度的基本单位，由于线程比进程更小，基本上不拥有系统资源，故对它的调度所付出的开销就会小得多，能更高效的提高系统多个程序间并发执行的程度。

进程是程序在执行过程中分配和管理资源的基本单位。线程是进程的一个实体，是CPU调度和分派的基本单位，它是比进程更小的独立运行的基本单位。线程自己基本上不拥有系统资源，只拥有一点在运行中必不可少的资源(程序计数器、一组寄存器和栈)，它与进程的其他线程共享进程的资源。

进程和线程是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其他进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但是线程没有单独的地址空间。一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大。对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。

进程的通信包含资源的通信和状态的同步。因为线程资源是共享的，所以线程的通信基本就是线程的同步。

qq.exe 就是一个进程， 他多个聊天窗口就是多个线程,Word：一个进程，断电关闭后重新打开能恢复之前未保存的文档，也会检查语法，两个线程，容灾备份，语法检查

## Volatile

- 保证可见性
- 禁止指令排序
- 不保证原子性

## JMM

- 线程解锁前，必须把共享变量的值刷新回主内存
- 线程加锁前，必须读取主内存的最新值到自己的工作内存
- 加锁解锁是同一把锁
- 由于 JVM 运行程序的实体是线程，而每个线程创建时 JVM 都会为其创建一个工作内存，工作内存是每个线程的私有数据区域，而 Java 内存模型中规定所有变量的储存在主内存，主内存是共享内存区域，所有的线程都可以访问，但线程对变量的操作（读取赋值等）必须都工作内存进行看。

- 首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷贝，前面说过，工作内存是每个线程的私有数据区域，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成。

![java 内存模型](/Jvav/img/jmm.png)

### 三大特性

- 可见性
- 原子性
- 有序性

可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

1. 为 uniqueInstance 分配内存空间
1. 初始化 uniqueInstance
1. 将 uniqueInstance 指向分配的内存地址

由于 JVM 具有指令重排的特性，执行顺序有可能变成 1>3>2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。  
不保证原子性  
保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

## 锁分类

1. 乐观锁(适合读取操作多)
   - 原理：自旋(for(;;)比while快,for一个指令,while4个指令) + CAS 或 版本号控制
   - 举例：JUC中的原子类
2. 悲观锁(适合写入操作多)
   1. 隐式锁synchronized
      - 无锁 01 （不锁资源，多线程只有一个成功，其余等待）
      - 偏向锁 01 （同一线程执行同步资源时自动获得资源）
      - 轻量级锁 00 (多线程竞争同步资源，失败线程自旋等待锁释放)
      - 重量级锁 10 (多线程竞争同步资源，失败线程阻塞等待唤醒)
   2. 显示锁：基于AQS的锁
      1. 按同一线程是否可重入分
         1. 不可重入锁 （调用内层方法要释放外层方法的所，否则死锁）
         2. ReentrantLock （同一线程在外层方法获得了锁，调用内层方法时，自动获取内层的锁(前提内外方法的锁得是同一个对象或者class对象,即线程可以进入任何一个他已经拥有的锁所同步着的代码块(两个代码块用的同一个锁,防止死锁))
            1. 公平锁 (排队)
            2. 非公平锁 (插队，失败再排队)
      2. 流量拆分
         1. 读锁-->原理：共享锁 (只能读数据，不能修改)
         2. 写锁-->原理：排他锁 (写数据，同一个资源，读锁全部释放后，才能获取写锁)
对于同步的方法或者代码块来说，必须获得对象锁才能够进入同步方法或者代码块进行操作;  
如果采用method级别的同步，则对象锁即为method所在的对象，如果是静态方法，对象锁即指method所在的Class对象(唯一)；

## AQS

AQS使用一个FIFO的队列表示排队等待锁的线程，它维护一个status的变量，每个节点维护一个waitstatus的变量，当线程获取到锁的时候，队列的status置为1，此线程执行完了，那么它的waitstatus为-1；队列头部的线程执行完毕之后，它会调用它的后继的线程(百度面试)。

队列头节点称作“哨兵节点”或者“哑节点”，它不与任何线程关联。其他的节点与等待线程关联，每个节点维护一个等待状态waitStatus。如图

![AQS双向链表](/Jvav/img/AQSFIFO.png)

AQS中还有一个表示状态的字段state，例如ReentrantLocky用它表示线程重入锁的次数，Semaphore用它表示剩余的许可数量，FutureTask用它表示任务的状态。对state变量值的更新都采用CAS操作保证更新操作的原子性。

AbstractQueuedSynchronizer继承了AbstractOwnableSynchronizer，这个类只有一个变量：exclusiveOwnerThread，表示当前占用该锁的线程，并且提供了相应的get，set方法。

理解AQS可以帮助我们更好的理解JCU包中的同步容器。

## CAS算法

1. 原理
   - CAS有3个操作数，内存值V(栈中的值?)，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做，就继续重试；错误发生在一个线程v栈下面的值可能是W，另一个线程将W换成了C,D.上一步依然比较V,A相等，将A出栈，顶栈指向了W,而不是CD
   - CAS（比较并交换）是CPU指令级的操作，只有一步原子操作，所以非常快。而且CAS避免了请求操作系统来裁定锁的问题，不用麻烦操作系统，直接在CPU内部就搞定了
2. 缺点
   - CPU开销较大
   - 不能保证代码块的原子性:基于硬件级别的原子操作
   - ABA问题：使用AtomicInteger 可能导致A->B->A问题，使用这个类解决AtomicStampedReference\<V>,AtomicStampedReference(V initialRef, int initialStamp) 带版本号解决ABA问题
3. ``for(;;)``和``while(true)``的区别: ``for(; ;)``语句不仅指令少、不占用寄存器，还没有跳转判断指令，比``while(1)``好。
4. CAS与synchornized比较: 如果代码块中逻辑较简单,Synchronized涉及用户态内核态转换,消耗较大,但是可以保证原子性,不会指令重排
5. 线程数相对较少，CAS实现比较快，性能优于synchronized,当线程数多于8后，CAS实现明显开始下降，反而时间消耗高于synchronized；  
synchronized是java提供的又简单方便，性能优化又非常好的功能，建议大家常用；CAS的话，线程数大于一定数量的话，多个线程在循环调用CAS接口，虽然不会让其他线程阻塞，但是这个时候竞争激烈，会导致CPU到达100%，同时比较耗时间，所以性能就不如synchronized了。

## synchronized & Lock

synchronized 锁类型 可重入 不可中断 非公平 而 lock 是： 可重入 可判断 可公平（两者皆可),竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。

- 可重入锁。可重入锁是指同一个线程可以多次获取同一把锁。ReentrantLock和synchronized都是可重入锁。
- 可中断锁。可中断锁是指线程尝试获取锁的过程中，是否可以响应中断。synchronized是不可中断锁，而ReentrantLock则提供了中断功能。 ``lock.lockInterruptibly()``
- 公平锁与非公平锁。公平锁是指多个线程同时尝试获取同一把锁时，获取锁的顺序按照线程达到的顺序，而非公平锁则允许线程“插队”。synchronized是非公平锁，而ReentrantLock的默认实现是非公平锁，但是也可以设置为公平锁。
- CAS操作(CompareAndSwap)。CAS操作简单的说就是比较并交换。CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。” Java并发包(java.util.concurrent)中大量使用了CAS操作,涉及到并发的地方都调用了sun.misc.Unsafe类方法进行CAS操作。

## Threadlocal

- 目的：通过本地化资源来避免共享，避免多线程竞争导致的锁等消耗，
- 强调：并不是所有东西都要避免共享来解决，比如需要累加同一个标量时，就不能避免共享
- 如何本地化资源：在线程对象内部搞个 map，把 ThreadLocal 对象自身作为 key，把它的值作为 map 的值。在线程对象内部搞个 map，把 ThreadLocal 对象自身作为 key，把它的值作为 map 的值。
- 每个线程维护自己的变量，互不干扰，JDK就是这样实现的！
- 这个 map 是 ThreadLocal 的静态内部类，记住这个变量的名字 threadLocals
- 内部类这个东西是编译层面的概念，就像语法糖一样，经过编译器之后其实内部类会提升为外部顶级类，和平日里外部定义的类没有区别，也就是说在 JVM 中是没有内部类这个概念的。
- 一般情况下非静态内部类用在内部类，跟其他类无任何关联，专属于这个外部类使用，并且也便于调用外部类的成员变量和方法，比较方便。
- 而静态内部类其实就等于一个顶级类，可以独立于外部类使用，所以更多的只是表明类结构和命名空间。
- 这样定义的用意就是说明 ThreadLocalMap 是和 ThreadLocal 强相关的，专用于保存线程本地变量。
- 最佳实践是用完了之后，调用一下 ThreadLocal.remove 方法，手工把 Entry 清理掉，这样就不会发生内存泄漏了！(放在finally里面)
- tomcat 这种隐式线程池，第一次调用执行 Threadlocal 之后，如果没有显示调用 remove 方法，则这个 Entry 还是存在的，那么下次这个线程再执行任务的时候，不会再调用 withInitial 方法，也就是说会拿到上一次执行的值。  
实战中:  每一个请求进来到返回客户端都是同一个thread,要资源共享在这个thread里面,用threadlocal Map<Thread,Object>,多个线程用自己的

![锁膨胀](/Jvav/img/deadLock.png)

## DeadLock概念

死锁是指两个或多个以上的进程在执行过程中，因争夺资源而造成一种互相等待的现象，若无外力干涉那他们都将无法推进下去。如果资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。

![死锁产生原因](/Jvav/img/deadLock.png)

1. 原因
   - 系统资源不足
   - 进程运行推进的顺序不对
   - 资源分配不当

## 死锁产生的四个必要条件

- 互斥
  - 解决方法：把互斥的共享资源封装成可同时访问
- 占有且等待
  - 解决方法：进程请求资源时，要求它不占有任何其它资源，也就是它必须一次性申请到所有的资源，这种方式会导致资源效率低。
- 非抢占式
  - 解决方法：如果进程不能立即分配资源，要求它不占有任何其他资源，也就是只能够同时获得所有需要资源时，才执行分配操作
- 循环等待
  - 解决方法：对资源进行排序，要求进程按顺序请求资源。

```java
public class testReturn {
    public static void main(String[] args) {
        String lockA ="lockA";
        String lockB = "lockB";
        new Thread(new HoldLockThread(lockA,lockB),"t1").start();
        new Thread(new HoldLockThread(lockB,lockA),"t2").start();
    }
}

class HoldLockThread implements Runnable{
    private String lockA;
    private String lockB;

    public HoldLockThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA){
            System.out.println(Thread.currentThread().getName() +"\t has "+ lockA + "\t try require" + lockB );

            try {TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) {e.printStackTrace();}

            synchronized (lockB){
                System.out.println(Thread.currentThread().getName() +"\t has "+ lockB + "\t try require" + lockA );
            }}}}
// 打印结果
// t1 自己持有lockA 尝试获取：lockB
// t2 自己持有lockB 尝试获取：lockA
```

## 排错

jps -l 查看java进程,选择对应的id号
jstack  7560   # 后面参数是 jps输出的该类的pid

最后一行可以看见deadlock

```java
Found one Java-level deadlock:
=============================
"t2":
  waiting to lock monitor 0x000000001cfc0de8 (object 0x000000076b696e80, a java.lang.String),
  which is held by "t1"
"t1":
  waiting to lock monitor 0x000000001cfc3728 (object 0x000000076b696eb8, a java.lang.String),
  which is held by "t2"

Java stack information for the threads listed above:
===================================================
"t2":
        at com.moxi.interview.study.Lock.HoldLockThread.run(DeadLockDemo.java:42)
        - waiting to lock <0x000000076b696e80> (a java.lang.String)
        - locked <0x000000076b696eb8> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:745)
"t1":
        at com.moxi.interview.study.Lock.HoldLockThread.run(DeadLockDemo.java:42)
        - waiting to lock <0x000000076b696eb8> (a java.lang.String)
        - locked <0x000000076b696e80> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```
