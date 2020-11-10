---
title: RabbitMQ的使用
date: 2020-09-15 14:45:18
tags:
  - 消息队列
  - 中间件
  - RabbitMQ
categories:
  - 中间件
  - 消息队列
---

# 基本概念

## 模型架构概念

- 生产者
- 消息中间件的服务节点(Broker)

  - 交换器
  - 队列
  - 消息

- 消费者

## 连接

- 连接（connection）

  - TCP连接
  - 生产者、消费者分别与Broker建立连接

- 信道（channel）

  - AMQP 信道
  - connection上的虚拟连接，实体是TCP连接
  - 每个信道唯一ID

## AMQP

- 高级消息队列协议

  - 应用层协议

- 协议层次

  - Module Layer

    - 提供客户端调用命令

  - Session Layer

    - 客户端与服务器之间的通信，传输客户端命令等

  - Transport Layer

    - 传输二进制数据流

- rabbitMQ是AMQP协议的一种实现

  - rabbitMQ客户端方法都对应了Module Layer中的AMQP命令
  - 对高层来说，AMQP是通过协议命令进行交互的，可以看作一系列结构化命令的集合

# RabbitMQ服务器常用指令

## 打开可视化界面

- rabbitmq-plugins enable rabbitmq_management
- http://127.0.0.1:15672/
- 默认用户名密码guest

## 启动关闭

- 以应用方式启动

  - rabbitmq-server -detached 后台启动
  - Rabbitmq-server 直接启动，如果你关闭窗口或者需要在改窗口使用其他命令时应用就会停止
  - 关闭:rabbitmqctl stop

- 以服务方式启动（安装完之后在任务管理器中服务一栏能看到RabbtiMq）

  - rabbitmq-service install 安装服务
  - rabbitmq-service start 开始服务
  - Rabbitmq-service stop  停止服务
  - Rabbitmq-service enable 使服务有效
  - Rabbitmq-service disable 使服务无效

# 生产者/消费者常用操作

1. 生存者和消费者与RabbitMQ服务器建立连接
2. 生产者/消费者创建交换器和队列
3. 生产者生产数据
4. 消费者消费数据

## 建立连接

- 生成工厂

  `ConnectionFactory factory = new ConnectionFactory()`

- 设置用户名密码

  `factory.setUsername("A")`
  `factory.setPassword("123")`

- 设置虚拟主机（类似命名空间或权限，不同虚拟主机broker、队列等不共享）

  `facotry.setVirtualHost(virtualHost)`

- 设置Borker IP和端口号

  `factory.setHost(IP)`
  `factory.setPort(PORT)`

- 生产(建立)连接

  `Connection conn = factory.newConnection()`

## 创建交换器与队列

### 声明交换器 exchangDeclare

`Exchange.DeclareOk exchangeDeclare(交换器名 , 交换器类型 , 是否持久化=true ,
是否自动删除=true , 是否内置=true ,Map<String, Object> 其他参数) throws IOException ;`

- 持久化
  保存到硬盘
- 自动删除
  有队列或者交换器绑定了本交换器，然后所有队列或交换器都与本交换器解除绑定，此交换器就会被自动删除
- 内置
  客户端程序无法直接发送消息到这个交换器中，只能通过交换器路由发送到交换器

### 声明队列 queueDeclare

`Queue.DeclareOk queueDeclare (队列名 ,  是否持久化=true , 是否排他=true , 是否自动删除=true , Map<String, Object> 其他参数) throws IOException;`

- 持久化：保存到硬盘
- 排他：针对连接可见，只要是当前connection下的信道都可以访问
  一旦该队列被声明，其他连接无法声明相同名称的排他队列。
  队列即使显示声明为durable，连接断开时（注意不是信道断开）也会被自动删除
  这种队列适用于一个客户端同时发送和读取消息的应用场景
-  自动删除
  至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除

> 不带任何参数的queueDeclare 方法默认创建一个由RabbitMQ 命名的(类似这种amq.gen-LhQzlgv3GhDOv8PIDabOXA 名称，这种队列也称之为匿名队列〉、排他的、自动删除的、非持久化的队列
>

### 绑定队列 queueBind

`Queue.BindOk queueBind(队列名 , 交换器名 , 绑定键, Map<String, Object> 其他参数) throws IOException;`

### 解绑队列 queueUnbind

`Queue. UnbindOk queueUnbind(队列名 , 交换器名 , 绑定键, Map<String, Object> 其他参数)  throws IOException;`

### 绑定交换器 exchangBind

`Exchange . BindOk exchangeBind(目标交换器 , 源交换器, 绑定键, Map<String , Object> 其他参数) throws IOException `

## 生产消息

basicPublish
`void basicPublish(交换器名, 路由键, mandatory, immediate , BasicProperties 消息的基本属性集 , byte[] 消息体) throws IOException ;`
	
**消息的14个基本属性集**

- contentType 
- content Encoding 
- headers ( Map<String ， Object>)
- deliveryMode 
- priority 
- correlationld 
- replyTo
- expiration
- messageld
- immediate
- type
- userld
- appld
- clusterId

属性集构造方法

`new AMQP.BasicPropertieS.Builder().headers(headers).contentType( " text/plain" ).build()`

- mandatory：不可达时是否返回生产者
- immediate：队列不存在消费之时是否放入队列

## 消费消息

### 接收

**推**

接收消息一般通过实现Consumer 接口或者继承DefaultConsumer 类来实现。
`String basicConsume(队列名 , 是否自动确认autoAck, 消费者标签同channel唯一,boolean noLocal , 是否排他, Map<String ， Object> 其他参数 , Consumer callback) throws IOException ;`

- autoAck：为true则RabbitMQ 会等待消费者显式地回复确认信号后才从内存(或者磁盘)中移去消息(实质上是先打上删除标记，之后再删除)

- nolocal：不能将同一个Connectio口中生产者发送的消息传送给这个Connection 中的消费者:

- Consumer callback：设置消费者的回调函数。用来处理Rabb itMQ 推送过来的消息，比如DefaultConsumer ， 使用时需要客户端重写(override) 其中的方法,可重写方法很多，重写handleDelivery方便

```java
new DefaultConsumer(channel) {
	@Override
	public void handleDelivery(){}
	}
```

> 订阅某一个队列中的消息,channel会自动在处理完上一条消息之后，接收下一条消息。（同一个channel消息处理是串行的）。除非关闭channel或者取消订阅，否则客户端将会一直接收队列的消息。

**拉**

`GetResponse basicGet(队列名 , 是否自动确认) throws IOException;`

- basicGet不阻塞，如果没消息不判断就用会抛出空指针异常。要加上message==null的判空操作
- basic.get RabbitMQ在实际执行的时候，是首先consume某一个队列，然后检索第一条消息，然后再取消订阅。

**区别**

- consume是只要队列里面还有消息就一直取。
- get是只取了队列里面的第一条消息。
- 因为get开销大，如果需要从一个队列取消息的话，首选consume方式，慎用循环get方式。

### 拒绝

**拒绝一条**

`void basicReject(long 消息的编号, boolean requeue) throws IOException;`

- requeue：为true则会重新存回队列中

**批量拒绝**

`void basicNack(long deliveryTag, boolean multiple , boolean requeue) throws IOException;`

- multiple：为true 则表示拒绝deliveryTag 编号之前所有未被当前消费者确认的消息；为false，则等于basicReject

## 关闭连接

- channel.close();
- conn.close() ;

# 交换器类型

## fanout

交换器广播（与两键无关）
交换器将收到的消息全部发送给所有与之绑定的队列

## direct

绑定键=路由键
将消息发送到路由键能与队列绑定键匹配的队列中

## topic

模糊匹配绑定键与路由键

- direct模式的扩展，两键使用word1.word2.word3模式
- 路由键中加入通配符*和#，*用于匹配一个word，#用于匹配多个。如com.*可匹配绑定键位com.ABC的队列

## headers

- 根据消息内容中的headers属性与绑定键匹配
- 性能差，不常见

## system

## 自定义

# 注意事项

- 一个connection可创建多个channel，多线程共享channel实例是非线程安全的不推荐使用channel和connection中的isOpen()
- 推荐捕获异常com.rabbitmq.client.ShutdownSignalException，该异常会在关闭状态抛出试着捕获IOExceptio口或者SocketException ，防止Connection 意外关闭。
- 生产者和消费者都能够使用queueDeclare 来声明一个队列，但是如果消费者在同一个信道上订阅了另一个队列，就无法再声明队列了。必须先取消订阅，然后将信道置为"传输"

