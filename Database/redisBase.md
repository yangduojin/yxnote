# 数据结构
1. ``String``
   - SDS
2. ``List`` 队列结构
   - Ziplist
   - Quicklist
3. ``Hash`` 适合存对象
   - Ziplist
   - Dict
4. ``Set``
   - Inset
   - Dict
5. ``Sorted Set``
   - Hash + SkipTable
- Stream流
  - RadixTree
- bitmap 位图
- pubsub
- geo 地理位置
- hyperloglog 基数统计

## SDS 简单动态字符串
空间换时间

## Ziplist压缩列表
- 连续内存
- 特殊编码
- 节省内存
- 时间换空间

## linkedList(淘汰)

## intset整数集合
- 当value是数字
- 当size没有超过阈值
  - 数组项从小到大排序
  - 二分查找
- 时间换空间

## dict
![](/Database/img/redisDict.png)
- 渐进式rehash
- murmurhash

## quicklist
![](/Database/img/redisQuicklist.png)
- 链表和ziplist的组合

## skiptable 跳跃表
![](/Database/img/redisSkiptable.png)

# 持久化
- RDB
  - 会丢失一部分最新数据
  - 每个redis实例只会存一份rdb文件
  - 可以通过save,bgsave来调用
  - 二进制文件,lzf
- AOF
  - 类似 binlog 机制,可以做到不丢数据
  - 每次数据操作调用 flushAppendOnlyFile 文件来刷新aof
  - 每次操作都需要 fsync ,前台线程阻塞
    - always
    - every sec

- 混合模式 RDB+AOF
- 加载顺序
  - 先AOF
  - 后RDB

# 删除策略
- 惰性删除
- 定时删除
- 定期删除策略是前两种策略的折中：100s  
  - 定期删除策略每隔一段时间执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。
  - 周期性轮询Redis库中的时效性数据，来用随机抽取的策略，利用过期数据占比的方式控制删除频度
  - 特点1：CPU性能占用设置有峰值，检测频度可自定义设置
  - 特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理
  - 总结：周期性抽查存储空间（随机抽查，重点抽查）

三种策略都有问题,需要一个兜底方案,所以**缓存淘汰**登场了

## 缓存淘汰
- ``volatile-lru(default)`` 从设置过期数据集里查找最近最少使用
- ``volatile-lfu`` 对所有设置了过期时间的key使用LFU算法进行删除
- ``volatile-ttl`` 从设置过期数据集里清理已经过期的key
- ``volatile-random`` 从设置过期数据集里任意选择数据淘汰
- ``allkeys-lfu`` 对所有key使用LFU算法进行删除 Least frequently used
- ``allkeys-lru`` 从数据集中挑选最近最少使用的数据淘汰 Least frequently used
- ``allkeys-random`` 从数据集中任意选择数据淘汰
- ``no-enviction`` 不清理

#### 如何配置，修改
命令  
- config set maxmemory-policy noeviction
- config get maxmemory
配置文件 - 配置文件redis.conf的maxmemory-policy参数


# 缓存一致性
1. ``write cache -> write db`` 写数据库失败时
2. ``write db -> write cache`` 更新cache失败,并发出现脏数据
3. ``evict cache -> write db`` 延迟,并发出现脏数据
4. ``write db -> evict cache`` 并发出现脏数据
5. ``evict cache -> write db -> evict cache`` 减少脏数据概率
## 避免脏数据
- ttl
- 定时更新
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
  
# 穿透 & 雪崩
- 穿透: 访问一个不存在的key
  - 在缓存中加入该key的null值,设置短期过期
- 雪崩 大量key失效
  - 随机ttl
## 击穿
- 击穿 大量请求未缓存的key
  - redis分段锁,同样的请求争夺一把锁
  - 一个去数据库查,其余的自旋100ms
  - 再去缓存读取,如果依然不存在尝试去数据库拿

# 分布式锁
lua make (compare and set)

``set + nx + ex`` 原子性只有一个能拿到锁

# redlock
- 2/n + 1
- 推荐
  - 最少5个实例
  - 3个及以上拿到锁
  - 未拿到锁的释放锁

# 事务
- MULTI
- EXEC
- DISCARD
- WATCH
- UNWATCH
