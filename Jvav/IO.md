# IO

- [IO](#io)
  - [File类](#file类)
    - [流的类别](#流的类别)
    - [流的类别总结](#流的类别总结)
  - [socket编程](#socket编程)
    - [零拷贝](#零拷贝)

## File类

File类的一个对象，代表一个文件或一个文件目录(俗称：文件夹)
File类中涉及到关于文件或文件目录的创建、删除、重命名、修改时间、文件大小等方法，
并未涉及到写入或读取文件内容的操作。如果需要读取或写入文件内容，必须使用IO流来完成。
后续File类的对象常会作为参数传递到流的构造器中，指明读取或写入的"终点".

File(String filePath) 常用构造器
File(String parentPath,String childPath)
File(File parentFile,String childPath)

windows和DOS系统默认使用“\”来表示 路径分隔符
UNIX和URL使用“/”来表示

File类的常用方法
public String getAbsolutePath();
public String getPath();
public String getName();
public String getParent();
public long length();
public long lastModified();// 上次修改时间 毫秒值

public Stirng[] list();// 指定目录下的所有文件或文件目录的名称数组
public File[] listFiles();// 指定目录下的所有文件或文件目录的File数组
public boolean renameTo(File dest);// 文件重命名为指定的文件路径

public boolean isDirectory();
public boolean isFile();
public boolean exists();
public boolean canRead();
public boolean canWrite();
public boolean isHidden();

public boolean createNewFile();// 此3方法，如果未写盘符则是项目默认路径
public boolean mkdir();// 存在则不创建，上层目录不存也不创建
public boolean mkdirs();// 存在则不创建，上层目录不存则一起创建

public boolean delete();// 删除文件或文件夹，不走回收站，删除目录里面不能有文件或目录

### 流的类别

| 分类       | 字节输入流              | 字节输出流               | 字符输入流            | 字符输出流             |
| ---------- | ----------------------- | ------------------------ | --------------------- | ---------------------- |
| 抽象基类   | **InputStream**         | **OutputStream**         | **Reader**            | **Writer**             |
| 访问文件   | **FileInputStream**     | **FileOutputStream**     | **FileReader**        | **FileWriter**         |
| 访问数组   | ByteArrayInputStream    | ByteArrayOutputStream    | CharArrayReader       | CharArrayWriter        |
| 访问管道   | PipedInputStream        | PipedOutputStream        | PipedReader           | PipedWriter            |
| 访问字符串 |                         |                          | StringReader          | StringWriter           |
| 缓冲流     | **BufferedInputStream** | **BufferedOutputStream** | **BufferedReader**    | **BufferedWriter**     |
| 转换流     |                         |                          | **InputStreamReader** | **OutputStreamWriter** |
| 对象流     | **ObjectInputStream**   | **ObjectOutputStream**   |                       |                        |
|            | FilterInputStream       | FilterOutputStream       | FilterReader          | FilterWriter           |
| 打印流     |                         | PrintStream              |                       | PrintWriter            |
| 推回输入流 | PushbackInputStream     |                          | PushbackReader        |                        |
| 特殊流     | DataInputStream         | DataOutputStream         |                       |                        |

![io流的类别](/Jvav/img/ioStreamCategory.png)

### 流的类别总结

| 抽象基类     | 节点流(或文件流)                              | 缓冲流(处理流的一种)                                       |
| ------------ | --------------------------------------------- | ---------------------------------------------------------- |
| InputStream  | FileInputStream (read(byte[] buffer))         | bufferedInputStream(read(byte[] buffer))                   |
| OutputStream | FileOutputStream (write(byte[] buffer,0,len)) | bufferedOutputStream(write(byte[] buffer,0,len) / flush()) |
| Reader       | FileReader(read(char[]cbuf))                  | BufferedReader(read(char[]cbuf) / readLine())              |
| writer       | FileWriter(write (char[] cbuf,0,len))         | BufferedWriter(write(char[] cbuf,0,len) / flush())         |

输入过程

1. 创建File类的对象，指明读取的数据的来源。（要求此文件一定要存在）
1. 创建相应的输入流，将File类的对象作为参数，传入流的构造器中
1. 具体的读入过程：创建相应的byte[] 或 char[]。
1. 关闭流资源
说明：程序中出现的异常需要使用try-catch-finally处理。

输出过程

1. 创建File类的对象，指明写出的数据的位置。（不要求此文件一定要存在）
1. 创建相应的输出流，将File类的对象作为参数，传入流的构造器中
1. 具体的写出过程： write(char[]/byte[] buffer,0,len)
1. 关闭流资源
说明：程序中出现的异常需要使用try-catch-finally处理。

## socket编程

![IO参考原文](https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247483907&idx=1&sn=3d5e1384a36bd59f5fd14135067af1c2&chksm=fb0be897cc7c61815a6a1c3181f3ba3507b199fd7a8c9025e9d8f67b5e9783bc0f0fe1c73903&scene=21#wechat_redirect)

服务器端编程经常需要构造高性能的IO模型，常见的IO模型有四种：

1. 阻塞IO（Blocking IO）：即传统的IO模型。
   - 用户线程通过系统调用read发起IO读操作，由用户空间转到内核空间。内核等到数据包到达后，然后将接收的数据拷贝到用户空间，完成read操作。
   - 即用户需要等待read将socket中的数据读取到buffer后，才继续处理接收的数据。整个IO请求的过程中，用户线程是被阻塞的，这导致用户在发起IO请求时，不能做任何事情，对CPU的资源利用率不够。
   - 当用户进程调用了recvfrom这个系统调用，内核就开始了IO的第一个阶段：等待数据准备。对于network io来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候内核就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当内核一直等到数据准备好了，它就会将数据从内核中拷贝到用户内存，然后内核返回结果，用户进程才解除block的状态，重新运行起来。所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

    ```java
    {read(socket, buffer);
    process(buffer);}
    ```

2. 非阻塞IO（Non-blocking IO）：
   - 当用户进程调用recvfrom时，系统不会阻塞用户进程，而是立刻返回一个ewouldblock错误，从用户进程角度讲 ，并不需要等待，而是马上就得到了一个结果。用户进程判断标志是ewouldblock时，就知道数据还没准备好，于是它就可以去做其他的事了，于是它可以再次发送recvfrom，一旦内核中的数据准备好了。并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。
   - 当一个应用程序在一个循环里对一个非阻塞调用recvfrom，我们称为轮询。应用程序不断轮询内核，看看是否已经准备好了某些操作。这通常是浪费CPU时间，但这种模式偶尔会遇到。

    ```java
    {while(read(socket, buffer) != SUCCESS);
    process(buffer);}
    ```

3. IO多路复用（IO Multiplexing）：即经典的Reactor设计模式，有时也称为异步阻塞IO，Java中的Selector和Linux中的epoll都是这种模型。IO multiplexing这个词可能有点陌生，但是如果我说select，epoll，大概就都能明白了。有些地方也称这种IO方式为event driven IO。我们都知道，select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程
4. 信号驱动的IO
5. 异步IO（Asynchronous IO）：即经典的Proactor设计模式，也称为异步非阻塞IO。

**同步和异步**的概念描述的是用户线程与内核的交互方式：同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行；而异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成后会通知用户线程，或者调用用户线程注册的回调函数。

**阻塞和非阻塞**的概念描述的是用户线程调用内核IO操作的方式：阻塞是指IO操作需要彻底完成后才返回到用户空间；而非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。

### 零拷贝

[文章参考,非常详细](https://mp.weixin.qq.com/s/5MdF4dt0X1anqtFPoLOKeA)

1. 将静态图片展示给客户(先将静态内容从磁盘中拷贝出来放到一个内存buf中，然后将这个buf通过socket传输给用户，进而用户或者静态内容的展示)
   - 首先，调用read时，文件A拷贝到了kernel模式；
   - 之后，CPU控制将kernel模式数据copy到user模式下；
   - 调用write时，先将user模式下的内容copy到kernel模式下的socket的buffer中；
   - 最后将kernel模式下的socket buffer的数据copy到网卡设备中传送；
2. Linux 2.1内核开始引入了sendfile函数（上一节有提到）,用于将文件通过socket传送(该函数通过一次系统调用完成了文件的传送，减少了原来read/write方式的模式切换。此外更是减少了数据的copy)
   - 通过sendfile传送文件只需要一次系统调用，当调用sendfile时：
   - 首先（通过DMA）将数据从磁盘读取到kernel buffer中；
   - 然后将kernel buffer拷贝到socket buffer中；
   - 最后将socket buffer中的数据copy到网卡设备（protocol engine）中发送；
3. Linux2.4 内核对sendfile做了改进
   - 将文件拷贝到kernel buffer中；
   - 向socket buffer中追加当前要发生的数据在kernel buffer中的位置和偏移量；
   - 根据socket buffer中的位置和偏移量直接将kernel buffer的数据copy到网卡设备（protocol engine）中；

经过上述过程，数据只经过了2次copy就从磁盘传送出去了。这个才是真正的Zero-Copy(这里的零拷贝是针对kernel来讲的，数据在kernel模式下是Zero-Copy)。
