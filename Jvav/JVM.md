- [jvm架构模型](#jvm架构模型)
  - [基于栈式](#基于栈式)
  - [指令](#指令)
    - [地址、操作数](#地址操作数)
  - [基于寄存器式](#基于寄存器式)
- [引用分类](#引用分类)
  - [强引用（Strong Reference）](#强引用strong-reference)
  - [软引用（Soft Reference）](#软引用soft-reference)
  - [弱引用  (Weak Reference)](#弱引用--weak-reference)
  - [虚引用（Phantom Reference）](#虚引用phantom-reference)
- [请谈谈你对 OOM 的认识？](#请谈谈你对-oom-的认识)
- [class文件反编译 & jvm信息](#class文件反编译--jvm信息)
  - [查看线程状态](#查看线程状态)
  - [jconsole & jvisualvm](#jconsole--jvisualvm)
- [JVM的生命周期](#jvm的生命周期)
  - [虚拟机的启动](#虚拟机的启动)
  - [虚拟机的执行](#虚拟机的执行)
  - [虚拟机的退出](#虚拟机的退出)
- [jvm结构](#jvm结构)
  - [类装载器子系统](#类装载器子系统)
    - [双亲委派](#双亲委派)
  - [运行时数据区](#运行时数据区)
    - [方法区 (永久区)](#方法区-永久区)
    - [堆](#堆)
    - [PC寄存器(程序计数器)](#pc寄存器程序计数器)
    - [java栈(虚拟机栈)](#java栈虚拟机栈)
    - [本地方法栈](#本地方法栈)
  - [执行引擎](#执行引擎)
  - [本地方法接口](#本地方法接口)
  - [本地方法库](#本地方法库)
- [你说你做过 JVM 调优和参数配置，请问如果盘点查看 JVM 系统默认值？](#你说你做过-jvm-调优和参数配置请问如果盘点查看-jvm-系统默认值)
  - [JVM 的参数类型:](#jvm-的参数类型)
    - [标配参数](#标配参数)
    - [X 参数（了解）](#x-参数了解)
    - [XX 参数](#xx-参数)
    - [查看 JVM 默认值](#查看-jvm-默认值)
- [JVM参数(在VM options里面添加)](#jvm参数在vm-options里面添加)
- [垃圾回收算法](#垃圾回收算法)
  - [可达性分析法](#可达性分析法)
  - [引用计数算法  (无法解决循环引用的情况)](#引用计数算法--无法解决循环引用的情况)
  - [复制算法(s1,s2 复制 清空 互换)](#复制算法s1s2-复制-清空-互换)
  - [标记清除算法  (回收的空间存在碎片)](#标记清除算法--回收的空间存在碎片)
  - [标记整理压缩算法](#标记整理压缩算法)
- [垃圾回收器](#垃圾回收器)
- [怎么查看服务器默认垃圾收集器是哪个？生产是如何配置垃圾收集器？谈谈你对垃圾收集器的理解？CMS你知道吗？](#怎么查看服务器默认垃圾收集器是哪个生产是如何配置垃圾收集器谈谈你对垃圾收集器的理解cms你知道吗)
- [G1 垃圾收集器你了解吗？](#g1-垃圾收集器你了解吗)
  - [以前收集器的特点](#以前收集器的特点)
  - [G1 是什么](#g1-是什么)
  - [G1的特点](#g1的特点)
  - [底层原理](#底层原理)
  - [生产环境服务器变慢，诊断思路和性能评估谈谈？](#生产环境服务器变慢诊断思路和性能评估谈谈)
  - [假如生产环境出现 CPU 过高，请谈谈你的分析思路和定位？](#假如生产环境出现-cpu-过高请谈谈你的分析思路和定位)
- [JMM](#jmm)

## jvm架构模型
[本文参考](http://blog.cuzz.site/2019/05/10/JVM%E9%9D%A2%E8%AF%95/)

### 基于栈式

优点
- 设计和实现简单，适用于资源受限的系统
- 避开了寄存器的分配难题：使用零地址指令方式分配
- 指令流中大部分都是零地址指令，执行过程依赖操作栈，指令集更小，编译器容易实现
- 8位字节码，所以说指令集更小，但是完成一项操作花费的指令相对多。
- 不需要硬件支持，可移植性更好，更好实现跨平台

缺点
- 性能下降，实现同样的功能需要更多的指令，毕竟还要入栈出栈等操作

### 指令

#### 地址、操作数

  - 零地址只有操作数:基于栈式的，因为是操作栈顶的元素，所以不需要地址
  - 一地址有一个地址，一个操作数
  - 二地址有两个地址，一个操作数

### 基于寄存器式

优点
- 性能优秀，执行更高效
- 花费更少的指令去完成一项操作

缺点
- 指令集架构完全依赖硬件，可移植性差
- 典型应用是X86的二进制指令集，比如传统的PC以及安卓的Davlik虚拟机
- 16位字节码
- 大部分情况下，指令集往往以一地址指令，二地址指令和三地址指令为主。

栈：简单，跨平台性，指令集小，指令多，执行性能比寄存器差

## 引用分类

### 强引用（Strong Reference）
- 通常我们通过new来创建一个新对象时返回的引用就是一个强引用，若一个对象通过一系列强引用可到达，它就是强可达的(strongly reachable)，那么它就不被回收,在系统内存很富裕的情况下，因为强引用内存不能被释放，所以会多申请了很多内存。即使oom也不会被回收;
```java
Object object = new Object();
Object object2 = object;
object = null;
system.gc();
sout(object2) object2依然是对象,object为null被回收了;
```

### 软引用（Soft Reference）
- 软引用和弱引用的区别在于，若一个对象是弱引用可达，无论当前内存是否充足它都会被回收，而软引用可达的对象在内存不充足时才会被回收，因此软引用要比弱引用“强”一些,只有内存马上要溢出了才触发它的GC;``Map<String, SoftReference<BitMap>> map = new HashMap<>();`` 用来加载图片地址,内存不足就直接回收

### 弱引用  (Weak Reference)
- 弱引用的引用强度比软引用要更弱一些，无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收。在 JDK1.2 之后，用 java.lang.ref.WeakReference 来表示弱引用。弱引用是发生了一次垃圾回收后，既存的弱引用对象就开始回收。通常，一个弱引用对象仅能生存到下一次垃圾回收前。

```java
public class WeakReferenceDemo {
    public static void main(String[] args) {
        Object obj = new Object();
        WeakReference<Object> weakReference = new WeakReference<>(obj);
        System.out.println(obj);
        System.out.println(weakReference.get());

        obj = null;
        System.gc();
        System.out.println("GC之后....");
        
        System.out.println(obj);
        System.out.println(weakReference.get());}}

//输出
// java.lang.Object@1540e19d
// java.lang.Object@1540e19d
// GC之后....
// null
// null
```

引用队列
```java
public class ReferenceQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        Object obj = new Object();
        ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
        WeakReference<Object> weakReference = new WeakReference<>(obj, referenceQueue);
        System.out.println(obj);
        System.out.println(weakReference.get());
        System.out.println(weakReference);

        obj = null;
        System.gc();
        Thread.sleep(500);

        System.out.println("GC之后....");
        System.out.println(obj);
        System.out.println(weakReference.get());
        System.out.println(weakReference);}}

// java.lang.Object@1540e19d
// java.lang.Object@1540e19d
// java.lang.ref.WeakReference@677327b6
// GC之后....
// null
// null
// java.lang.ref.WeakReference@677327b6

如果 WeakReference 换成 PhantomReference 打印结果如下

// java.lang.Object@1540e19d
// null
// null
// GC之后....
// null
// null
// java.lang.ref.WeakReference@677327b6
```
会把该对象的包装类即weakReference放入到ReferenceQueue里面，我们可以从queue中获取到相应的对象信息，同时进行额外的处理。比如反向操作，数据清理等。

### 虚引用（Phantom Reference）
- 虚引用是Java中最弱的引用，那么它弱到什么程度呢？它是如此脆弱以至于我们通过虚引用甚至无法获取到被引用的对象，虚引用存在的唯一作用就是当它指向的对象被回收后，虚引用本身会被加入到引用队列中，用作记录它指向的对象已被销毁。
- 任何时候可能被GC回收，就像没有引用一样。
- 并且他必须和引用队列一起使用，用于跟踪垃圾回收过程，当垃圾回收器回收一个持有虚引用的对象时，在回收对象后，将这个虚引用对象加入到引用队列中，用来通知应用程序垃圾的回收情况。

Java的强软弱虚引用被回收的时机不同：强引用是引用被释放才会回收；软引用是没释放，但是快OOM了就会被回收；弱引用是引用没释放，但是发生了GC后就会被回收；虚引用随时会回收，好像没有存在过，但是会有一个队列来跟踪它的垃圾回收情况。

Guava在没有显示设置强、软、弱引用的情况下默认是强引用。

构造一个弱引用
```java
  productA = new Product(...);
  WeakReference<Product> weakProductA = new WeakReference<>(productA);
  Product product = weakProductA.get();
```

WeakHashMap
- jdk提供WeakHashMap，当productA变为null时（表明它所引用的Product已经无需存在于内存中），这时指向这个Product对象的就是由弱引用对象weakProductA了，那么显然这时候相应的Product对象时弱可达的，所以指向它的弱引用会被清除，这个Product对象随即会被回收，指向它的弱引用对象会进入引用队列中。 
- 
WeakReference类有两个构造函数：
- WeakReference(T referent) //创建一个指向给定对象的弱引用
- WeakReference(T referent, ReferenceQueue<? super T> q) //创建一个指向给定对象并且登记到给定引用队列的弱引用
- 创建的弱引用对象注册到了一个引用队列上，这样当它被垃圾回收器清除时，就会把它送入这个引用队列中，我们便可以对这些被清除的弱引用对象进行统一管理。

## 请谈谈你对 OOM 的认识？

1. java.lang.StackOverflowError
     - 在一个函数中调用自己就会产生这个错误(没有判断条件的分支)

2. java.lang.OutOfMemoryError : Java heap space
     - new 一个很大对象

3. java.lang.OutOfMemoryError : GC overhead limit exceeded
     - 执行垃圾收集的时间比例太大， 有效的运算量太小，默认情况下,，如果GC花费的时间超过 **98%**， 并且GC回收的内存少于 **2%**， JVM就会抛出这个错误。

4. java.lang.OutOfMemoryError : Direct buffer memory(redis循环操作里面遇到过一次,跟netty和letter有关系)
     - 配置参数：-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m

```java 
//sun.misc.VM.maxDirectMemory() jdk文件夹里面的rt.jar 里面的vm.class
public class DirectBufferDemo {
    public static void main(String[] args) {
        System.out.println("maxDirectMemory : " + sun.misc.VM.maxDirectMemory() / (1024 * 1024) + "MB");
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(6 * 1024 * 1024);
    }
}
```
输出
```
maxDirectMemory : 5MB
[GC (System.gc()) [PSYoungGen: 1315K->464K(2560K)] 1315K->472K(9728K), 0.0008907 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (System.gc()) [PSYoungGen: 464K->0K(2560K)] [ParOldGen: 8K->359K(7168K)] 472K->359K(9728K), [Metaspace: 3037K->3037K(1056768K)], 0.0060466 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
    at java.nio.Bits.reserveMemory(Bits.java:694)
    at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
    at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:311)
    at com.cuzz.jvm.DirectBufferDemo.main(DirectBufferDemo.java:17)
Heap
 PSYoungGen      total 2560K, used 56K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 2% used [0x00000000ffd00000,0x00000000ffd0e170,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 7168K, used 359K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 5% used [0x00000000ff600000,0x00000000ff659e28,0x00000000ffd00000)
 Metaspace       used 3068K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 336K, capacity 388K, committed 512K, reserved 1048576K
```

5. java.lang.OutOfMemoryError : unable to create new native thread
   - 与平台有关,创建线程数太多了;linux系统中是unable to create new native thread,linux默认1024线程,可以修改最大线程数;ulimit -u 查看当前用户的可用线程数,普通是1024

6. java.lang.OutOfMemoryError : Metaspace
   - Java 8 之后的版本使用元空间（Metaspace）代替了永久代，元空间是方法区在 HotSpot 中的实现，它与持久代最大的区别是：元空间并不在虚拟机中的内存中而是使用本地内存。
      - 元空间存放的信息：
      - 虚拟机加载的类信息
      - 常量池
      - 静态变量
      - 即时编译后的代码

## class文件反编译 & jvm信息

javap 选项 xxx.class,能看 `synchornized` 执行时 `class` 文件的执行流程和锁的获取 `monitorenter` 和 `monitorexit`
选项
- -v 输出附加信息
- -l 输出行号和本地变量表
- -p 显示所有类和成员
- -c 对代码进行反汇编

### 查看线程状态

jps 查看进程的id号 , jstack +看jps得到的pid号 可以查看线程当前的一个状态(快照);jps -l  类比linux  ps |grep xxx ls -l

### jconsole & jvisualvm  

要使用jvm自带的性能监控工具要在idea里面的service,每个微服务的配置里面jre选择跟工具在同一个路径的jre版本,idea使用的的jdk可能跟自己安装的不一样 

安装插件网络有问题,自己看java版本的后缀,去https://visualvm.github.io/pluginscenters.html 这个里面对应的地方下载插件到本地,安装工具里面有添加本地插件,选择文件安装即可

## JVM的生命周期

### 虚拟机的启动

通过引导类加载器bootstrap class loader创建一个初始类来完成的，这个类是由虚拟机的具体实现指定的。

### 虚拟机的执行

执行一个所谓的Java程序的时候，真正执行的是一个叫Java虚拟机的进程

### 虚拟机的退出

- 程序正常执行结束
- 执行过程遇到异常或错误而异常终止
- 操作系统错误导致Java虚拟机进程终止
- Runtime类或System类的exit方法、runtime类的halt方法，并且Java安全管理器允许这次exit或halt操作
  - halt停止、停下、阻止
  - exit方法源码：static native void halt0（int status）

JNI(Java Native Interface)规范描述了用JNI Invocation API来加载或卸载Java虚拟机时，Java虚拟机退出的情况

## jvm结构

![](/Jvav/img/jvmBase.png)
![](/Jvav/img/jvmBase2.png)
![](/Jvav/img/jvmBase3.png)

### 类装载器子系统

这四者不是继承或父子关系，应该是包容的关系，只负责加载，能不能运行由执行引擎决定
- 启动类加载器BootstrapClassLoader，虚拟机的启动就是通过它创建一个初始类来完成的，这个类是由虚拟机的具体实现指定的；
- 扩展类加载器ExtClassLoader
- 应用程序类加载器AppClassLoader  也叫系统类加载器，加载当前应用的classpath的所有类.我们写的类用这个加载器
- 用户自定义类加载器 是lang.ClassLoader的子类，用户可以自定义类的加载器

#### 双亲委派

![](/Jvav/img/parentQuery.png)
类加载向上查询是否加载过,再从顶到尾尝试加载,都不能加载就classNotFound

沙箱安全机制: 避免类的重复加载，保护程序安全 保护JAVA原生的JDK代码 

双亲委派优点？
- 安全，可避免用户自己编写的类动态替换Java的核心类，如java.lang.String
- 避免全限定命名的类重复加载(使用了findLoadClass()判断当前类是否已加载)

在JVM中表示两个class对象，是否为同一个类存在两个必要条件？
- 类的完整类名必须一致，包括包名
- 加载这个类的ClassLoader必须相同

有一个自定义类加载器, 加载的是~/com/lxl/jvm/User1.class类, 而在应用程序的target目录下也有一个com/lxl/jvm/User1.class,  最终User1.class一定是被应用程序类加载器AppClassLoader加载, 而不是我们自定义的类加载器, 为什么呢? 因为他要向上寻找, 向下委托. 当找到了以后, 便不再向后执行了. 

如何打破双亲委派机制呢? 双亲委派机制是在哪里实现的? 是在ClassLoader类的loadClass(...)方法实现的. 如果我们不想使用系统自带的双亲委派模式, 只需要重新实现ClassLoader的loadClass(...)方法即可. 下面是ClassLoader中定义的loadClass()方法. 里面实现了双亲委派机制

![](/Jvav/img/parentClassLoad.png)

下面给DefinedClassLoaderTest.java增加一个loadClass方法, 拷贝上面的代码即可. 删除掉中间实现双亲委派机制的部分

![](/Jvav/img/parentClassLoad2.png)

这里需要注意的是, com.lxl.jvm是自定义的类包, 只有我们自己定义的类才从这里加载. 如果是系统类, 依然使用双亲委派机制来加载.

JVM必须知道一个类型是由启动类加载器加载的，还是由用户类加载器加载的。如果是用户类加载器加载的，JVM会将这个类加载器的一个引用作为类型信息的一部分，保存到方法区中。



### 运行时数据区

#### 方法区 (永久区)

- 一个常驻内存的区域，包含 静态变量(类变量) + 常量 + 类信息(.class 构造方法/接口定义) + 运行时常量池
- 类加载器加载的类信息就放到方法区，该区归所有线程共享，后面改名元空间Matespace
- 实例变量存在堆内存中,和方法区无关
- ThreadLocal 将线程共享的资源本地化到线程私有
![](/Jvav/img/heapAndMetaspace.png)

#### 堆

- 新生代 老年代 永久区(Java7之前)
- 新生代(33%:1)，老年代(66%:2)，元空间(java8)
- 新生代又分为eden(80%:8),S0(10%:1),S1(10%:1)
- eden进入下一代都会gc，old fullgc之后依然无法进行对象的保存就会OOM，javaHeapSpace  
![](/Jvav/img/heap.png)

- 一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域
- Java堆区在JVM启动的时候即被创建，其空间大小也就确认了。堆内存的大小是可调节的
- Java虚拟机规范规定，堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的。
- 所有的线程共享Java堆，在这里还可以划分线程私有的缓冲区（TLAB）
- “几乎”所有的对象实例都在这里分配内存
- 数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，引用指向对象或者数组在堆中的位置
- 方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。
- 堆是GC执行垃圾回收的重点区域
- 类一般都是在eden被创建的，通过垃圾算法慢慢交换到old，但是也可以通过设置vm参数，设定大于多大的文件直接在old创建；
- MinorGC，MajorGC，FullGC
    - 出现了MajorGC，经常会伴随至少一次MinorGC
    - 如果MajorGC后，内存还不足，就报OOM了
- FullGC的触发机制
  - 调用System.gc()时，系统建议执行FullGC，但是不必然执行
  - 老年代空间不足
  - 方法区空间不足
  - 通过MinorGC后进入老年代的平均大小，大于老年代的可用内存
  - 由Eden区，Survivor 0区向Survivor 1区复制时，对象的大小大于ToSpace可用内存，则把改对象转存到老年代，且老年代的可用内存小于该对象的大小

FullGC是开发或调优中尽量要避免的，这样暂停时间会短一些。

#### PC寄存器(程序计数器)

- 线程私有,就是一个指针，指向方法区中的方法字节码，指向下一个方法所在的地址，灰色的这几个都不会被垃圾回收器回收，因为生命周期短
- 运行时数据区中唯一不会出现OOM的区域，没有垃圾回收
- 当前线程所执行的字节码的行号指示器: 为了线程切换后能恢复到正确的位置
- 每个线程有一个独立的程序计数器，线程之间互不影响。
- 如果线程执行的Java方法，则计数器记录正在执行的虚拟机字节码的指令的地址
- 如果正在执行的本地方法，这个计数器值则应为空。（undefined）

#### java栈(虚拟机栈)

栈和堆的区别，各自解决什么问题
- 栈是运行时的单位而堆是存储的单位，栈解决程序如何执行，如何处理数据。
- 堆解决的是数据存储问题，即数据怎么放，放在哪里。
  
栈的生命周期
- 栈也叫栈内存，每个线程创建时都会创建一个虚拟机栈，内部保存一个个栈帧，对应着一次次的Java方法调用
，生命周期和线程的一致，对于栈来说不存在垃圾回收问题，只要线程一结束该栈就Over，是线程私有的。

保存的数据类型：
- 8种基本类型的变量 
- 对象的引用变量
- 实例方法都是在函数的栈内存中分配。

栈帧中主要保存3 类数据： 
- 本地变量(Local Variables):输入参数和输出参数以及方法内的变量； 局部变量，对象的引用变量
- 栈操作(Operand Stack):记录出栈、入栈的操作； 
- 栈帧数据(Frame Data):包括类文件、方法等等
  
优点
- 快速有效的存储方式，访问速度仅次于程序计数器
- JVM直接对JAVA栈的操作只有两个
    - 每个方法执行，伴随着进栈（入栈，压栈）
    - 执行结束的出栈
- 栈不存在垃圾回收，但是存在OOM: Java栈大小是动态或者固定不变的。如果是动态扩展，无法申请到足够内存会OOM，如果是固定，线程请求的栈容量超过固定值，则StackOverflowError
- 使用-Xss，设置线程的最大栈空间

栈的存储单位
- 每个线程都有自己的栈，栈中的数据以栈帧格式存储
- 线程上正在执行的每个方法都各自对应一个栈帧
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各个数据信息
- 先进后出，后进先出
- 一条活动的线程中，一个时间点上，只会有一个活动的栈帧。只有当前正在执行的方法的栈顶栈帧是有效的，这个称为当前栈帧，对应方法是当前方法，对应类是当前类
- 执行引擎运行的所有字节码指令只针对当前栈帧进行操作
- 如果方法中调用了其他方法，对应的新的栈帧会被创建出来，放在顶端，成为新的当前帧

栈运行原理
- 不同线程中包含的栈帧不允许存在相互引用。
- 当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为新的栈帧。
- Java方法有两种返回方式
    - 一种是正常的函数返回，使用return指令
    - 另外一种是抛出异常，不管哪种方式，都会导致栈帧被弹出

栈的内部结构

局部变量表
- 定义为一个数字数组，主要用于存储方法参数，定义在方法体内部的局部变量，数据类型包括各类基本数据类型，对象引用，以及return address类型
- 局部变量表建立在线程的栈上，是线程私有的，因此不存在数据安全问题
- 局部变量表容量大小是在编译期确定下来的
- 局部变量表存放编译期可知的各种基本数据类型（8种），引用类型（reference）,return address 类型
- 最基本的存储单元是`slot`: 32位占用一个slot，64位类型（long和double）占用两个slot
- 局部变量表中的变量只有在当前方法调用中有效，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。
- 方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁

关于Slot的理解
- JVM虚拟机会为局部变量表中的每个Slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值
- 如果当前帧是由构造方法或者实例方法创建的，那么该对象引用this，会存放在index为0的slot处，其余的参数表顺序继续排列

#### 本地方法栈

- Native 方法执行的的栈
- Java虚拟机栈管理Java方法的调用，而本地方法栈用于管理本地方法的调用
- 本地方法栈，也是线程私有的。
- 允许被实现成固定或者是可动态扩展的内存大小。
   - 内存溢出情况和Java虚拟机栈相同
- 使用C语言实现
- 具体做法是Native Method Stack 中登记native方法，在Execution Engine执行时加载到本地方法库
- 当某个线程调用一个本地方法时，就会进入一个全新，不受虚拟机限制的世界，它和虚拟机拥有同样的权限。
- 并不是所有的JVM都支持本地方法，因为Java虚拟机规范并没有明确要求本地方法栈的使用语言，具体实现方式，数据结构等
- Hotspot JVM中，直接将本地方法栈和虚拟机栈合二为一

### 执行引擎

Hotspot，Jkit

### 本地方法接口
- jvm的方法，C++程序写的，如Thread的native start0()
- Java调用非Java代码的接口,与Java环境外交互,与操作系统底层或硬件交换信息时的情况,启动一个线程

### 本地方法库
- 内存中的一块区域负责登记 native方法  自定义String。

## 你说你做过 JVM 调优和参数配置，请问如果盘点查看 JVM 系统默认值？

### JVM 的参数类型:

#### 标配参数
- -version
- -help

#### X 参数（了解）

- -Xint：解释执行
- -Xcomp：第一次使用就编译成本地代码
- -Xmixed：混合模式

#### XX 参数

- Boolean 类型：-XX：+ 或者 - 某个属性值（+ 表示开启，- 表示关闭）
  - -XX:+PrintGCDetails：打印 GC 收集细节
  - -XX:-PrintGCDetails：不打印 GC 收集细节
  - -XX:+UseSerialGC：使用了串行收集器
  - -XX:-UseSerialGC：不使用了串行收集器
- KV 设置类型：-XX:key=value
  - -XX:MetaspaceSize=128m
  - -XX:MaxTenuringThreshold=15

```java
public class HelloGC {
    public static void main(String[] args) {
        System.out.println("hello GC...");
        try {
            Thread.sleep(Integer.MAX_VALUE);
        } catch (InterruptedException e) {
            e.printStackTrace();}}}
```  
jps -l 命令，查出进程 id

再使用 jinfo -flag PrintGCDetails 1933 命令查看显示 `-XX:-PrintGCDetails` -P显示未开启

jinfo -flags 1933 查看所有参数

#### 查看 JVM 默认值

- 查看初始默认值 java -XX:+PrintFlagsInitial

- 查看修改更新 java -XX:+PrintFlagsFinal -Xss128k X(运行的类的名字)

打印结果 = 与 := 的区别是，一个是默认，一个是人物改变或者 jvm 加载时改变的参数

- 打印命令行参数(可以看默认垃圾回收器) java -XX:+PrintCommandLineFlags -version

## JVM参数(在VM options里面添加)

堆大小参数配置
- -Xmx4096m -Xms128m -Xss1m -XX:MetaspaceSize=512m -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseSerialGC 常用配置
- -Xms6m 设置java程序启动时初始堆大小6m,等价于 -XX:InitialHeapSize,默认为物理内存 1/64
- -Xmx20m 设置java程序启动最大堆大小20m(也可设置在微服务启动的vm上面,建议100m),等价于 -XX:MaxHeapSize,默认为物理内存的 1/4
- -XX:MaxTenuringThreshold=15 最多轮换15次进入old
- -XX:MetaspaceSize 元空间大小,默认比较小，我们可以调大一点防止oom
- -XX:MaxTenuringThreshold 设置垃圾最大年龄
- -XX:PretenureSizeThreshold=1024000  大于大小的文件直接在老年代创建
- -XX:+PrintGC 
- -XX:+PrintGCDetails
- -XX:+UseSerialGC 使用串行化的gc回收器，当gc时其他程序需要停止，只能gc
- -Xmx20m -Xms5m -XX:+PrintCommandLineFlags 
初始堆大小和最大堆大小可以设置为一样，可以减GC回收次数 提升性能
- Runtime.getRuntime().maxMemory()/10241024)+"MB"
  
堆分配参数
- -Xmn5m 设置新生代大小，设置太大会减少老年代大小，一般为整个堆空间的1/3 or 1/4
- -XX:NewRatio设置老年代:新生代的比例,整体要在比例+1;与Xmn同时存在时,以Xmn为准
- -XX:SurvivorRatio 设置eden : from|to的比例 8:1:1 4:1:1 设置前面那个数字
  
堆溢出设置
- -XX:+HeapDumpOnOutOfMemoryError 出现OOM打印日志
- -XX:HeapDumpPath=c:\\tmp\\test03.hprof OOM快照导出路径，需要Mat解读这个格式

栈设置
- -Xss1m 设置单个线程栈大小(查看的结果如果是0就是windows jvm默认,别的系统不一样,linux是1024)
  
方法区设置
- 默认 -XX:MaxPermSize 为64mb，难以演示，动态代理实现多个类可能导致溢出

## 垃圾回收算法

### 可达性分析法

从GC Roots的对象作为起始点，从这些节点出发所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连的时候说明对象不可用;gcroot可以为set

哪些对象可以作为 GC Roots 的对象：

- 虚拟机栈（栈帧中的局部变量区，也叫局部变量表）中引用的对象
- 方法区中的类静态属性引用的对象
- 方法区常量引用的对象
- 本地方法栈中 JNI (Native方法)引用的对象

### 引用计数算法  (无法解决循环引用的情况)

- 如果不下小心直接把 Obj1-reference 和 Obj2-reference 置 null。则在 Java 堆当中的两块内存依然保持着互相引用无法回收。无法解决循环引用的情况(没人用了)

### 复制算法(s1,s2 复制 清空 互换)

- 对象在eden产生，当eden不够用的时候，就gc一次，存活的放入from，再下一次就是eden和from一起gc 存活的放入to，然后to变成from，from清空变成to，谁空谁为to，保证名为to的survivor区域是空的
- 从from过来的次数+1，达到一定次数就放入老年代，老年代再通过标记压缩算法慢慢清理，如果fullgc还不能放下对象就OOM
- 缺点 它浪费了一半的内存，这太要命了。 
- 如果对象的存活率很高，我们可以极端一点，假设是100%存活，那么我们需要将所有对象都复制一遍，并将所有引用地址重置一遍。

### 标记清除算法  (回收的空间存在碎片)

- 原理 当堆中的有效内存空间(available memory)被耗尽的时候，就会停止整个程序(也被  称为stop the world)，然后进行两项工作，第一项则是标记，第二项则是清除。 
- 标记 从引用根节点开始标记所有被引用的对象。标记的过程其实就是遍历所有的GC Roots，  然后将所有GC Roots可达的对象标记为存活的对象。 
- 清除 遍历整个堆，把未标记的对象清除。 
- 缺点 此算法需要暂停整个应用，会产生内存碎片

### 标记整理压缩算法

- 适合老年代进行垃圾回收,先标记,再整理到一起,但是需要付出代价，因为移动对象需要成本
![](/Jvav/img/markCompact.png))

新生代和老年代调换一下是否可以
- 不可以，老年代大部分对象在垃圾回收的时候 都会存活,需要标记压缩算法来慢慢清楚。eden程序死亡的快，复制算法压力不大

## 垃圾回收器

Serial收集器(了解)
- 单线程执行垃圾回收的。当需要执行垃圾回收时，程序会暂停一切手上的工作，然后单线程执行垃圾回收。STW stop the world
- 新生代的特点是对象存活率低，所以收集算法用的是复制算法，把新生代存活对象复制到老年代，复制的内容不多，性能较好。

ParNew收集器(Parallel New Generation)
- ParNew同样用于新生代，是Serial的 多线程版本，并且在 参数 、算法(同样是复制算法)上也完全和Serial相同。
- Par是Parallel的缩写，但它的并行仅仅指的是收集多线程并行，并不是收集和原程序可以并行进行。ParNew也是需要暂停程序一切的工作，然后多线程执行垃圾回收。老年区配合CMS

Parallel Scavenge收集器(和Parallel Old配合使用)
- 多个垃圾收集线程并行工作，此时用户线程是暂停的，用于科学计算、大数据处理等弱交互场景。
- 新生代的收集器，同样用的是复制算法，也是并行多线程收集。与ParNew最大的不同，它关注的是垃圾回收的吞吐量。
- 这里的吞吐量指的是 总时间与垃圾回收时间的比例。这个比例越高，证明垃圾回收占整个程序运行的比例越小。

Parallel Old收集器
- 老年代的收集器，是Parallel Scavenge老年代的版本。其中的算法替换成Mark-Compact

Serial Old收集器(已经被抛弃了)
- 老年代的收集器，与Serial一样是单线程，不同的是算法用的是标记压缩(Mark-Compact标记压缩)
- 因为老年代里面对象的存活率高，如果依旧是用复制算法，需要复制的内容较多，性能较差。并且在极端情况下，当存活为100%时，没有办法用复制算法。所以需要用标记压缩Mark-Compact，以有效地避免这些问题

CMS(ConcMarkSweep并发标记清除)收集器
- 用户线程和垃圾收集线程同时执行（不一定是并行，可能是交替执行），不需要停顿用户线程，互联网公司多用它，适用对相应时间有要求的场景。
- CMS, Concurrent Mark Sweep同样是老年代的收集器。它关注的是垃圾回收最短的停顿时间(低停顿)，在老年代并不频繁GC的场景下，是比较适用的。命名中用的是concurrent，而不是parallel，说明这个收集器是有与工作执行并发的能力的。MS则说明算法用的是Mark Sweep算法,来看看具体地工作原理,CMS整个过程比之前的收集器要复杂,整个过程分为四步:
- 初始标记(initial mark)
  - 单线程执行，需要“Stop The World”，但仅仅把GC Roots的直接关联可达的对象给标   记一下，由于直接关联对象比较小，所以这里的速度非常快。
- 并发标记(concurrent mark)
  - 对于初始标记过程所标记的初始标记对象，进行并发追踪标记，此时其他线程仍可以继  续工作。此处时间较长，但不停顿。
- 重新标记(remark)
  - 在并发标记的过程中，由于可能还会产生新的垃圾，所以此时需要重新标记新产生的垃  圾。此处执行并行标记，与用户线程不并发，所以依然是“Stop The World”，时间比   初始时间要长一点。
- 并发清除(concurrent sweep)
  - 并发清除之前所标记的垃圾。其他用户线程仍可以工作，不需要停顿。
- 缺点
  - Mark Sweep算法会导致内存碎片比较多
  - CMS的并发能力依赖于CPU资源，所以在CPU数少和CPU资源紧张的情况下，性能较差并发清除阶段，用户线程依然在运行，所以依然会产生新的垃圾，此阶段的垃圾并不会再本次GC中回收，而放到下次。所以GC不能等待内存耗尽的时候才进行GC，这样  的话会导致并发清除的时候，用户线程可以了利用的空间不足。所以这里会浪费一些内  存空间给用户线程预留。

G1收集器
  - G1 垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收。
  - 可以说是CMS的终极改进版，解决了CMS内存碎片、更多的内存空间登问题。虽然流程与CMS比较相似，但底层的原理已是完全不同。高效益优先。G1会预测垃圾回收的停顿时间，原理是计算老年代对象的效益率，优先回收最大效益的对象。
  - 堆内存结构的不同。以前的收集器分代是划分新生代、老年代、持久代等。

ZGC (JDK11)

AliGC （极致追求）后面是垃圾回收器整体图,上面是新生代,下面是老年代

![](/Jvav/img/garbage.png)

## 怎么查看服务器默认垃圾收集器是哪个？生产是如何配置垃圾收集器？谈谈你对垃圾收集器的理解？CMS你知道吗？

- 怎么查看服务器默认垃圾收集器是哪个？
  - Java -XX:+PrintCommandLineFlags
- Java 的 GC 回收的类型主要有：
  - UseSerialGC，UseParallelGC，UseConcMarkSweepGC，UseParNewGC，UseParallelOldGC，UseG1GC
  - Java 8 以后基本不使用 Serial Old
- 参数说明
  - DefNew : Default New Generation
  - Tenured : Old
  - ParNew : Parallel New Generation
  - PSYoungGen : Parallel Scavenge
  - ParOldGen : Parallel Old Generation
- Server/Client 模式分别是什么意思(了解,现在都是server)
  - 最主要的差别在于：-Server模式启动时，速度较慢，但是一旦运行起来后，性能将会有很大的提升。
  - 当虚拟机运行在-client模式的时候，使用的是一个代号为C1的轻量级编译器, 而-server模式启动的虚拟机采用相对重量级，代号为C2的编译器，C2比C1编译器编译的相对彻底，服务起来之后,性能更高。
  - 所以通常用于做服务器的时候我们用服务端模式，如果你的电脑只是运行一下java程序，就客户端模式就可以了。当然这些都是我们做程序优化程序才需要这些东西的，普通人并不关注这些专业的东西了。其实服务器模式即使编译更彻底，然后垃圾回收优化更好，这当然吃的内存要多点相对于客户端模式。
- 新生代
  - 串行 GC (Serial/ Serital Copying)
  - 并行 GC (ParNew)
  - 并行回收 GC (Parallel/ Parallel Scanvenge)
- 老年代
  - 串行 GC (Serial Old/ Serial MSC)
  - 并行 GC (Parallel Old/ Parallel MSC)
  - 并发标记清除 GC (CMS) 
    - 是一种以获取最短回收停顿时间为目标的收集器，适合应用在互联网站或者 B/S 系统的服务器上，这个类应用尤其重视服务器的响应速度，希望系统停顿时间最短。
    - CMS 非常适合堆内存大、CPU 核数多的服务器端应用，也是 G1 出现之前大型应用首选收集器。
    - 并发停顿比较少，并发指的是与用户线程一起执行。
    - 过程
      1. 初始标记（initail mark）：只是标记一下 GC Roots 能直接关联的对象，速度很快，需要暂停所有的工作线程
      2. 并发标记（concurrent mark 和用户线程一起）：进行 GC Roots 的跟踪过程，和用户线程一起工作，不需要暂停工作线程。
      3. 重新标记（remark）：为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程。
      4. 并发清除（concurrent sweep 和用户线程一起）：清除 GC 不可达对象，和用户线程一起工作，不需要暂停工作线程，基于标记结果，直接清除。由于耗时最长的并发标记和并发清除过程中，垃圾收集线程和用户线程可以一起并发工作，所以总体来看 CMS 收集器的内存回收和用户线程是一起并发地执行。
    - 优缺点
      - 优点：并发收集停顿低
      - 缺点：并发执行对 CPU 资源压力大，采用的标记清除算法会导致大量碎片
    - 由于并发进行， CMS 在收集与应用线程会同时增加对堆内存的占用，也就是说，CMS 必须要在老年代堆用尽之前完成垃圾回收，否者 CMS 回收失败，将触发担保机制，SerialOld串行老年代收集器将会以 STW 的方式进行一次 GC，从而造成较大的停顿时间(理论知道即可,实际已经被优化了)。
    - 标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐渐耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS 也提供了参数 -XX:CMSFullGCsBeForeCompaction (默认0，即每次都进行内存整理) 来指定多少次 CMS 收集之后，进行一次压
![](/Jvav/img/garbage2.png)
- 如何选择垃圾收集器
  - 单 CPU 或者小内存，单机程序：-XX:UseSerialGC
  - 多 CPU 需要最大吞吐量，如后台计算型应用：-XX:UseParallelGC 或者 -XX:UseParallelOldGC
  - 多 CPU 追求低停顿时间，需要快速响应，如互联网应用：-XX:+UseConcMarkSweepGC,新区会自动开启parNew;

## G1 垃圾收集器你了解吗？

### 以前收集器的特点

- 年轻代和老年代是各自独立且连续的内存块
- 年轻代收集器使用 eden + S0 + S1 进行复制算法
- 老年代收集必须扫描整个老年代区域
- 都是以尽可能的少而快速地执行 GC 为设计原则

### G1 是什么

- G1 是一种面向服务端的垃圾收集器，应用在多核处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能的满足垃圾收集器的暂停时间要求。
- 像 CMS 收集器一样，能与应用程序线程并发执行，整理空闲空间更快，需要更多的时间来预测 GC 停顿时间，不希望牺牲大量的吞吐性能，不需要更大的 JAVA Heap。
- G1 收集器的设计目的是取代 CMS 收集器，同时与 CMS 相比，G1 垃圾收集器是一个有整理内存过程的垃圾收集器，不会产生很多内存碎片。G1 的 Stop The World 更可控，G1 在停顿上添加了预测机制，用户可以指定期望的停顿时间。
G1 是在 2012 年才在 jdk.1.7u4 中可以呀用，在 jdk9 中将 G1 变成默认垃圾收集器来代替 CMS。它是以款面向服务应用的收集器。
- 主要改变是 Eden、Survivor 和 Tenured 等内存区域不再是连续的，而是变成了一个个大小一样的 region，每个 region 从 1M 到 32M 不等，一个 region 有可能属于 Eden、Survivor 或者 Tenured 内存区域。

### G1的特点

- G1 能充分利用多 CPU、多核环境硬件优势，尽量缩短 STW。
- G1 整体采用标记-整理算法，局部是通过是通过复制算法，不会产生内存碎片。
- 宏观上看 G1 之中不在区分年轻代和老年代，被内存划分为多个独立的子区域。
- G1 收集器里面讲整个的内存区域混合在一起，但其本身依然在小范围内要进行年轻代和老年代的区分。保留了新生代和老年代，但她们不在是物理隔离，而是一部分 Region 的集合且不需要 Region 是连续的，也就是说依然会采用不同的 GC 方式来处理不同的区域。
- G1 虽然也是分代收集器，但整个内存分区不存在物理上的年轻代和老年代的区别，也不需要完全独立的 Survivor to space 堆做复制准备。G1 只有逻辑上的分代概念，或者说每个分区都可能随 G1 的运行在不同代之间前后切换。
- heap里面只有heap和metaspace

### 底层原理

Region 区域化垃圾收集器：最大好处是化整为零，避免全内存扫描，只需要按照区域来进行扫描即可。
![](/Jvav/img/g1.png)

G1的内存结构和传统的内存空间划分有比较的不同。G1将内存划分成了多个大小相等的Region（默认是512K），Region逻辑上连续，物理内存地址不连续。同时每个Region被标记成E、S、O、H，分别表示Eden、Survivor、Old、Humongous。其中E、S属于年轻代，O与H属于老年代。

一些E和一个S经历一次gc,压缩,移动整合成一个单独的S;这样之前的空间就被清理出来了,而且没有碎片

H表示Humongous。从字面上就可以理解表示大的对象（下面简称H对象）。当分配的对象大于等于Region大小的一半的时候就会被认为是巨型对象。H对象默认分配在老年代，可以防止GC的时候大对象的内存拷贝。通过如果发现堆内存容不下H对象的时候，会触发一次GC操作。

### 生产环境服务器变慢，诊断思路和性能评估谈谈？

- 整机：top
- CPU：vmstat
- 内存：free
- 硬盘：df
- 磁盘IO：iostat
- 网络IO：ifstat

### 假如生产环境出现 CPU 过高，请谈谈你的分析思路和定位？

- 先用 top 命令找出 CPU 占比最高的进程pid
- ps -ef 或者 jps 进一步定位，得知是一个怎么样的一个后台程序
- 定位到具体的线程或代码
  - ps -mp 进程id -o THREAD,tid,time
  - -m 显示所有的线程
  - -p 进程使用cpu的时间
  - -o 该参数后是用户自定义格式
- 将需要的线程 ID 转化为 16 进制格式
- jstat <进程ID> | grep <线程ID(16进制)> -A60



## JMM

![](/Jvav/img/jmm.png)
Java Memory Model 是抽象的概念,定义了程序中各个变量的访问方式

jmm关于同步的规定
- 线程解锁前，必须把共享变量的值刷新回主内存
- 线程加锁前，必须读取主内存的最新值到自己的工作内存
- 加锁解锁是同一把锁


