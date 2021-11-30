## 线程 进程

进程:操作系统动态执行的基本单元,在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元。进程彼此之间不能共享内存，线程可以

线程:通常都是把进程作为分配资源的基本单位,线程作为独立运行和独立调度的基本单位，由于线程比进程更小，基本上不拥有系统资源，故对它的调度所付出的开销就会小得多，能更高效的提高系统多个程序间并发执行的程度。

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

![](/Jvav/img/jmm.png)


### 三大特性：

- 可见性
- 原子性
- 有序性


	可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。
		a. 为 uniqueInstance 分配内存空间
		b. 初始化 uniqueInstance
		c. 将 uniqueInstance 指向分配的内存地址
		由于 JVM 具有指令重排的特性，执行顺序有可能变成 1>3>2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。
	不保证原子性
保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

































## DeadLock概念

死锁是指两个或多个以上的进程在执行过程中，因争夺资源而造成一种互相等待的现象，若无外力干涉那他们都将无法推进下去。如果资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。

![](/Jvav/img/deadLock.png)

### 原因

系统资源不足
进程运行推进的顺序不对
资源分配不当

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
// t1	 自己持有lockA	 尝试获取：lockB
// t2	 自己持有lockB	 尝试获取：lockA
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