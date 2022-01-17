# RabbitMQ

- [RabbitMQ](#rabbitmq)
  - [为什么使用MQ](#为什么使用mq)
    - [流量削峰](#流量削峰)
    - [异步处理](#异步处理)
    - [应用解耦](#应用解耦)
  - [MQ的分类](#mq的分类)
    - [内部原理](#内部原理)
    - [JMS支持](#jms支持)
    - [核心功能](#核心功能)
    - [rabbitMQ](#rabbitmq-1)
    - [TTL](#ttl)
    - [延迟队列加载](#延迟队列加载)
    - [轮询扫描](#轮询扫描)
    - [Dlx (死信队列，兜底队列，本身也有routekey)](#dlx-死信队列兜底队列本身也有routekey)
    - [核心组件](#核心组件)
    - [持久化](#持久化)
    - [消息的应答模式](#消息的应答模式)
    - [可靠投递机制](#可靠投递机制)
    - [幂等性](#幂等性)
  - [RabbitMQ](#rabbitmq-2)
  - [备份交换机](#备份交换机)

## 为什么使用MQ

### 流量削峰

排队能保证有条不紊，代价是整体处理速度会慢些。秒杀的时候

### 异步处理

三种方式来异步处理。

A调用B，B返回A说收到调用请求了。同步请求已经完成，但B的执行才刚开始。这时候，

第一种方式是A每隔一段时间来查询一次，看B是否执行完，这是拉的方式；

第二种方式是A提供一个回调地址，B执行完之后回调A，这是推的方式；

第三种就是使用MQ，A使用MQ给B发消息，B处理完再回一个消息，好处是上面提到的同时可以流量削峰。

### 应用解耦

MQ实现了逻辑解耦+物理解耦。逻辑上，将请求和结果处理分开了；物理上，系统只用与MQ通信。异步处理的三种方式的前两种，现在也多很常见。那是因为MQ是有代价的，那就是需要一套MQ设施。做开放平台，用户之间的唯一设施就是互联网，这时候更依赖双方的协议约定，所以前两种异步处理方式不会被MQ取代。

## MQ的分类

Kafka大数据的杀手锏，以百万级TPS吞吐量名声大噪。时效是ms级别，分布式的可用性高。消费者采用拉的方式获取消息，消息有序，通过控制可以保证消息仅被消费一次。但是单机超过64个分区，load会明显飙高；实时性取决于轮询时间间隔，关键是有可能丢消息，不适合订单业务中使用。

RocketMQ是国货，用Java语言实现，在设计时参考了Kafka，单机吞吐量达到十万级别，分布式架构可用性高，消息可以0丢失，扩展性高。但是支持的客户端成熟的也就是Java，核心代码没有实现JMS，迁移需要修改大量代码。

RabbitMQ是erlang开发的，吞吐量达到万级别，稳定、健壮、跨平台，支持多种语言，企业间通信中常用。

### 内部原理

Kafka怎么保证消息能且仅能收到一次？这是个埋坑题，是与面试官斗智斗勇的开始。什么幂等、事务、流式EOS呀，其实呢，Kafka本身是不保证仅且仅收到一次的，所以这些实现方法都不优雅。

RabbitMQ通过AMQP事务机制，还有上面已经提过的ack也就是confirm两种可选方式保证消息被收到。

### JMS支持

RabbitMQ不支持JMS协议。这个很好理解。因为JMS是Java消息服务，提供了消息传递的Java标准API。

而RabbitMQ是Erlang写的，对Java的支持会弱一些。但是RabiitMQ实现了AMQP标准协议。AMQP只是统一了数据交换的标准格式，与语言无关。

### 核心功能

RabbitMQ的核心实际上就是AMQP的核心：MessageQueue、Exchange和Binding。

MessageQueue就是消息队列，一个队列里的一条消息，也就是同一个message ID对应的消息，不管有多少个消费者来分摊压力，也只能被消费一次。消息队列和消费者之间有ack机制，消息一旦确认安全送达，RabbitMQ服务端就可以安全删除消息了。

Binding是MessageQueue与Exchange之间的连接，Exchange只能给Binding的MessageQueue发送消息。

Exchange有四种类型：fanout(广播),topic(主题,路径),direct(bindingkey),header.后两个是点对点,最后一个性能差.前两个是发布订阅
本质上就是有一堆MessageQueue，一个消息是要被复制几份，发到哪几个Binding的消息队列去。Exchange给定了规则：fanout是对每个消息队列复制一份发送；direct意思是只发指定的一份，不复制；topic是发送通配符匹配的几份；header可以指定一些其他的过滤条件发送。消息从生产者发送到exchange之后也有ack机制来保证消息的可靠传输。

Kafka只有topic的概念。这是因为Kafka的设计上消息只用存一份，通过游标，发送后不立即删除消息。多个消费者组可以互不影响的消费。这是Kafka的一大改进。

消息百分百投递成功

1. 一张表存放消息信息(业务数据)，另一张表存放消息记录表
2. 发送消息到MQ broker节点(confim发送，返回异步结果)
3. 生产者接受broker的confim结果，正常就更新消息记录表的该消息状态status = 1；
4. 如果网络等其他问题导致broker或producer未收到消息，producer重复投递会造成消息重复，需要消费端做幂等处理(乐观锁?),需要一个定时任务，如每5分钟拉取处于中间状态的消息，消息可以设置一个超时时间，超过1分钟 Status = 0，就将该消息拉取出来
5. 把中间状态的消息重新投递retry send，继续发送到MQ，
6. 可以设置最大尝试次数，如3次之后还是失败就 status = 2；转人工处理或把消息转入失败表中

### rabbitMQ

基于AMQP协议的rpc远程调用，将微服务各模块的通信依赖统一改为依赖该中间件

消息目的地的主要是两种: queue(队列) 和 topic(发布/订阅) 两种

五种模式:基本模式，工作模式，发布/订阅模式，路由模式，通配符模式

Ttl 整个队列消息存活时间，也可以设置单独的消息存活时间(很少用)

### TTL

消息ttl 每个消息具有单独的ttl,到期也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间,不设置ttl就永不过期,设置为0即立刻投递消费者成功,不然马上丢弃

队列ttl 消息进入队列开始,领取一个队列规定的ttl,到期会被队列立刻丢弃

### 延迟队列加载

消息设置ttl,到达死信,消费者一直消费死信队列就是一种延迟加载的机制(订单付款),但是队列机制长时间的ttl会堵住后面的短时间消息ttl,需要一个插件,只要消息 ttl 到期,就会立即被丢到死信.之前是在交换机里面

### 轮询扫描

quartz , spring task

长期没有付款的订单，要定期关闭掉。

如果时限比较小，比如30分钟未付款的订单就关闭（一般是锁了库存的订单），也可以用延时队列解决。

如果时限比较长比如1-2天，可以选择用轮询扫描。

### Dlx (死信队列，兜底队列，本身也有routekey)

1. 队列消息长度达到限制
2. 消费者拒绝消费消息(basicNack/basicReject)并requeue为false
3. 原队列存在消息过期，过期消息转入dlx

Dlx 跟普通的队列一样，也需要交换机和queue，只是一个兜底队列，别的queue在配置的时候设置消息失败之后如何转入dlx交换机和队列

队列，所以遵循FIFO先进先出原则。因为存放的是消息，所以是一种跨进程的通信机制。

### 核心组件

AMPQ消息路由必要三部分：交换器、队列、绑定。

Java核心组件：ConnectionFactory、Connection、Channel、Delivery、DeliverCallback、CancelCallback

### 持久化

消息持久化
队列持久化

### 消息的应答模式

![MQReturnConfirm](/spring%20boot&cloud/img/MQReturnConfirm.png)

自动签收 消息可能丢失(出现故障会被直接丢掉消息)    MessageListener 自动签收 需要被消费者实现;

手动签收 消息不会丢失,而且可以批量签收  ChannelAwareMessageListener 手动签收 需要被消费者实现

### 可靠投递机制

从producer和consumer 两方面确认消息的可靠投递

所有消息的发布确认机制,成功传递到交换机,投递到队列 都有各自的确认回调;

``rabbitTemplate.setMandatory(true);``回退机制开启,不然不能投递队列的消息会被直接丢弃

``confim``确认机制: 需要实现接口;消息被交换机收到

``return``后退机制: 需要实现接口;消息没有被投递,并且没有备份交换机处理该消息,就会回退
``producer``到``exchange``有``confirmCallback``

``exchange``到``queue``有``returnCallback``

``spring.rabbitmq.publisher-confirms=true``

在创建 ``connectionFactory`` 的时候设置 ``PublisherConfirms(true)`` 选项，开启
``confirmcallback`` 。

``CorrelationData``：用来表示当前消息唯一性。
消息只要被 ``broker`` 接收到就会执行 ``confirmCallback``，如果是 ``cluster``模式，需要所有
``broker`` 接收到才会调用 ``confirmCallback``。
被 ``broker`` 接收到只能表示 ``message`` 已经到达服务器，并不能保证消息一定会被投递 到目标 ``queue`` 里。所以需要用到接下来的 ``returnCallback``。
``spring.rabbitmq.publisher-returns=true``
``spring.rabbitmq.template.mandatory=true``
``confrim`` 模式只能保证消息到达 ``broker`` ，不能保证消息准确投递到目标 ``queue`` 里。在有 些业务场景下，我们需要保证消息一定要投递到目标 ``queue`` 里，此时就需要用到
``return`` 退回模式。
这样如果未能投递到目标 ``queue`` 里将调用 ``returnCallback`` ，可以记录下详细到投递数 据，定期的巡检或者自动纠错都需要这些数据。

确认机制参数: ``spring.rabbitmq.publisher-confirm-type=correlated``

1. ``NONE``    禁用发布确认模式，是默认值

2. ``CORRELATED``  发布消息成功到交换器后会触发回调方法

3. ``SIMPLE`` 经测试有两种效果，

其一效果和CORRELATED值一样会触发回调方法，

其二在发布消息成功后使用``rabbitTemplate`` 调用 ``waitForConfirms`` ``或waitForConfirmsOrDie``; 等待 ``broker`` 节点返回发送结果，根据返回结果来判定下一步的逻辑; ``waitForConfirmsOrDie`` 方法如果返回 `false` 则会关闭 `channel` ，则接下来无法发送消息到 ``broker``

### 幂等性

意外导致ack丢失,消费者幂等性一般用全局id来解决

高并发生产端可能重复生产消息,消费端就要幂等性检查,幂等性用redis的原子性setnx解决

消息ID作为key执行 setnx 命令，如果执行成功就表示没有处理过这条消息，可以进行消费了，执行失败表示消息已经被消费了。

原子性的setnx 或 lua脚本读取删除  保证订单失效最后一刻付款不会出现付款却订单失效的问题

1. ``Message message``: 原生消息类型 详细信息
T<发送消息的类型> OrderEntity orderEntity  [Spring自动帮我们转换] 不用我们去转换message的类型和请求体

2. ``Channel channel``: 当前传输数据的通道
3. ``queue``: 可以很多人来监听,只要收到消息,队列删除消息,而且只能有一个收到此消息
场景: 订单服务启动多个: 同一个消息只能一个客户端收到
只有一个消息完全处理完,方法运行结束后,我们就可以接收到下一个消息

``@RabbitListener``： 只能标注在类、方法上配合

``@RabbitHandler`` : 前一个注解指定监听什么队列,通过方法参数直接拿到消息内容而不用拿到包装的message,再转换成需要的类型

``@RabbitHandler``: 只能标注在方法上[重载区分不同的消息]

```java
@RabbitHandler
public void receiveMessageA(Message message, OrderEntity orderEntity, Channel channel)
```

@EnableJms @EnableRabbit / @JmsListener

@RabbitListener / JmsAutoConfiguration RabbitAutoConfiguration

## RabbitMQ

![rabbitmqBase](/spring%20boot&cloud/img/rabbitmqBase.png)

``Broker``

Broker简单理解就是RabbitMQ服务器(代理)，图中灰色的整个部分。后面说Broker说的就是RabbitMQ服务器。

``vHost`` 虚拟主机

每一个RabbitMQ服务器可以开设多个虚拟主机vhost（图中粉色的部分），或者说每一个Broker里可以开设多个vhost，每一个vhost本质上是一个mini版的RabbitMQ服务器，拥有自己的 "交换机exchange、绑定Binding、队列Queue"，更重要的是每一个vhost拥有独立的权限机制，这样就能安全地使用一个RabbitMQ服务器来服务多个应用程序，其中每个vhost服务一个应用程序。

每一个RabbitMQ服务器都有一个默认的虚拟主机 "/"，客户端连接RabbitMQ服务时须指定vHost，如果不指定默认连接的就是"/"。

``Exchange`` 交换机

交换机的作用就是根据路由规则，将消息转发到对应的队列上。Direct bindingkey , fanout 广播模式 ,topic 路径匹配 , headers 交换机模式.有一个默认交换机,名字是空字符串

``Connection``

我们知道无论是生产者还是消费者，都需要和 Broker 建立连接，这个连接就是Connection（看图），是一条 TCP 连接 ，一个生产者或一个消费者与 Broker 之间只有一个Connection，即只有一条TCP连接,而且是长连接。

``ConnectionFactory``

Connection工厂，负责创建和管理Connection的。

``Channel``

信道是建立在真实的TCP连接内的虚拟连接（图中白色的channel）。AMQP的命令都是通过信道发送出去的，每条信道都会被指派一个唯一ID，不论是发布消息、订阅队列还是接收消息都是通过信道完成的。一个TCP连接下包含多个信道，实现共用TCP、减少TCP创建和销毁的开销。

``Routing key``

Routing key是消息头的属性，生产者将消息发送到交换机时，会在消息头上携带一个 key，这个 key就是routing key，来指定这个消息的路由规则,交换机去匹配对应的路由键。topic Routingkey 和 bindkey一样的话会直接投递,#可有可无,*必须有一个``Binding``绑定，可以理解成一个动词，它的作用就是把exchange和queue按照路由规则绑定起来。

``Binding key``

在绑定Exchange与Queue时，一般会指定一个binding key，生产者将消息发送给Exchange时，消息头上会携带一个routing key，当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。

``Message``

消息是不具名的,由消息头和消息体组成,消息体不透明,消息头由一系列可选属性组成,包括routing-key(路由键),priority(相对于其他消息的优先权),delivery-mode(指出该消息可能需要持久性存储)

## 备份交换机

![BackupExchange](/spring%20boot&cloud/img/BackupExchange.png)
