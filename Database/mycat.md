# mycat

- [mycat](#mycat)
  - [基础](#基础)
    - [基本功能](#基本功能)
    - [Mycat读写分离](#mycat读写分离)
    - [分库分表](#分库分表)

## 基础

### 基本功能

mycat 消息中间件

- 数据库读写分离
- 读写分离，写主表，读从表
- 主故障时，自动切换到另一个主上面,配置多主多从模式

数据切片

- 水平切分: 根据表中数据的逻辑关系，将同一个表中的数据按照某种条件拆分到多台数据库服务器上面，一张表里面的数据分散到多个表里面，按照年/月/日分表(分表)
- 垂直切分: 按照不同的表来切分到不同的数据库服务器之上，一个数据库里面的多个表分散到多个数据库里面(分库)

总结

- 性能有瓶颈了，可以读写分离
- 数据库容量有瓶颈了，可以分库分表
- 能整合MySQL，Oracle，DB2, Redis等数据库

原理: 拦截发来的SQL语句,对SQL分析:如分片分析、路由分析、读写分离分析、缓存分析等，将此SQL 发往后端的真实数据库，并将返回的结果做适当的处理，最终再返回给用户

安装: 上传到software目录下执行

- yum install -y java-1.8.0-openjdk-devel.x86_64(jdk8，安装成功后jps如果有反应，就是jdk安装成功)
- tar –zxvf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz

启动 / 停止

- 切换到mycat的bin路径下，执行    ./mycat start 或者 bin/mycat start
- 切换到mycat的bin路径下，执行    ./mycat stop

mycat命令行

- 登录mycat命令行，使用mysql的命令行工具来操作的：    mysql -umycat -p -P8066 -h127.0.0.1     **mycat默认数据访问端口是8066**

配置文件

server.xml 常用配置

- 配置序列生成方式
- 配置mycat逻辑数据库，表 分片
- 配置mycat的访问账户和密码

schema.xml

- 用于配置的逻辑数据库的映射、表、分片规则、数据结点及真实的数据库信息；
- 常用配置：
- 配置逻辑库映射
- 配置垂直切分的表
- 配置真实的数据库
- 配置读写结点

rule.xml

- 定义分片规则

### Mycat读写分离

配置server.xml文件

```xml
    <property name="password">123</property>
    <property name="schemas">mycat</property>
```

配置schema文件

- ``<schema>`` 的 dataNode 要与 ``<dataNode>`` 名字匹配，``<dataNode>`` 中的database要与真实数据库的名字一致 dataHost要与 ``<dataHost>`` 名字一致 balance改为3 下面的writeHost与readHost要与主从表的端口,账号,密码一致
- 之后要chmod 777 schema.xml 修改权限
- 只做读写分离，不做分库分表，Mycat只是帮我们转发一下请求，读转发到从库，写转发到主库，则schema标签里面不用配置table给schema标签加上属性dataNode，配置dataNode的名字(name)

配置dataNode

dataNode定义了Mycat中的数据节点，也就是我们通常说所的数据分片，一个dataNode标签就是一个独立的数据分片，通俗理解，一个分片就是一个物理数据库

- name: 定义数据节点的名字，这个名字需要是唯一的，这个名字在schema里面会使用到；
- dataHost: 用于定义该分片属于哪个数据库实例的,属性值是引用dataHost标签上定义的name属性
- database: 用于对应真实的数据库名，必须是真实存在的；
  
最终配置如下 \<dataNode name="dn1" dataHost="localhost1" database="test" />

配置dataHost

balance

- balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的writeHost上;
- balance="1"，全部的readHost与stand by writeHost参与select语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且M1与 M2互为主备)，正常情况下，M2,S1,S2都参与select语句的负载均衡。
- balance="2"，所有读操作都随机的在writeHost、readhost上分发
- balance="3"，所有读请求随机的分发到wiriterHost对应的readhost执行，writerHost不负担读压力

writeType 已过时，1.6版本就不用了

switchType属性  用于指定主服务器发生故障后的切换类型

- -1 表示不自动切换
- 1 默认值，自动切换（推荐）
- 2 基于MySQL主从同步的状态决定是否切换
- 3 基于MySQL galary cluster的切换机制（适合集群）（1.4.1）

通常情况下，我们MySQL采用双主双从的模式下，switchType为1即可。因为双主从模式下，主从同步关系很复杂，不能根据MySQL的状态来切换。只需要在一个主出问题后，切换到另外的主。

heartbeat标签

- 用于和后端数据库进行心跳检查的语句，检测MySQL数据库是否正常运行
- 当switchType为1时，mysql心跳检查语句是select user()
- 当switchType为2时，mysql心跳检查语句是show slave status
- 当switchType为3时，mysql心跳检查语句是show status like 'wsrep%'

writeHost与readHost标签

- 这两个标签都指定后端数据库的相关配置给mycat，用于实例化后端连接池。唯一不同的是，writeHost指定写实例、readHost指定读实例，组合这些读写实例来满足系统的要求。
- 在一个dataHost内可以定义多个writeHost和readHost。但是，如果writeHost指定的后端数据库宕机，那么这个writeHost绑定的所有readHost都将不可用。另一方面，由于这个writeHost宕机系统会自动的检测到，并切换到备用的writeHost上去。

测试读写分离

- 在主表写入一个数据，在从表修改这个数据，查询看返回的是主从表那个的数据

### 分库分表

分库分表(垂直)  一个数据库里面的多个表分散到多个数据库里面(分库)

分库原则
    Join操作原则 多个数据库之间表join是不行的(跨库join行不同)，所以只要表之间有点关联就不能分到不同的库  
    分析,客户表,客户登录之后用户信息存放在session中,通过session我们可以获取到  customer_id 就可以单独分库

总结: 垂直切分带来的价值：可以屏蔽掉多数据源的问题，只需要一个统一入口mycat就可以操作下面的多个数据库一个数据库里面的多个表分散到多个数据库里面(分库)

分库分表(水平)

一张表里面的数据分散到多个表里面，按照年/月/日分表(分表)

分库分表(全局表)

设定为全局的表，会直接复制给每个数据库一份，所有写操作也会同步给多个库。所以全局表一般不能是大数据表或者更新频繁的表，一般是字典表或者系统表为宜。

全局序列号

- 本地文件方式
- 时间戳方式
- 数据库方式(推荐)：这里是数据库方式生成主键ID，不是采用数据库的主键自增，而是mycat利用mysql数据库生成一个主键
