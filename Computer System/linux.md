# Linux

- [Linux](#linux)
  - [性能查看](#性能查看)
    - [整机](#整机)
      - [cpu](#cpu)
      - [内存 free和pidstat](#内存-free和pidstat)
      - [硬盘查看df -h(转化为单位制) 查看磁盘剩余空间数](#硬盘查看df--h转化为单位制-查看磁盘剩余空间数)
      - [磁盘IO查看iostat -xdk 2 3(2秒/共3次) 和 pidstat](#磁盘io查看iostat--xdk-2-32秒共3次-和-pidstat)
      - [网络IO查看 ifstat l](#网络io查看-ifstat-l)
    - [CPU占用过高的定位分析思路(遇到的印象深刻的问题)](#cpu占用过高的定位分析思路遇到的印象深刻的问题)
  - [常用命令](#常用命令)
  - [linux 配置](#linux-配置)
    - [配置基本信息](#配置基本信息)
    - [经验错误汇总](#经验错误汇总)
    - [虚拟机软件设置网关，ip，nat模式](#虚拟机软件设置网关ipnat模式)

## 性能查看

### 整机

按键盘数字“1”可以监控每个逻辑CPU的状况

top [-d number] | top [-bnp]: -d：number代表秒数，表示top命令显示的页面更新一次的间隔。默认是5秒。 -b：以批次的方式执行top。 -n：与-b配合使用，表示需要进行几次top命令的输出结果。 -p：指定特定的pid进程号进行观察。

在top命令显示的页面还可以输入以下按键执行相应的功能（注意大小写区分的）:  
?：显示在top当中可以输入的命令 P：以CPU的使用资源排序显示 M：以内存的使用资源排序显示 N：以pid排序显示 T：由进程使用的时间累计排序显示 k：给某一个pid一个信号。可以用来杀死进程 r：给某个pid重新定制一个nice值（即优先级） q：退出top（用ctrl+c也可以退出top）。

![Centos top命令结果](./img/topCommandInfo.png)

第一行
| 内容                           | 含义                                                                                   |
| ------------------------------ | -------------------------------------------------------------------------------------- |
| up 1min                        | 系统运行时间 格式为时：分                                                              |
| 1 users                        | 当前登录用户数                                                                         |
| load average: 0.00, 0.01, 0.05 | 系统负载，即任务队列的平均长度。 三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。 |

load average: 如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了

第二,三行为进程和CPU的信息,当有多个CPU时，这些内容可能会超过两行

| 内容         | 含义                                                 |
| ------------ | ---------------------------------------------------- |
| 132 total    | 进程总数                                             |
| 12 running   | 正在运行的进程数                                     |
| 131 sleeping | 睡眠的进程数                                         |
| 0 stopped    | 停止的进程数                                         |
| 0 zombie     | 僵尸进程数                                           |
| 0.2 us       | 用户空间占用CPU百分比                                |
| 0.7 sy       | 内核空间占用CPU百分比                                |
| 0.0 ni       | 用户进程空间内改变过优先级的进程占用CPU百分比        |
| 99.2 id      | 空闲CPU百分比                                        |
| 0.0 wa       | 等待输入输出的CPU时间百分比                          |
| 0.0 hi       | 硬中断（Hardware IRQ）占用CPU的百分比                |
| 0.0 si       | 软中断（Software Interrupts）占用CPU的百分比         |
| 0.0 st       | 用于有虚拟cpu的情况，用来指示被虚拟机偷掉的cpu时间。 |

top命令经常用来监控linux的系统状况，是常用的性能分析工具，能够实时显示系统中各个进程的资源占用情况。

- 主要看load average, CPU, MEN三部分
- load average表示系统负载，即任务队列的平均长度。 三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。三个值相加/3 * 100 > 60% 系统压力较大; 如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。
- 按键盘上面的1 查看cpu

uptime(top的精简版)

#### cpu

- vmstat -n 2 3 采样cpu等信息,每2秒一次,一共3次

procs

- r：运行和等待的CPU时间片的进程数，原则上1核的CPU的运行队列不要超过2，整个系统的运行队列不超过总核数的2倍，否则代表系统压力过大，我们看蘑菇博客测试服务器，能发现都超过了2，说明现在压力过大
- b：等待资源的进程数，比如正在等待磁盘I/O、网络I/O等

cpu

- us：用户进程消耗CPU时间百分比，us值高，用户进程消耗CPU时间多，如果长期大于50%，优化程序
- sy：内核进程消耗的CPU时间百分比
- us + sy 参考值为80%，如果us + sy 大于80%，说明可能存在CPU不足，从上面的图片可以看出，us + sy还没有超过百分80，因此说明蘑菇博客的CPU消耗不是很高
- id：处于空闲的CPU百分比
- wa：系统等待IO的CPU时间百分比
- st：来自于一个虚拟机偷取的CPU时间比

- mpstat -P ALL 2  查看看所有cpu核信息,2秒刷新

- pidstat -u 1 -p 进程编号,每个进程使用cpu的用量分解信息,1秒刷新

#### 内存 free和pidstat

应用程序可用内存数

经验值

- 应用程序可用内存l系统物理内存>70%内存充足
- 应用程序可用内存/系统物理内存<20%内存不足，需要增加内存
- 20%<应用程序可用内存/系统物理内存<70%内存基本够用

free -m/-g：兆/G

- pidstat -p 5101(进程号) -r 2 (采样间隔秒数,查看额外)

#### 硬盘查看df -h(转化为单位制) 查看磁盘剩余空间数

#### 磁盘IO查看iostat -xdk 2 3(2秒/共3次) 和 pidstat

iostat -xdk 2 3(2秒/共3次)

磁盘块设备分布

- rkB/s每秒读取数据量kB;wkB/s每秒写入数据量kB;
- svctm lO请求的平均服务时间，单位毫秒;
- await l/O请求的平均等待时间，单位毫秒;值越小，性能越好;
- util一秒中有百分几的时间用于I/O操作。接近100%时，表示磁盘带宽跑满，需要优化程序或者增加磁盘;**这个比较关键**
- rkB/s、wkB/s根据系统应用不同会有不同的值，但有规律遵循:长期、超大数据读写，肯定不正常，需要优化程序读取。
- svctm的值与await的值很接近，表示几乎没有IO等待，磁盘性能好。
- 如果await的值远高于svctm的值，则表示IO队列等待太长，需要优化程序或更换更快磁盘。

pidstat -d 2 -p 5101

#### 网络IO查看 ifstat l

```log
wget http://gael.roualland.free.fr/lifstat/ifstat-1.1.tar.gz
tar -xzvf ifstat-1.1.tar.gz
cd ifstat-1.1
./configure
make
make install
```

查看网络IO

各个网卡的in、out

观察网络负载情况程序

网络读写是否正常

- 程序网络I/O优化
- 增加网络I/O带宽

### CPU占用过高的定位分析思路(遇到的印象深刻的问题)

结合Linux和JDK命令一块分析

案例步骤

- 先用top命令找出CPU占比最高的
- ps -ef或者jps进一步定位，得知是一个怎么样的一个后台**进程**作搞屎棍
  - ps -ef|grep java|grep -v grep
  - jps -l |grep atguigu
- 定位到具体**线程**或者代码
  - ps -mp 进程编号 -o THREAD,tid,time
    - -m 显示所有的线程
    - -p pid进程使用cpu的时间
    - -o 该参数后是用户自定义格式
- 将需要的线程ID转换为16进制格式（英文小写格式），命令  printf "%x\n" 172 将172转换为十六进制(英文要小写)
- jstack **进程ID** | grep tid（16进制**线程ID**小写英文）-A60

将会打印是那个类多少行代码导致的问题

```log
ps - process status
-A Display information about other users’ processes, including those without controlling terminals.

-e Identical to -A.

-f Display the uid, pid, parent pid, recent CPU usage, process start time, controlling tty, elapsed CPU usage, and the associated command. If the -u option is also used, display the user name rather then the numeric uid. When -o or -O is used to add to the display following -f, the command field is not truncated as severely as it is in other formats.
```

[对于JDK自带的JVM监控和性能分析工具用过哪些?一般你是怎么用的?](https://blog.csdn.net/u011863024/article/details/106651068)

下表是Sun JDK监控和故障处理工具

| 名称   | 主要作用                                                                                                 |
| ------ | -------------------------------------------------------------------------------------------------------- |
| jps    | JVM Process Status Tool，显示指定系统内所有的HotSpot虚拟机进程                                           |
| jstat  | JVM Statistics Monitoring Tool，用于收集HotSpot虚拟机各方面的运行数据                                    |
| jinfo  | Configuration Info for Java，显示虚拟机配置信息                                                          |
| jmap   | Memory Map for Java，生成虚拟机的内存转储快照（Heapdump文件）                                            |
| jhat   | JVM Heap Dump Browser，用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结 | 果 |
| jstack | Stack Trace for Java，显示虚拟机的线程快照                                                               |

## 常用命令

- Man  or  --help
- Date
- Cal
- Pwd 查看当前完整路径
- Ls  -a -l ll
- netstat -nltp 看系统占用的端口号
- netstat -tunlp|grep 端口号    查看端口号，或查看全部
- source /etc/profiles  (虚拟机重新加载配置)
- lsof -i:xxx
- Ps -ef | grep xxx
- Mkdir -p
- Rmdir
- Crontab
- Tail -n5
- Tail -f 文件名  看日志,-f 实时滚动
- History
- Echo
- Rm  -rvf   -rf
- Touch
- Top   top / free  查看整机信息
- df -l
- Cp -r -v \cp
- Mv old n new n
- Mv /old dir /new dir
- More space / enter / q / ctrl f
- Cat *  ,  ** , * > *  ,  * > >*  , tac
- Less pageDown/Up  /  /?  N   n
- systemctl list-unit-files 查看开机启动项
- service mysqld stop
- kill -s 9 pid
- Free -m 查看内存和占用
- Chmod -R 777 更改权限
- Yum makecache
- find / -name "*maven*" / 下搜索所有匹配maven的
- ulimit -u 查看当前用户的可用线程数,普通是1024

## linux 配置

### 配置基本信息

虚拟机内部配置ip,网关等 vim  /etc/sysconfig/network-scripts/ifcfg-ens33

```conf
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
#IP和子网掩码
IPADDR=192.168.134.11
NETMASK=255.255.255.0
#网关和DNS服务器 要与虚拟机nat一致
GATEWAY=192.168.134.2
DNS1=8.8.8.8
DNS2=114.114.114.114
#IP地址的前24为代表网络地址，后面是主机地址
PREFIX=24
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="9b0a1ad0-dbac-4636-ad5e-098be246d2fa"
DEVICE="ens33"
ONBOOT="yes"
ZONE=public

# 配置一些别的信息 ,我暂时还没搞清楚是什么

JAVA_HOME=/opt/jdk1.8.0_221
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME PATH
export 
CATALINA_HOME=/opt/apache-tomcat-7.0.70
export PATH=$CATALINA_HOME/bin:$PATH

无网络时修改
Cd /etc/sysconfig/network-scripts/ 里面有eth0 eth1 两个网卡,看ip 修改那个 在里面加上网关地址,两个dns,wq保存后service network restart 重启
```

- systemctl status firewalld
- systemctl stop firewalld
- systemctl disable firewalld
- systemctl enable firewalld

### 经验错误汇总

安装完成后ifconfig无反应，cd /sbin,在sbin目录下输入ls，可见下图所示，并没有ifconfig。
安装  sudo yum install net-tools 即可

安装yum
直接yum update？
rpm -aq|grep yum|xargs rpm -e --nodeps 删除所有yum

### 虚拟机软件设置网关，ip，nat模式

1. 去虚拟软件右键网络配置选择nat模式
2. 选择还原默认设置
3. 查看左下角子网ip，再查看nat设置里网关ip，他两应该是在同一个网段，但是后面不一样192.168.zzz.0，192.168.zzz.2
4. 去vm8虚拟网卡，设置ip地址为192.168.zzz.1，介于软件子网ip和网关之间，子网掩码255，默认网关就是软件的网关192.168.zzz.2
5. 最后虚拟系统网络无论选nat还是自定义都可以ping通(需关闭防火墙)ping 宿主机或者网关都可以ping通
6. 但是系统里面要把网关设置的跟外面软件一样，ip最后一位可以变，但是需要重启192.168.zzz.11
