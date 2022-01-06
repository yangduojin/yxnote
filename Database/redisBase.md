# Redis

- [Redis](#redis)
  - [数据结构](#数据结构)
    - [存入StringJSON的转换和提取](#存入stringjson的转换和提取)
  - [持久化](#持久化)
  - [删除策略](#删除策略)
    - [缓存淘汰](#缓存淘汰)
    - [缓存一致性](#缓存一致性)
  - [Redis / 数据库 双写一致性](#redis--数据库-双写一致性)
    - [避免脏数据](#避免脏数据)
    - [结论](#结论)
    - [选择](#选择)
  - [穿透 & 雪崩 & 击穿 等](#穿透--雪崩--击穿-等)
    - [无锁双缓存](#无锁双缓存)
  - [Redis发布订阅](#redis发布订阅)
  - [分布式锁](#分布式锁)
  - [redlock](#redlock)
  - [事务](#事务)
  - [4种缓存](#4种缓存)
  - [redis集群](#redis集群)
    - [哨兵](#哨兵)
    - [主从复制](#主从复制)
    - [集群分布](#集群分布)
    - [redis集群使用时有什么注意事项](#redis集群使用时有什么注意事项)
  - [Redis 安装和指令](#redis-安装和指令)
    - [Redis key](#redis-key)
    - [String](#string)
    - [Hash](#hash)
    - [List](#list)
    - [Set](#set)
    - [Zset](#zset)

## 数据结构

1. ``String``
   - SDS 简单动态字符串 空间换时间
   - 单键值最大512m,二进制安全,如数字,字符串,jpg图片或者序列化的对象,spring session + redis实现session共享
2. ``List`` 队列结构
   - Ziplist
     - 连续内存
     - 特殊编码
     - 节省内存
     - 时间换空间
   - Quicklist
     - ![redisQuicklist](/Database/img/redisQuicklist.png)
     - 链表和ziplist的组合
   - 链表,有序,简单的字符串列表，按插入顺序排序,利用lrange命令做基于redis的分页功能,性能极好,微博的时间轴，有人发布微博，用lpush加入时间轴，展示新的列表信息
3. ``Hash`` 适合存对象
   - Ziplist
   - Dict
     - ![redisDict](/Database/img/redisDict.png)
     - 渐进式rehash
     - murmurhash
   - hash 键值对集合，hashkey field value映射表，适合存储对象(用户信息),视频信息,比string节约空间,可模拟session
4. ``Set``
   - Inset
     - 当value是数字
     - 当size没有超过阈值
       - 数组项从小到大排序
       - 二分查找
     - 时间换空间
   - Dict
   - string类型的无序集合，不可重复，可以做全局去重，交集并集差集可以计算喜好,点赞,点踩,收藏等
5. ``Sorted Set``
   - Hash + SkipTable
     - ![redisSkiptable](/Database/img/redisSkiptable.png)
   - Zset (sorted set)string类型有序集合，不可重复，每个元素需要分数，分数排序，权重排行并应用
6. Stream流
   - RadixTree
7. bitmap位图
8. pubsub
9. geo地理位置
10. hyperloglog基数统计

### 存入StringJSON的转换和提取

- 存
  - ``String jsonString = JSON.toJSONString(catelogJsonFromDB);``
  - ``stringRedisTemplate.opsForValue().set("catalogJSON",jsonString);``
- 取
  - ``Map<String, List<Catelog2Vo>> result = JSON.parseObject(catalogJSON,new TypeReference< Map<String, List<Catelog2Vo>> >(){});``

## 持久化

- RDB
  - 快照读,fork一个进程，先将数据写入临时文件中，待上次持久化结束后，会将该临时文件替换场次的持久化文件，比aof效率高，但是最后一次数据可能会丢失
  - 每个redis实例只会存一份rdb文件
  - 可以通过save,bgsave来调用
  - 二进制文件,lzf
  - Fork：在linux中，fork()跟主进程一样的子进程，效率考虑，主,子进程会公用一段物理内存，当发生改变的时候，才会把主进程"写时复制"一份给子进程
- AOF
  - 类似 binlog 机制,可以做到不丢数据,默认不开启，每秒记录一次，宕机该秒数据可能会消失，超过一定大小就会将之前的数据转为rdb保存
  - 每次数据操作调用 flushAppendOnlyFile 文件来刷新aof
  - 每次操作都需要 fsync ,前台线程阻塞
    - always
    - every sec

- 混合模式 RDB+AOF
- 加载顺序
  - 先AOF
  - 后RDB

优缺点

- rdb：节约磁盘，恢复速度快 / 数据太大时消耗性能，宕机最后一次可能没保存
- Aof： 备份机制稳健、可读日志，可以处理事务 / 更占磁盘、速度慢、同步日志性能差

## 删除策略

- 惰性删除: 是在使用某个key前，redis会检查一下，这个key如果设置了过期时间是否过期，过期就删除，但是可能有key一直没删除也没被使用，内存会越来越高，这时就需要缓存淘汰
- 定时删除
- 定期删除策略是前两种策略的折中：100s  
  - 定期删除策略每隔一段时间执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。
  - 周期性轮询Redis库中的时效性数据，来用随机抽取的策略，利用过期数据占比的方式控制删除频度
  - 特点1：CPU性能占用设置有峰值，检测频度可自定义设置
  - 特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理
  - 总结：周期性抽查存储空间（随机抽查，重点抽查）

三种策略都有问题,需要一个兜底方案,所以**缓存淘汰**登场了

### 缓存淘汰

- ``volatile-lru(default)`` 从设置过期数据集里查找最近最少使用
- ``volatile-lfu`` 对所有设置了过期时间的key使用LFU算法进行删除
- ``volatile-ttl`` 从设置过期数据集里清理已经过期的key
- ``volatile-random`` 从设置过期数据集里任意选择数据淘汰
- ``allkeys-lfu`` 对所有key使用LFU算法进行删除 Least frequently used
- ``allkeys-lru`` 从数据集中挑选最近最少使用的数据淘汰 Least frequently used 推荐
- ``allkeys-random`` 从数据集中任意选择数据淘汰
- ``no-enviction`` 不清理,内存不足写入会报错

没有设置过期时间的key，volatile就会跟no-enviction一样报错

配置

- 命令 config set maxmemory-policy noeviction
- config get maxmemory
- maxmemory-policy volatile-lru(Least Recently Used)

配置文件 - 配置文件redis.conf的maxmemory-policy参数

### 缓存一致性

1. ``write cache -> write db`` 写数据库失败时
2. ``write db -> write cache`` 更新cache失败,并发出现脏数据
3. ``evict cache -> write db`` 延迟,并发出现脏数据
4. ``write db -> evict cache`` 并发出现脏数据
5. ``evict cache -> write db -> evict cache`` 减少脏数据概率

## Redis / 数据库 双写一致性

缓存设置过期时间

- 缓存只能用于最终一致性，强一致性不能用缓存，且只能降低不一致的概率，不能避免
- 采用正确的更新策略，先更新数据库，再删除缓存，其次可能存在删除缓存失败的问题，提供一个补偿措施如消息队列，确认删除成功
- 或者给缓存设置过期时间，所有的写操作以数据库为准，缓存过期读操作自然会更新缓存

不设置过期时间

1. 先更新数据库，再更新缓存
   - 普遍反对，线程不安全，
2. 先删除缓存，再更新数据库
   - AB一个更改数据库一个查询，导致放入缓存中的值不一样，采用延时双删策略:先删除缓存，写入数据库，再延迟一段时间，再删除缓存，延迟的时间自己决定。保证写请求可以删除读请求造成的缓存脏数据
   - 如果是读写分离架构，睡眠时间修改为主从同步的延时时间基础上，加上几百ms，如果吞吐量降低就用异步删除
3. 先更新数据库，再删除缓存
   - 先从缓存中查，没有就去数据库查，成功后放入缓存
   - 先从数据库中查，查到直接返回
   - 先把数据存入数据库，成功后再让缓存失效
   - 极小的概率出现脏数据，可采用延时双删

删除缓存失败的重试机制

- 业务代码发消息队列，不停的重试删除，直到成功(侵入性太强)
- 业务代码更新数据库，数据库将操作写入binlog日志中，订阅程序提取出所需要的数据以及key
- 另起一段非业务代码，获取该信息，尝试删除缓存，失败就放入消息队列中，重新从消息队列中获得该数据，重试删除操作

### 避免脏数据

- ttl
- 定时更新
  - 有缓存就有缓存同步的问题，我们可以引入缓存同步服务，来定期把有更改的数据批量同步到缓存里。当然这里的数据一定不是哪种实时性要求高的数据，比方说红绿码变更，近期核算检测结构等。对于实时性高的数据，例如个人信息登记和修改，一定是要同时更新存储和缓存的。
- Binlog订阅更新
- Delay Queue

### 结论

- 难以保证强一致
- 最终一致性
- 拼概率,减少脏数据的可能性

### 选择

- evict cache -> write db;binlog or ttl 双写
  - 多数公司的选择
- write db -> binlog更新
  - didi
- write db -> evict cache 也是好选择
  
## 穿透 & 雪崩 & 击穿 等

1. 穿透: 访问一个不存在的key
   - 在缓存中加入该key的null值,设置短期过期
   - 接口层增加校验，如用户鉴权，id做基础校验，id<=0的直接拦截,反正就是不符合规范的key值拦截；
   - 互斥锁，缓存失效先去获得锁，得到了锁再去请求数据库，没得到锁的则休眠一段时间重试。单机可以使用synchronized或ReentrantLock加锁,分布式环境需要加分布式锁，如Redis分布式锁setnx 配上过期时间。
   - 异步更新策略：是否key取到值，都直接返回，v值中维护一个缓存失效时间，如缓存过期异步起一个线程去读数据库，更新缓存，需要做缓存预热
2. 雪崩 大量key失效
   - 用布隆过滤器或布谷鸟过滤器解决
   - 随机ttl
   - 当我们设置缓存的时候，如果不注意缓存过期时间，如果在同一时刻大批量的缓存失效，就会有大量的访问同时进入存储。所以我们可以基于数据 sharing 分片设置不同的缓存时间。另外我们还可以有一个缓存续约服务，对于那些没有数据更新的缓存，定期批量的延长缓存时间。当然这个服务也可以基于数据 sharing 分片提高效率。
   - 如果缓存数据库是分布式部署，将热点数据均匀分布在不同得缓存数据库中。
   - 双缓存：A设置过期时间，B设置不过期，先从A读取，不存在就去读取B返回结果，异步启动一个更新线程，同时更新AB
   - 设置热点数据永远不过期
3. 击穿 大量请求未缓存的key或key过期
   - redis分段锁,同样的请求争夺一把锁,一个请求去数据库查,其余的自旋100ms,再去缓存读取,如果依然不存在尝试去数据库拿
   - 热点数据永远不过期
   - 加互斥锁，缓存中有数据，直接返回结果了
   - 缓存中无数据，第1个进入的线程，获取锁setnx(可以用热点key做锁)并从数据库去取数据，没释放锁之前，其他并行进入的线程会等待100ms，再重新去缓存取数据。这样就防止都去数据库重复取数据，重复往缓存中更新数据情况出现。
4. 缓存容量：西安常住人口大约1200万人，一个人分配10KB的缓存估算，大约就需要120GB，在加上25%的 Buffer，所以需要大约总共150GB的缓存。当然这么大的缓存不可能是单机的，一定是分布式的的，需要利用一些基于缓存数据分片的 sharding 方式把他们均匀的缓存在不同的机器上
5. 缓存预加载：我们不可以指望通过应用程先查询缓存，没有数据在去存储里取并放到缓存里，这样在并发大的时候依然会有问题。所以需要有缓存的预加载过程，当然我们可以基于数据 sharing 分片的方式去加载，例如可以基于人所属的区域，分不同的批次做，这样也提高效率。

### 无锁双缓存

- 使用系统调用进行同步的主要问题在于频繁切换上下文耗时较长，而后台系统的处理速度又是除正确性之外最为关键的指标。为提高系统的运行速度，我们可以使用用其他系统资源来换取时间的办法，从而避免使用锁之类系统调用。在这些方法中，最常见的就是用空间换取时间。
  - 针对一写多读的情况，可以使用”双 buffer“ 及共享指针机制来实现对同一变量高效访问，同时又能保证不会出现竞争条件。这一实现的技术关键点在于以下两个方面：
    - 双 buffer 的备份机制，避免了同时读写同一变量。双buffer 就是指对于通常要被多个线程访问的变量，再额外定义一个备份变量。由于是一写多读，写线程只向备份变量中写入，而所有的读线程只需要访问主变量本身即可。当写进程对备份变量的写操作完成后，会触发主变量指针和备份变量指针的互换操作，即指针切换，从而将原变量和备份变量的身份进行互换，达到数据更新的目的。
    - 共享指针 shared_ptr，由于其记录了对变量的引用次数，因而可以避免指针切换时的“访问丢失”问题。
    - 所谓双buffer技术，其实就是准备两个Obj，一个用来读，一个用来写。写完成之后，原子交换两个Obj；之后的读操作，都放在交换后的读对象上，而原来的读对象，在原有的“读操作”完成之后，又可以进行写操作了。
  - 但是，这里有两个问题：
    - "原子交换"如何做？
    - 但是，shared_ptr的读写无法做到原子操作——shared_ptr的引用计数是原子的，但是shared_ptr本身不是。
  - 这时，可以换个思路。我们将两个shared_ptr对象放到一个数组中，用一个原子的下标表示当前的读对象，此时“原子交换”，只需要原子赋值下标即可。
    - 如何判断，原来的读对象上的读取操作都结束了？
    - 获得其引用计数，来判断当前是否还有其他线程在读取这个Obj。
- 无论是双写模式还是失效模式，都会导致缓存的不一致问题。即多个实例同时更新会出事。怎么办？
  - 如果是用户纬度数据（订单数据、用户数据），这种并发几率非常小，不用考虑这个问题，缓存数据加
  - 上过期时间，每隔一段时间触发读的主动更新即可
  - 如果是菜单，商品介绍等基础数据，也可以去使用canal订阅binlog的方式。
  - 缓存数据+过期时间也足够解决大部分业务对于缓存的要求。
  - 通过加锁保证并发读写，写写的时候按顺序排好队。读读无所谓。所以适合使用读写锁。(业务不关心脏数据，允许临时脏数据可忽略)
- 总结
  - 我们能放入缓存的数据本就不应该是实时性、一致性要求超高的。所以缓存数据的时候加上过期时间，保证每天拿到当前最新数据即可。
  - 我们不应该过度设计，增加系统的复杂性
  - 遇到实时性、一致性要求高的数据，就应该查数据库，即使慢点。

## Redis发布订阅

进程中的一种消息通信模式，发送者pub发送消息,订阅者sub接收消息

## 分布式锁

lua make (compare and set)

``set + nx + ex`` 原子性只有一个能拿到锁

## redlock

- 2/n + 1
- 推荐
  - 最少5个实例
  - 3个及以上拿到锁
  - 未拿到锁的释放锁

## 事务

事务(生产环境一般都是集群,做了数据分片操作，事务涉及多个key，但key不一定都存储在同一个server上，所以事务非常鸡肋)

- Multi 输入的命令会被检查，错误直接整体失效，最后exec执行，discard放弃组队
- 悲观锁：锁
- 乐观锁：非锁，版本机制，cas
- 特性
  - 单独的隔离操作，不会被其他客户端打断
  - 没有隔离级别概念
  - 不能保证原子性：跳过错误，依旧执行，没有回滚

- MULTI
- EXEC
- DISCARD
- WATCH
- UNWATCH

## 4种缓存

JVM堆内缓存

- JVM堆内缓存因为可以避免memcache、redis等集中式缓存网络通信故障问题，目前还在项目中广泛使用。
- 第一不要全量拉取覆盖，有很大的网络开销,削峰填谷
- 第二不要把一个大对象整体替换为新对象.new 对象 会造成GC频繁。
- 一天1-2次fullgc是正常的,提前知道有大促,可通过提前触发GC等方式避免高峰期爆发fullgc.younggc至少是5分钟一次

JVM堆外缓存

- 堆外缓存的内存回收原理使用的是Java的虚引用。这个设计可以避免JVM的GC问题，

linux的buffers/cached
  
- linux系统上运行一下top 命令或者free -h | -m命令，都能够看到buffers和cached相关的数据。需要注意的是通常我们看到的监控数据 空闲内存百分比

集中式缓存

- redis缓存其实也有本机代理，可以缓存一些活跃的数据在本机上，本机可以取到不数据时不需要跨网络通信。但是因为redis本质是key-value的结构。如果需要根据通配符取数据全量，如果网络出现故障，可能会影响数据的完整性。
- 但是redis缓存最让人担心的是不规范的使用方法。比如存一个很大的value。具体这个对网络和存储造成的问题就不详细说了。可以想象下马桶堵了的情景

## redis集群

三主三从是最小的分布式集群配置

### 哨兵

配置哨兵，在主机宕机后，投票决定新的一轮主机由谁来当,使用的是RAFT算法

### 主从复制

1. 主机更新数据后根据配置，自动同步到备份机的master、slaver，master写为主，slaver读为主
2. 读写分离，性能扩展、容灾快速恢复(故障恢复)
3. 数据冗余：数据的冷热备份，持久化之外的另一个冗余方式
4. 负载均衡：读写分离，主写从读
5. 高可用基石：哨兵，集群的基石
   1. 操作：拷贝多个redis.conf文件
   2. 开启daemonize yes
   3. Pid文件名字
   4. 指定端口
   5. log文件名字
   6. Dump.rdb名字
   7. Appendonly 关掉或换名字

### 集群分布

1. 水平扩展，将整个数据分布在n个节点中
   1. 拷贝多个redis.conf文件
   2. 开启daemonize yes
   3. Pid文件名字
   4. 指定端口
   5. Log文件名字
   6. Dump.rdb名字
   7. Appendonly 关掉或者换名字
2. 配置cluster
   1. cluster-enable yes 打开集群模式
   2. cluster-config-file xxx.conf 设置生成的节点配置文件名
   3. cluster-node-timeout 15000设置节点失联时间，超多该时间（毫秒），集群自动进入主从切换
3. 集群的不足：
   - 一个key，对应的value是Hash类型的。如果Hash对象非常大，是不支持映射到不同节点的！只能映射到集群中的一个节点上！还有就是做批量操作比较麻烦！
   - Redis Cluster集群架构，不同的key会划分到不同的slot中，因此直接使用mset或者mget等操作是行不通的。
4. Redis集群模式下，如何进行批量操作？
   - 如果执行的key数量比较少，就不用mget了，就用串行get操作。如果真的需要执行的key很多，就使用Hashtag保证这些key映射到同一台Redis节点上。简单来说语法如下
5. Redis做读写分离
   - Redis Cluster的架构，是属于分片集群的架构。而Redis本身在内存上操作，不会涉及IO吞吐，即使读写分离也不会提升太多性能，Redis在生产上的主要问题是考虑容量，单机最多10-20G，key太多降低Redis性能.因此采用分片集群结构，已经能保证了我们的性能。其次，用上了读写分离后，还要考虑主从一致性，主从延迟等问题，徒增业务复杂度。

### redis集群使用时有什么注意事项

1. 防止集中失效
   - 防止缓存集中失效是对后端存储的保护。
   - 缓存穿透、缓存集中失效和缓存雪崩
2. 单线程执行，注意不要卡住
   - 单线程会阻塞,不能多线程执行任务,而且卡住常用于避免大key问题,大key既是大value,如果大value会导致网络和存储问题
3. 注意客户端和服务端的版本匹配
   - 客户端做了什么事情。我理解它就做了两件事：第一是使用RESP（Redis自定义的序列化协议）传输客户端命令并返回结果。第二是为了做第一件事，因为Redis集群是直连服务端模式，所以计算命令要落在哪个节点、哪个哈希槽上也是客户端来做的，我就称为选节点吧。
   - 升级客户端依赖的jar包变了。这个可能会引起程序启动错误
4. 分片要保持流量均匀
   - redis集群的发展史，从单机版到主从版到哨兵模式.分片模式
   - redis集群就是将一个完整服务数据分成几份，每份都带着从节点，故障时可自动转移的一个整体。
   - 分片保持流量均匀更不容易阻塞嘛。
5. 注意超时时间配置
   - 带过期时间的key又叫volatile key不稳定key,相当key这个对象有value和过期时间2个属性
   - 服务端有两种超时配置
   - 一个是惰性删除，就是访问的时候发现过期了，就直接删除了；
   - 另一个策略会定期去删除，这个是为了防止一个过期的key总是不被访问到，还占着资源不释放。
6. 当内存缓存用，推荐删除代替更新
   - 先删除缓存,再更新数据库,更新缓存
   - Cache-Aside选择这个模式
   - Read-Through/Write-Through、Write-Behind这些都是以缓存为主,不可靠

## Redis 安装和指令

- Yum install gcc  
- Gcc --version
- Make
- Make install
- Copy mv redis.conf
- Daemonize no --> yes
- Redis-server /myredis/redis.conf
- Redis-cli Shutdown exit
- Redis-cli -p 6379

</br>
</br>

- Select
- Dbsize
- Flushdb / Flushall

</br>
</br>

- NX 键不存在才执行
- XX 键存在才执行
- EX 设置键过期时间为秒
- PX  设置键过期时间是毫秒

</br>
</br>

- redis-server /myredis/redis.conf
- redis-cli -p 6379

### Redis key

- Keys * ,hgetall , smembers , sunion(遍历少用)
- Exists key
- Type key
- Rename[nx] oldk newkey
- Del key / Unlink key
- Expire key time
- Expireat key timestamp
- pexpire key milliseconds
- pexpireat key milliseconds-timestamp
- redis内部都是pexpireat
- Persist k  (删除任意键的过期时间)
- Ttl key
- Pttl key
- Move (了解) / dump + restore(非原子性) / migrate(简化，原子性，可多键)
- Keys *, hll*3 , [ ] , \x  生产环境不要用，1：不对外提供服务的节点，2：总键值对较少
- Scan 0(大数据渐进式 得到cursor,不会阻塞, hscan , sscan , zscan类似)
- Set k v
- Unlink key   reids 4.0 async del

### String

- Set k1 v1(会删除字符串类型键的过期时间)
- Set k v ex 60 nx
- Get key
- Append key  v
- Strlen key
- Incr key
- Decr key
- Incrby/decrby key int
- Mset k v k v  多用这种多值
- Mget k k k
- Msetnx k v k v
- Getrange key sta end
- Setrang key sta v
- Setnx key  v
- Setex key time v
- Getset k v (return old v)

### Hash

- Hset k f1 v1 f2 v2
- Hget k f
- Hmset k f1 v1 f2 v2
- Hmget  k1 f1 f2 f3
- Hgetall k
- Hlen k
- Hstrlen k f
- Hincrby/hincrbyfloat k f
- Hdel k f1 f2 f3
- Hexists k f
- Hkeys k
- Hvals k
- Hincrby k f int
- Ksetnx k f v

### List

- Lpush / rpush kv kv..
- Linsert k before / after old v new v
- Lrange key sta end
- Lindex k v
- Llen key
- Lpop / rpop
- Lrem key int v
- Int > < = 0
- Ltrim key start end
- Lset key index new v
- Rpoplpush
- Blpop/Brpop key [key…] timeout 阻塞
- Lpush + blpop
- lpush+lpop=Stack(栈)
- lpush+rpop=Queue(队列)
- lpush+ltrim=Capped Collection(有限集合)
- lpush+brpop=Message Queue(消息队列)

### Set

- Sadd k v1 v2 v3
- Smembers k
- Sismember k v
- Scard k
- Sscan k cursor
- Srem k v1 v2
- Spop k
- Srandmember k n
- Smove k1 k2 v1
- Sinter k1 k2
- Sinterstore k3 k1 k2
- Sunion k1 k2
- Sdiff k1 k2
- sadd=Tagging（标签
- spop/srandmember=Random item（生成随机数，比如抽奖）
- sadd+sinter=Social Graph(社交需求)

### Zset

- Zadd k1 score1 member1 score2 member2 …
- Zscore k member
- Zcard k
- Zcount k min max
- Zrange k start stop [withscores]
- Zrangebyscore k min max [withscores] [limit offset count]
- Zrevrangebyscore k max min [withscores] [limit offset count]
- Zincrby k increment member 返回增加后的结果 无值就是自增
- Zrem k member
- Zrank k member
- Zrevrank k member
- Zremrangebyrank k sta end
- Zremrangebyscore k min  max
- -inf / +inf
