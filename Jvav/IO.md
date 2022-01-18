# IO

- [IO](#io)
  - [File类](#file类)
    - [流的类别](#流的类别)
    - [流的类别总结](#流的类别总结)
  - [socket编程](#socket编程)

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

服务器端编程经常需要构造高性能的IO模型，常见的IO模型有四种：

1. 同步阻塞IO（Blocking IO）：即传统的IO模型。
   - 用户线程通过系统调用read发起IO读操作，由用户空间转到内核空间。内核等到数据包到达后，然后将接收的数据拷贝到用户空间，完成read操作。
   - 即用户需要等待read将socket中的数据读取到buffer后，才继续处理接收的数据。整个IO请求的过程中，用户线程是被阻塞的，这导致用户在发起IO请求时，不能做任何事情，对CPU的资源利用率不够。

    ```java
    {read(socket, buffer);
    process(buffer);}
    ```

2. 同步非阻塞IO（Non-blocking IO）：默认创建的socket都是阻塞的，非阻塞IO要求socket被设置为NONBLOCK。注意这里所说的NIO并非Java的NIO（New IO）库。
   - 同步非阻塞IO是在同步阻塞IO的基础上，将socket设置为NONBLOCK。这样做用户线程可以在发起IO请求后可以立即返回。
   - 由于socket是非阻塞的方式，因此用户线程发起IO请求时立即返回。但并未读取到任何数据，用户线程需要不断地发起IO请求，直到数据到达后，才真正读取到数据，继续执行。
   - 即用户需要不断地调用read，尝试读取socket中的数据，直到读取成功后，才继续处理接收的数据。整个IO请求的过程中，虽然用户线程每次发起IO请求后可以立即返回，但是为了等到数据，仍需要不断地轮询、重复请求，消耗了大量的CPU的资源。一般很少直接使用这种模型，而是在其他IO模型中使用非阻塞IO这一特性。

    ```java
    {while(read(socket, buffer) != SUCCESS);
    process(buffer);}
    ```

3. IO多路复用（IO Multiplexing）：即经典的Reactor设计模式，有时也称为异步阻塞IO，Java中的Selector和Linux中的epoll都是这种模型。
4. 异步IO（Asynchronous IO）：即经典的Proactor设计模式，也称为异步非阻塞IO。

**同步和异步**的概念描述的是用户线程与内核的交互方式：同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行；而异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成后会通知用户线程，或者调用用户线程注册的回调函数。

**阻塞和非阻塞**的概念描述的是用户线程调用内核IO操作的方式：阻塞是指IO操作需要彻底完成后才返回到用户空间；而非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。








