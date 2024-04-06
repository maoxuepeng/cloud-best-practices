---
date: 2019-4-9
title: Kafka最佳实践
tags: [Kafka, 最佳实践]
---

Kafka作为流式消息中间件，设计之初就是用于解决大数据场景下海量数据采集、传输问题，传输过程还需要保证时延与可靠性。Kafka已经成为大数据场景下的标配部件。本文介绍Kafka基本概念与最佳实践。

## 架构与概念

![](https://uproject-octopus-picture-hub.obs.cn-north-1.myhwclouds.com/picture-hub/mao/apache-kafka-best-practices-3-638.jpg)

- Message 消息：Kafka队列中的数据单元，消息由Key与Value、以及可选的Header组成。
- Producer 生产者：生产者往Kafka Topic中发布消息；生产者随机（round-robin）的将消息写到Topic分区中、或者使用特定的分区算法（根据消息的Key作为输入）。
- Broker ：Kafka是一个分布式集群系统，集群中每个节点称为 Broker 。
- Topic ：Topic 是消息发布的一个类别，生产者往特定Topic投递消息，消费者在特定的Topic上消费消息。
- Topic partition: 每个 Topic 都被分成若干个分区，一个分区有一个或多个副本数；一个Topic的多个副本，由一个leader以及多个follower组成，leader与follower都与Broker绑定，一个Broker可能同时是leader与follower。leader负责所有的读写请求。
- Offset: 分区内的每条消息都有一个唯一的偏移量，偏移量是一个自增的整数。
- Consumer group: 消费组是一个逻辑的消费者集合，Topic 分区以均衡方式分配给消费组内的消费者。在一个消费组内，消息以负载均衡方式分配给消费者，一条消息只能分配给一个消费者；如果一个消费者下线了，分配会被分配给另外一个消费者，这个过程称为 **rebalance** 。如果消费组内消费者比 Topic 分区要多，那么有一些消费者是空闲的；如果消费组消费者比分区数要少，那么一个消费者会从多个分区消费消息。Consumer group 用于实现多播，一个Topic分区的消息同时广播到多个Consumer group。
- Lag: 当消息消费的速率比消息生产的速率要慢的时候，消息处于消费滞后状态；消息滞后追赶时间通过公式计算： ```time = messages / (consume rate per second - produce rate per second)```

## Topic与Topic分区最佳实践
分区 Partition 是Kafka设计用于水平扩展的特性，理论上分区数量越多Kafka的吞吐量越大；当然分区数量越多相应的系统资源消耗会越大，失败重选举的耗时约长，因此并不是盲目设置一个超大分区。分区的数量是可以动态增加的（只能加不能减），因此在选择分区数量时候满足当前需求即可，随着系统吞吐量增大相应增加分区数量。

### Topic 分区数量与Broker数量保持倍数关系
Kafka默认的分区策略是取消息key的Hash值并与分区数量取模，Topic分区数量与Broker保持倍数关系可以大概率维持Broker之间的均衡。

### 确定分区数据写入速率用于估计预留空间
分区数据写入速率用于评估Kafka预留磁盘空间大小，数据写入速率bytes/s = 消息数量/s * 消息大小 。如果不清楚数据写入速率则无法评估预留磁盘空间大小。数据写入速率同时也确定了单个消费者的消费能力，达到此消费能力才能避免消费滞后 (lagging) 。

### 往 Topic 数据写入时候优先使用随机分区策略
如果指定分区写入的话容易导致数据在Topic分区之间分布不均衡，数据分布不均衡会引起一些列问题。

- “热”分区上的消费者会一直处于高负荷状态，容易造成部分Broker性能问题，如网络拥塞、CPU负载过高。
- Topic 的预留空间是按照Topic内最大分区预留的，数据分布不均衡的话会导致磁盘利用率不高。

Kafka默认的分区策略，如果消息Key是空则随机分区，如果消息Key不为空则取Key的Hash值并与分区数量取模映射到对应分区。通过设置 **partitioner.class** 的值指定自定义的分区策略。

### Topic 的副本数不要超过Broker数量

min.insync.replicas 控制Topic内消息副本数，默认值是1，在高可靠场景下，建议设置为2。由于同一个分区的leader与follower不会在同一个Broker，因此副本数量不能超过Broker数量。

## 生产消息最佳实践

### 显示设置acks保证消息生产可靠性

- acks=0: 不管消息生产是否成功，直接返回给生产者。可靠性最大，时延最低。此时生产者设置了 retries 不生效
- acks=1: leader 确认收到消息，follower 还未全部同步。可靠性适中，时延适中。
- acks=-1, acks=all: leader 与所有 follower 都已经接收到消息。可靠性最高，时延最大。

### 设置生产消息重试次数

重试参数 retries 默认值是3。如果设置了retries而没有将max.in.flight.request.per.connection设置为1, 在两个batch发送到同一个partition时有可能打乱消息的发送顺序(第一个发送失败, 而第二个发送成功)。

### 设置生产者批量发送消息数量以提高生产性能

producer会尝试批量发送属于同一个partition的消息以减少请求的数量。这样可以提升客户端和服务端的性能。默认大小是16348 byte (16k)。 
发送到broker的请求可以包含多个batch, 每个batch的数据属于同一个partition。
太小的batch会降低吞吐. 太大会浪费内存。

通过 batch.size 配置批次大小，注意调整了 batch.size 之后对应的 buffer.memory 需要确认是否足够。

## 消费消息最佳实践

### 优先使用显示提交消费确认
Kafka默认是自动提交消费确认，通过 auto.commit.enable 配置为 false 改为显示提交消费确认。自动提交消费确认，如果消费者取到了消息，但是并未消费成功，此时消息就丢失了。

## Broker 配置

### 禁止不在ISR列表中的follower选举为leader，避免出现数据不一致

unclean.leader.election.enable 参数控制是否允许不在ISR列表中的follower选举为leader。在Kafka 0.11.0.0版本之前默认值是 true，Kafka 0.11.0.0版本之后默认值是 false。
如果允许不在ISR列表内的follower选举为leader，可以提升Kafka的可用性，但是会导致数据不一致。因为不在ISR内的follower已经落后了很多个消息数，这时候如果选举为leader且之前ISR的follower恢复之后，会强制截断日志导致数据丢失。详细[参考这篇文章](https://blog.csdn.net/u013256816/article/details/80790185)。

## Reference
[Apache Kafka Best Practices](https://www.slideshare.net/HadoopSummit/apache-kafka-best-practices)
[Kafka Best Practices 1](https://community.hortonworks.com/articles/80813/kafka-best-practices-1.html)
[Kafka Best Practices 1](https://blog.newrelic.com/engineering/kafka-best-practices/)
[Effective Strategies for Kafka Topic Partitioning](https://blog.newrelic.com/engineering/effective-strategies-kafka-topic-partitioning/)
[Kafka水位(high watermark)与leader epoch的讨论](https://www.cnblogs.com/huxi2b/p/7453543.html)
[Kafka参数图鉴——unclean.leader.election.enable](https://blog.csdn.net/u013256816/article/details/80790185)
[某互联网大厂kafka最佳实践](https://www.jianshu.com/p/8689901720fd)

