## 概念

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

```
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