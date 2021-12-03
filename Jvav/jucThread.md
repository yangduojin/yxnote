#### 获得多线程的方法

- 继承thread
- 实现runnable
- Callable接口+FutureTask 可拿到返回结果,可处理异常
- ThreadPoolExecutor

#### 线程状态

1. NEW
2. RUNNABLE
3. BLOCKED
4. WAITING
5. TIMED_WAITING
6. TERMINATED

![](/Jvav/img/threadState.jpg)

#### Wait 和 Sleep的区别

Wait 进入一个 Waiting 状态 释放重新抢夺锁

Sleep 进入 TimeWaiting 状态 不释放锁

#### Concurrent & Parallel

并发 Concurrent 同时段或时间点多个访问，都要运行这段程序

并行 Parallel 用下载来比喻，它把内容截取多份，然后分别  从n个点开始下载，最后拼在一起