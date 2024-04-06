---
date: 2019-4-2
title: RabbitMQ基础
tags: [RabbitMQ]
---

### 为什么使用队列，为什么使用RabbitMQ队列
消息队列用于系统之间解耦，通过高性能消息中间件，提升系统吞吐量，降低导致系统耦合。
当前有各种消息队列，RabbitMQ、Kafka、ActiveMQ等，为什么使用RabbitMQ？ 
消息队列从使用场景来分为两类：
- 一类是大数据的数据流处理，数据采集者作为生产者数据通过消息队列从到后端处理。这种场景要求高吞吐高并发，Kafka专门为这种场景设计。
- 另一类是消息高可靠低时延在系统之间传递，这种场景要求可靠性高，消息不能丢；要求时延低。AMQP协议专门为这种场景设计，RabbitMQ是AMQP协议实现者之一，也是当前使用的最广的AMQP消息队列。

### 典型的消息队列处理流程
以一个提供网页转PDF的Web应用为例，描述消息处理流程。

![](https://www.cloudamqp.com/img/blog/rabbitmq-beginners-updated.png)

1. 用户发送网页转PDF请求到Web应用
2. Web应用发送一个消息到RabbitMQ，包括网页内容、用户名、邮箱等
3. RabbitMQ的Exchange接收到消息后，将消息路由到某一个队列
4. PEF生成器消费队列的消息，生成PDF

### 基本概念

![](https://upload-images.jianshu.io/upload_images/5826441-af147cca2925ff2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

- 生产者
- 消费者
- 服务器 Broker：Broker是一个物理上的服务器（或虚拟机），它是部署了消息中间件并接收处理客户端请求的实体。
- 队列 Queue：用于存储消息的一块缓存
- 消息 Message
- 连接 Connection：应用与RabbitMQ Broker之间的TCP连接
- 通道 Channel：连接内的虚拟连接，应用生产或消费队列中的消息时，都是通过Channel完成的
- 绑定 Binding：Queue与Exchange之间连接
- 路由键 Routing Key：Exchange根据Routing Key来决定将消息投递到哪个队列，类似消息的地址
- 用户 User：RabbitMQ连接需要用户密码认证；同时可以给用户设定读、写等权限
- 虚拟主机 Virtual Host：Virtual Host用来区使用同一个RabbitMQ实例的多个应用。connections, exchanges, queues, bindings, user permissions, policies and some other things 都属于Virtual Host。

### Exchange 类别

- Direct Exchange

![](https://www.cloudamqp.com/img/blog/direct-exchange.png)
Direct Exchange 根据 Routing Key 投递消息，Routing Key 由生产者在生产消息时候设置到消息头内。 Direct Exchange投递规则是 **精确匹配 Binding Key 与 Routing Key** 。

- Default Exchange

Default Exchange 是一个系统预定义的、Routing Key 为空字符串 "" 的 Direct Exchange 。每个 Queue 都自动绑定到 Default Exchange ，其 Routing Key 就是 队列名称。

- Topic Exchange

![](https://www.cloudamqp.com/img/blog/topic-exchange.png)
Topic Exchange 根据通配符匹配规则投递消息：匹配 Routing Key 与 Routing Pattern ， Routing Pattern 在 binding 时候指定。

- Fanout Exchange

![](https://www.cloudamqp.com/img/blog/fanout-exchange.png)
Fanout Exchange 将消息投递到所有队列。

- Headers Exchange

![](https://www.cloudamqp.com/img/blog/headers-exchange.png)
Headers Exchange 根据头部选项投递消息，通过 "x-match" 定义匹配动作: any, all

- Dead Letter Exchange

如果消息找不到投递队列，则会被丢弃；RabbitMQ提供了 Dead Letter Exchange ，用于记录无法投递的消息。

### Reference
[RabbitMQ for beginners - What is RabbitMQ?](https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html)
[RabbitMQ Exchanges, routing keys and bindings](https://www.cloudamqp.com/blog/2015-09-03-part4-rabbitmq-for-beginners-exchanges-routing-keys-bindings.html)
[RabbitMQ（一）AMQP简介](https://www.jianshu.com/p/66bb8445ea6e)

