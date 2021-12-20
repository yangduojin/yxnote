# 多线程

- [多线程](#多线程)
  - [概念详解](#概念详解)
    - [获得多线程的方法](#获得多线程的方法)
    - [Wait 和 Sleep的区别](#wait-和-sleep的区别)
    - [Concurrent & Parallel](#concurrent--parallel)
    - [volatile和synchronized的比较](#volatile和synchronized的比较)
    - [synchronized和lock的区别](#synchronized和lock的区别)
  - [线程](#线程)
    - [线程状态](#线程状态)
      - [看线程的状态](#看线程的状态)
    - [线程获取锁对象和wait流程图](#线程获取锁对象和wait流程图)
    - [多线程口诀](#多线程口诀)
      - [交替打印1A2B3C4D 代码](#交替打印1a2b3c4d-代码)
      - [多线程循环，轮流执行 代码](#多线程循环轮流执行-代码)
    - [锁顺序和不同的锁对象](#锁顺序和不同的锁对象)
    - [Callable接口及其线程执行控制的几种方法(辅助类)](#callable接口及其线程执行控制的几种方法辅助类)
      - [callable接口与runnable接口的区别？](#callable接口与runnable接口的区别)
      - [FutureTuak类构造方法包装Callable接口，自身实现了Runnable接口，作为一个适配器模式](#futuretuak类构造方法包装callable接口自身实现了runnable接口作为一个适配器模式)
      - [JUC辅助类](#juc辅助类)
    - [BlockingQueue 阻塞队列 是Collection下面的 Queue接口](#blockingqueue-阻塞队列-是collection下面的-queue接口)
    - [线程池 池化思想 最大化收益,最小化风险,线程池,内存池,连接池](#线程池-池化思想-最大化收益最小化风险线程池内存池连接池)
      - [ThreadPoolExecutor 重要的7个参数](#threadpoolexecutor-重要的7个参数)
      - [4种拒绝策略: 都是 ThreadPoolExecutor 的静态内部类](#4种拒绝策略-都是-threadpoolexecutor-的静态内部类)
      - [自定义线程池](#自定义线程池)

## 概念详解

### 获得多线程的方法

- 继承thread(多线程不一定按照代码的顺序执行,主要看谁能抢到时间片)
- 实现runnable
- Callable接口+FutureTask 可拿到返回结果,可处理异常
- ThreadPoolExecutor

![对象头详情](/Jvav/img/objectHeadMarkWord.png)

### Wait 和 Sleep的区别

Wait 进入一个 Waiting 状态 释放重新抢夺锁

Sleep 进入 TimeWaiting 状态 不释放锁

### Concurrent & Parallel

并发 Concurrent 同时段或时间点多个访问，都要运行这段程序

并行 Parallel 用下载来比喻，它把内容截取多份，然后分别  从n个点开始下载，最后拼在一起

### volatile和synchronized的比较

- 当线程对 volatile变量写时，java 会把值刷新到共享内存中；而对于synchronized，指的是当线程释放锁的时候，会将共享变量的值刷新到主内存中。
- 线程读取volatile变量时，会将本地内存中的共享变量置为无效；对于synchronized来说，当线程获取锁时，会将当前线程本地内存中的共享变量置为无效。
- synchronized 扩大了可见影响的范围，扩大到了synchronized作用的代码块。
- 关键字volatile 与内存模型紧密相关，是线程同步的轻量级实现，其性能要比 synchronized关键字好。在作用对象和作用范围上，volatile 用于修饰变量，而 synchronized关键字 用于修饰方法和代码块，而且 synchronized 语义范围不但包括 volatile拥有的可见性，还包括volatile 所不具有的原子性，但不包括 volatile 拥有的有序性，即允许指令重排序(因为只有一个线程能够执行,所以不需要指令重排)。因此，在多线程环境下，volatile关键字 主要用于及时感知共享变量的修改，并保证其他线程可以及时得到变量的最新值。

### synchronized和lock的区别

- lock锁更加的面向对象
- sync进入blocked状态是被动地 此时还没有进入到代码块之内
- lock进入Waiting状态是主动地 此时已经进入到代码块之内，恢复之后会直接从锁住的地方开始执行

- Synchronized(旧): wait / notify / notifyAll

- Lock(新):jdk8  await / signal / signalAll ReentrantLock
- Lock 底层是通过 AQS + CAS 机制来实现的

- synchronized 缺点 就算是去读也会进行加锁
- lock 可以主动释放锁 能做synchronized做的所有事情，更加面向对象
- jdk1.6以后对synchronized进行了优化 所以性能相差不大

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();多路通知
Condition condition2 = lock.newCondition();
public void increment() {
    lock.lock();
    try {
        while (num != 0) {
            try {
                condition.await();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();}}
        num++;
        condition.signalAll();可以多路通知不同的await
        System.out.println(Thread.currentThread().getName() + " ：\t" + num);
    } catch (Exception e) {
    } finally {
        lock.unlock();}}
```

## 线程

### 线程状态

1. NEW
2. RUNNABLE
3. BLOCKED
4. WAITING
5. TIMED_WAITING
6. TERMINATED

![线程状态](/Jvav/img/threadState.jpg)

#### 看线程的状态

- jps 看进程的id号
- jstack jps得到的线程ip号，看线程当前的一个状态(快照)
  - 线程调用sleep 进入一个TimeWaiting状态 不释放锁
  - 线程调用wait,进入一个Waiting状态 释放锁，是阻塞状态
  - 当一个线程进入另一个线程已经获得synchronized锁的时候,此时这个线程会进入Blocked状态
  - 当一个线程进入另一个线程已经获得lock锁的时候,此时这个线程会进入Waiting状态

### 线程获取锁对象和wait流程图

![线程获取锁对象或者wait流程图](/Jvav/img/threadGetMonitorOrWait.png)

notifyAll会把等待队列waitQueue中的任务，迁移至SynchronizedQueue同步队列

### 多线程口诀

1. 在高内聚低耦合的前提下，线程  操作(对外暴露的调用方法)  资源类
2. 判断 / 干活 / 通知
3. 多线程交互中，必须防止多线程的虚假唤醒，也即(判断要用while，不能用if )
4. 注意标志位的修改和定位(一把锁三个钥匙，修改标志位，依顺序唤醒下一位)
5. 多线程循环执行，要么一个条件1010正负执行，要么标志位++，轮流执行

```java
public synchronized void decrement() {
// 如果这里while用if，wait()被唤醒后，不会再次判断条件，直接跳出判断，往下走。如果多个该方法被notifyAll唤醒，可能导致num超出条件
    while (num == 0) {
        try {
            this.wait();
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block　
            e.printStackTrace();}}
    num--;
    notifyAll();
    System.out.println(Thread.currentThread().getName() + " ：\t" + num);}
```

#### 交替打印1A2B3C4D 代码

```java
public class A1HomeWork2 {
    public static void main(String[] args) {
        Print print = new Print();
        new Thread(() -> {
            for (int j = 1; j <= 26; j++) {
                print.printChar(j);}
        }).start();
        new Thread(() -> {
            for (int i = 1; i <= 26; i++) {
                print.printNum(i);}
        }).start();}}
class Print {
    int flag = 0;
    Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    void printChar(int num) {
        lock.lock();
        try {
            while (flag != 0) {
                condition1.await();
            }
            flag = 1;
            condition2.signal();
            System.out.println(num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();}}
    void printNum(int num) {
        lock.lock();
        try {
            while (flag != 1) {
                try {
                    condition2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();}}
            flag = 0;
            condition1.signal();
            System.out.println((char) (64 + num));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();}}}
```

#### 多线程循环，轮流执行 代码

```java
public class N3ThreadOrderAccess {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    phone.print5();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();}}}, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    phone.print10();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();}}}, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    phone.print15();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }}}, "C").start();}}
class Phone {
    int num = 1;
    Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();
    public void print5() throws InterruptedException {
        lock.lock();
        try {
            while (num != 1) {
                condition1.await();}
            for (int i = 1; i < 6; i++) {
                System.out.println(Thread.currentThread().getName() + "   " + i);}
            num = 2;
            condition2.signal();
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            lock.unlock();}}
    public void print10() throws InterruptedException {
        lock.lock();
        try {
            while (num != 2) {
                condition2.await();}
            for (int i = 1; i < 11; i++) {
                System.out.println(Thread.currentThread().getName() + "   " + i);}
            num = 3;
            condition3.signal();
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            lock.unlock();}}
    public void print15() throws InterruptedException {
        lock.lock();
        try {
            while (num != 3) {
                condition3.await();}
            for (int i = 1; i < 16; i++) {
                System.out.println(Thread.currentThread().getName() + "   " + i);}
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            // TODO: handle exception
        } finally {
            lock.unlock();}}}
```

### 锁顺序和不同的锁对象

一个对象如果有多个synchronized方法，一个线程去调用其中的一个synchronized方法，该对象的其他线程访问其他synchronized方法只能等待。某一时刻内，只能有一个唯一的线程去访问对象的synchronized方法，锁的是当前对象this，被锁定后，其他线程都不能进入到当前对象的其他synchronized方法

- 对于普通同步方法，锁是当前实例对象
- 对于静态同步方法，锁是当前类的Class对象
- 对于同步方法块，锁是synchronized括号里配置的对象

### Callable接口及其线程执行控制的几种方法(辅助类)

#### callable接口与runnable接口的区别？

- 是否有返回值
- 是否抛异常
- 落地方法不一样，一个是run，一个是call
- Callable结果可以复用，调用过程中如果结果没有出来会阻塞

#### FutureTuak类构造方法包装Callable接口，自身实现了Runnable接口，作为一个适配器模式

- FutureTask futureTask2 = new FutureTask(new myThread());参数实现了Callable接口
- new Thread(futureTask2, "B").start();新起线程执行myThread的内容
- new Thread(futureTask2, "C").start();同一个FutureTask执行同样的任务会复用结果,而不会执行两次,这里是跟上一句作比较,但是如果是new Thread(futureTask3, "C").start(); 新的FutureTask就会各自执行一次
- futureTask.get();main里面阻塞等待所有的回调线程完成

#### JUC辅助类

- CountDownLatch类有个初始值，用数量来控制线程，执行一次就--；一个线程等待其他线程
  - CountDownLatch countDownLatch = new CountDownLatch(7);分配初次
  - countDownLatch.countDown();执行一次就--
  - countDownLatch.await();main里面阻塞等待所有的线程完成
- CyclicBarrier 初始化实例就指定次数和所有线程完成之后的结果，所有线程一起等待就绪
  - CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {System.out.println("集齐龙珠,召唤神龙");});
  - cyclicBarrier.await(); 在线程内部暂停，不是main，所有的完成后直接显示结果
- Semaphore 信号量控制，每个线程都可以执行一次，但是需要等待上一个线程释放信号量
  - Semaphore semaphore = new Semaphore(3); 初始化信号量个数
  - semaphore.acquire(); 获取信号量，执行后续操作 也有tryacquire 尝试获取,可以获取就成功,不成功直接返回,不阻塞
  - semaphore.release(); 释放信号量，一般放在finally里
- ReadWriteLock 读写锁，读是共享锁，写是互斥，排他锁
  - 读读可以共存
  - 读写不可共存
  - 写写不可共存
    - ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    - readWriteLock.writeLock().lock(); 开始上锁
    - readWriteLock.writeLock().unlock();放在finally
    - readWriteLock.readLock().lock(); 开始上锁
    - readWriteLock.readLock().unlock();放在finally

### BlockingQueue 阻塞队列 是Collection下面的 Queue接口

阻塞：某些情况下会挂起线程，一旦条件满足又会唤醒被挂起的线程。有了阻塞队列，不用自己处理阻塞，唤醒，队列会全部解决

- **ArrayBlockingQueue**<>(capacity)  数组有界阻塞列队,指定队列大小,put,take用的是同一把锁，不能同时进行
- **LinkedBlockingDeque**<>()         链表有界阻塞队列，put，take不是同一把锁，可以同时进行,很大，相当于无界
- PriorityBlockingQueue<>()       支持优先级排序的无界阻塞列队
- DelayQueue<>()                  使用优先级队列实现的延迟无界阻塞列队
- **SynchronousQueue**<>()            不存储元素的阻塞队列,单个元素的队列,put需要等待take操作,不然阻塞,适合传递数据场景
- LinkedTransferQueue<>()         链表无界阻塞队列
- LinkedBlockingDeque<>()         链表双向阻塞队列

核心方法：

- 阻塞队列满或者空，再执行操作会抛出异常 add( e ) / remove() / element( ) 检查队首元素并返回
- 特殊值 插入true fasle ,移除 返回元素或者null： offer ( e ) / poll() / peek()
- 一直阻塞 队列满或空，会一直阻塞：put( e ) / take()
- 超时退出 **：offer**( e, time, unit ) / **poll**( time , unit ) TimeUnit

### 线程池 池化思想 最大化收益,最小化风险,线程池,内存池,连接池

![线程池状态](/Jvav/img/threadPoolState.png)

- RUNNING:    能接受新提交的任务,并且也能够处理阻塞队列中的任务;
- SHUTDOWN:   不再接受新提交的任务,但是可以处理存量任务(即阻塞队列中的任务)
- STOP:       不再接受新提交的任务,也不处理存量任务;
- TIDYING:    所有任务都已终止;
- TERMINATED: 默认是什么也不做的,只是作为一个标识

三个自带的线程池，会OOM，这三个底层都是ThreadPoolExecutor，线程池就是在玩这个类，工作中一般用自定义的ThreadPoolExecutor

- ExecutorService executorService = Executors.newCachedThreadPool(); 堆积大量请求，可能导致OOM
- Executors.newFixedThreadPool(); 堆积大量请求，可能导致OOM
- Executors.newSingleThreadExecutor(); 和ScheduledThreadPool都可能创造大量线程，导致OOM

![线程池流程](/Jvav/img/threadPoolProcess.png)

![线程池流程2](/Jvav/img/threadPoolProcess2.png)

创建线程池成功之后，线程池中的线程数为零,当有新的任务来临的时候

1. 当提交的任务<核心线程数 马上去创建核心个线程数
1. 当提交的任务>核心线程数 没有处理的任务会在队列中等待
1. 如果等待队列满了 当提交的任务>(核心线程数+队列大小) 创建非核心线程去处理任务
1. 如果等待队列满了 当提交的任务>(max线程数+队列大小)  采用拒绝策略

#### ThreadPoolExecutor 重要的7个参数

ThreadPoolExecutor(

1. int corePoolSize,                    常驻核心线程数
1. int maximumPoolSize,                最大同时执行线程数,任务多就会增加到最大，数目 = 逻辑处理器+1
1. long keepAliveTime,                 多余的空闲线程存活时间,数目恢复到常驻数
1. TimeUnit unit,                      KeepAliveTime的单位
1. BlockingQueue \< Runnable \> workQueue,  任务队列,提交还未被执行的任务
1. ThreadFactory threadFactory,        线程工厂, 默认 Executors.defaultThreadFactory()
1. RejectedExecutionHandler handler)   拒绝策略，默认 new ThreadPoolExecutor.AbortPolicy()

上面的阻塞队列,如果使用别的如linked,默认是最大值,可以自己传入值,如压力测试系统的峰值

最大线程数设置：

- cpu密集型：Runtime.getRuntime().availableProcessors()逻辑处理器+1
- io密集型：1. 线程并不是一直在执行任务,配置尽可能多的线程,如cpu数*2;2.cpu核数/(1-阻塞系数) 阻塞系数在0.8~0.9之间
ex: 8核cpu: 8/(1-0.9) = 80.0 个线程数

#### 4种拒绝策略: 都是 ThreadPoolExecutor 的静态内部类

1. AbortPolicy(): 抛异常，终止运行，可以catch
1. CallerRunsPolicy()：返回给调用者执行，一般是main
1. DiscardPolicy()： 直接悄悄遗弃，允许任务丢失就选这个
1. DiscardOldestPolicy()： 遗弃等待最久的，将当前任务加入队列，尝试提交当前任务

#### 自定义线程池
ExecutorService executorService = new ThreadPoolExecutor(2, 5, 2L, TimeUnit.SECONDS,new LinkedBlockingQueue<>(3), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "处理任务");});
executorService.shutdown();
