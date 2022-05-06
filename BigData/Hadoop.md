# Hadoop

- [Hadoop](#hadoop)
  - [模板机](#模板机)
    - [用户权限](#用户权限)
    - [配置与安装](#配置与安装)
    - [hadoop配置文件](#hadoop配置文件)
    - [启动停止集群,myhadoop,jpsall](#启动停止集群myhadoopjpsall)
  - [HDFS](#hdfs)
  - [MapReduce](#mapreduce)
  - [YARN](#yarn)

## 模板机

### 用户权限

关闭防火墙，关闭防火墙开机自启  
[root@hadoop100 ~]# systemctl stop firewalld  
[root@hadoop100 ~]# systemctl disable firewalld.service

创建atguigu用户，并修改atguigu用户的密码  
[root@hadoop100 ~]# useradd atguigu  
[root@hadoop100 ~]# passwd atguigu

配置atguigu用户具有root权限，方便后期加sudo执行root权限的命令
[root@hadoop100 ~]# vim /etc/sudoers
修改/etc/sudoers文件，在%wheel这行下面添加一行，如下所示：

  ```bash
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)     ALL

  ## Allows people in group wheel to run all commands
  %wheel  ALL=(ALL)       ALL
  atguigu   ALL=(ALL)     NOPASSWD:ALL
  ```

注意：atguigu这一行不要直接放到root行下面，因为所有用户都属于wheel组，你先配置了atguigu具有免密功能，但是程序执行到%wheel行时，该功能又被覆盖回需要密码。所以atguigu要放到%wheel这行下面。

在/opt目录下创建文件夹，并修改所属主和所属组
（1）在/opt目录下创建module、software文件夹
[root@hadoop100 ~]# mkdir /opt/module
[root@hadoop100 ~]# mkdir /opt/software
 （2）修改module、software文件夹的所有者和所属组均为atguigu用户
[root@hadoop100 ~]# chown atguigu:atguigu /opt/module
[root@hadoop100 ~]# chown atguigu:atguigu /opt/software
（3）查看module、software文件夹的所有者和所属组
[root@hadoop100 ~]# cd /opt/
[root@hadoop100 opt]# ll
总用量 12
drwxr-xr-x. 2 atguigu atguigu 4096 5月  28 17:18 module
drwxr-xr-x. 2 root    root    4096 9月   7 2017 rh
drwxr-xr-x. 2 atguigu atguigu 4096 5月  28 17:18 software

### 配置与安装

虚拟机配置如下

  ```txt
  BOOTPROTO=static
  IPADDR=192.168.10.102
  GATEWAY=192.168.10.2
  DNS1=192.168.10.2
  ```

vim /etc/hostname  hadoop102

vim /etc/hosts  (这是修改linux,windows也要添加到host)
192.168.10.100 hadoop100
192.168.10.101 hadoop101
192.168.10.102 hadoop102
192.168.10.103 hadoop103
192.168.10.104 hadoop104
192.168.10.105 hadoop105
192.168.10.106 hadoop106
192.168.10.107 hadoop107
192.168.10.108 hadoop108

宿主机vm8配置
默认网关192.168.10.2
首选dns 192.168.10.2
备用dns 8.8.8.8

最小安装需要加装

- yum install -y epel-release  (Extra Packages for Enterprise Linux是为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS和Scientific Linux。相当于是一个软件仓库，大多数rpm包在官方 repository 中是找不到的）)
- yum install -y net-tools
- yum install -y vim
- yum install rsync 同步文件
- sudo yum -y install ntp 同步时间

安装jdk

- tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
- ``sudo vim /etc/profile.d/my_env.sh``  新建/etc/profile.d/my_env.sh文件, 添加如下信息后, 运行 ``source /etc/profile``, 验证是否成功 ``java -version``, 不能用可以重启试试

  ```bash
  #JAVA_HOME
  export JAVA_HOME=/opt/module/jdk1.8.0_212
  export PATH=$PATH:$JAVA_HOME/bin
  ```

群发拷贝

- scp -r /opt/module/jdk1.8.0_212  atguigu@hadoop103:/opt/module 安全拷贝,原始
- rsync -av hadoop-3.1.3/ atguigu@hadoop103:/opt/module/hadoop-3.1.3/ 远程同步工具,进阶
- xsync集群分发脚本, 一步到位

脚本

- cd /home/atguigu/bin (没有就新建)
- vim xsync

  ```bash
  # !/bin/bash

  # 1. 判断参数个数
  if [ $# -lt 1 ]
  then
      echo Not Enough Arguement!
      exit;
  fi

  # 2. 遍历集群所有机器
  for host in hadoop102 hadoop103 hadoop104
  do
      echo ====================  $host  ====================
      #3. 遍历所有目录，挨个发送

      for file in $@
      do
          #4. 判断文件是否存在
          if [ -e $file ]
              then
                  #5. 获取父目录
                  pdir=$(cd -P $(dirname $file); pwd)

                  #6. 获取当前文件的名称
                  fname=$(basename $file)
                  ssh $host "mkdir -p $pdir"
                  rsync -av $pdir/$fname $host:$pdir
              else
                  echo $file does not exists!
          fi
      done
  done
  ```

- chmod +x xsync (修改脚本 xsync 具有执行权限)
- xsync /home/atguigu/bin 测试脚本
- sudo cp xsync /bin/  将脚本复制到/bin中，以便全局调用
- sudo ./bin/xsync /etc/profile.d/my_env.sh  同步环境变量配置（root所有者）,注意：如果用了sudo，那么xsync一定要给它的路径补全。

SSH

1. ssh hadoop103
1. cd /home/atguigu/.ssh
1. ssh-keygen -t rsa 三次回车
1. ssh-copy-id hadoop102 拷贝公钥到无密登录的电脑(多台电脑相互之间无密,就要每台电脑都发公钥,也要给自己发)

### hadoop配置文件

路径:$HADOOP_HOME/etc/hadoop

- core-site.xml

  ```xml
  <configuration>
      <!-- 指定NameNode的地址 -->
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://hadoop102:8020</value>
      </property>

      <!-- 指定hadoop数据的存储目录 -->
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/opt/module/hadoop-3.1.3/data</value>
      </property>

      <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
      <property>
          <name>hadoop.http.staticuser.user</name>
          <value>atguigu</value>
      </property>
  </configuration>
  ```

- hdfs-site.xml

  ```xml
  <configuration>
  <!-- nn web端访问地址-->
  <property>
          <name>dfs.namenode.http-address</name>
          <value>hadoop102:9870</value>
      </property>
  <!-- 2nn web端访问地址-->
      <property>
          <name>dfs.namenode.secondary.http-address</name>
          <value>hadoop104:9868</value>
      </property>
  </configuration>
  ```

- yarn-site.xml

  ```xml
  <configuration>
      <!-- 指定MR走shuffle -->
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>

      <!-- 指定ResourceManager的地址-->
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>hadoop103</value>
      </property>

      <!-- 环境变量的继承 -->
      <property>
          <name>yarn.nodemanager.env-whitelist</name>
          <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
      </property>
      <!-- 开启日志聚集功能 -->
      <property>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
      </property>
      <!-- 设置日志聚集服务器地址 -->
      <property>  
          <name>yarn.log.server.url</name>  
          <value>http://hadoop102:19888/jobhistory/logs</value>
      </property>
      <!-- 设置日志保留时间为7天 -->
      <property>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>604800</value>
      </property>
  </configuration>
  ```

- mapred-site.xml

  ```xml
  <configuration>
  <!-- 指定MapReduce程序运行在Yarn上 -->
      <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
      </property>
      <!-- 历史服务器端地址 -->
      <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop102:10020</value>
      </property>

      <!-- 历史服务器web端地址 -->
      <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>hadoop102:19888</value>
      </property>
  </configuration>
  ```

- xsync /opt/module/hadoop-3.1.3/etc/hadoop/  分发配置文件
- vim /opt/module/hadoop-3.1.3/etc/hadoop/workers (垂直添加hadoop102 hadoop103 hadoop104,注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。)
- xsync /opt/module/hadoop-3.1.3/etc (同步所有节点配置文件)

### 启动停止集群,myhadoop,jpsall

如果集群是第一次启动，需要在hadoop102节点格式化NameNode（注意：格式化NameNode，会产生新的集群id，导致NameNode和DataNode的集群id不一致，集群找不到已往数据。如果集群在运行过程中报错，需要重新格式化NameNode的话，一定要先停止namenode和datanode进程，并且要删除所有机器的data和logs目录，然后再进行格式化。）

- hdfs namenode -format
- sbin/start-dfs.sh
- sbin/start-yarn.sh (在配置了ResourceManager的节点（hadoop103）启动YARN)

历史服务器

- mapred --daemon start historyserver (启动历史服务器)
- sbin/stop-yarn.sh
- mapred --daemon stop historyserver
- start-yarn.sh
- mapred --daemon start historyserver
- hadoop fs -rm -r /output  (删除HDFS上已经存在的输出文件)

启动/停止

- start-dfs.sh/stop-dfs.sh  整体启动/停止HDFS
- start-yarn.sh/stop-yarn.sh  整体启动/停止YARN
- hdfs --daemon start/stop namenode/datanode/secondarynamenode 分别启动/停止HDFS组件
- yarn --daemon start/stop  resourcemanager/nodemanager 启动/停止YARN

- **集群启停脚本**
  1. cd /home/atguigu/bin
  2. vim myhadoop.sh
  3. chmod +x myhadoop.sh

  ```bash
  #!/bin/bash

  if [ $# -lt 1 ]
  then
      echo "No Args Input..."
      exit ;
  fi

  case $1 in
  "start")
          echo " =================== 启动 hadoop集群 ==================="

          echo " --------------- 启动 hdfs ---------------"
          ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
          echo " --------------- 启动 yarn ---------------"
          ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
          echo " --------------- 启动 historyserver ---------------"
          ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
  ;;
  "stop")
          echo " =================== 关闭 hadoop集群 ==================="

          echo " --------------- 关闭 historyserver ---------------"
          ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
          echo " --------------- 关闭 yarn ---------------"
          ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
          echo " --------------- 关闭 hdfs ---------------"
          ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
  ;;
  *)
      echo "Input Args Error..."
  ;;
  esac
  ```

- **查看三台服务器Java进程脚本**
  1. cd /home/atguigu/bin
  2. vim jpsall
  3. chmod +x jpsall
  4. xsync /home/atguigu/bin/

  ```bash
  #!/bin/bash

  for host in hadoop102 hadoop103 hadoop104
  do
          echo =============== $host ===============
          ssh $host jps 
  done
  ```

|常用端口名称(面试)| Hadoop2.x | Hadoop3.x|
|---|--|--|
|NameNode内部通信端口| 8020 / 9000 | 8020 / 9000 / 9820|
|NameNode HTTP UI| 50070 | 9870|
|MapReduce查看执行任务端口| 8088| 8088|
|历史服务器通信端口| 19888| 19888|

- **主机器ntpd时间服务器配置（必须root用户）虚拟机不用配:-(**
  1. sudo systemctl status ntpd
  2. sudo systemctl start ntpd
  3. sudo systemctl is-enabled ntpd
  4. sudo vim /etc/ntp.conf
     1. 将#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap 修改为为restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap
     2. 修改4个配置,集群在局域网中，不使用其他互联网上的时间
        1. #server 0.centos.pool.ntp.org iburst
        2. #server 1.centos.pool.ntp.org iburst
        3. #server 2.centos.pool.ntp.org iburst
        4. #server 3.centos.pool.ntp.org iburst

     3. 添加server 127.127.1.0
        1. fudge 127.127.1.0 stratum 10
     4. sudo vim /etc/sysconfig/ntpd 修改hadoop102的/etc/sysconfig/ntpd 文件
        1. 增加内容 SYNC_HWCLOCK=yes
     5. 重启和开机自启 sudo systemctl start ntpd | sudo systemctl enable ntpd
- **其他机器执行**
  1. sudo systemctl stop ntpd
  2. sudo systemctl disable ntpd
  3. sudo crontab -e 设置同步时间和机器
  4. ``*/1* ** * /usr/sbin/ntpdate hadoop102``
  5. sudo date -s "2021-9-11 11:11:11" 随意修改时间
  6. sudo date

## HDFS

|名称|章节|内容|
|--|--|--|
|HDFS|概述|HDFS的产生背景和定义,优缺点,组成,文件块大小|
||HDFD的shell相关操作|开发的重点|
||HDFS的客户端api|数据的上传和下载|
||HDFS的读写流程|面试重点|
||NN和2NN|高可用只用NN,不要2NN|
||Datanode工作机制|了解|

![HDFS的组成架构](./img/hdfsarchitecture.png)

## MapReduce

## YARN
