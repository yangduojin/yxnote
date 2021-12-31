# IO

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

![io流的类别](/Jvav/img/ioStreamCategory.png)