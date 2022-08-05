---
title: 简单聊聊kafka
tags:
  - kafka
id: '12539'
categories:
  - - skills
date: 2022-05-14 16:35:28
---

# 消息队列的通信模型

## 点对点模式（queue）

消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。一条消息被消费以后，queue中就没有了，不存在重复消费。

## 发布/订阅（topic）

消息生产者（发布）将消息发布到`topic`中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到`topic`的消息会被所有订阅者消费（类似于关注了微信公众号的人都能收到推送的消息）。

# Kafka

`Apache Kafka`由`LinkedIn`开发，最初被设计用来解决`LinkedIn`公司内部海量日志传输等问题。

## 架构

![file](https://www.52ynn.top/wp-content/uploads/2022/05/627f6d0f5c780.png)

*   Producer：生产者，消息的产生者，是消息的入口。
*   Kafka cluster： kafka 集群，一台或多台服务器组成。
    *   Broker： Broker 是指部署了 kafka 实例的服务器节点。每个服务器上有一个或多个 kafka 实例。
    *   Topic： 消息的主题，可以理解为消息的分类，kafka 的数据就保存在 topic。每个 broker 都可以创建多个 topic。实际应用中通常是一个业务线建一个 topic。
    *   Partition: Topic 的分区，每个 topic 可以有多个分区，分区的作用是做负载。提高 kafka 吞吐量。同一个 Topic 在不同的分区的数据是不重复的，partition 的表现形式就是一个一个的文件夹。
    *   Replication： 每一个分区都有多个副本，副本的作用是做备胎。当主分区（Leader）故障的时候会选择一个备胎（Follower）上位，成为 Leader。
*   Consumer： 消费者，即消息的消费方，是消息的出口。
    *   Consumer Group： 我们可以将多个消费组组成一个消费者组，在Kafka 的设计中同一个分区的数据只能被消费者组中的某一个消费者消费。同一个消费者组的消费者可以消费同一个 topic 的不同分区的数据。

## 工作流程

![file](https://www.52ynn.top/wp-content/uploads/2022/05/627f708252162.png)

1.  生产者从 kafka 集群获取分区 leader 信息
2.  生产者将消息发送给 leader
3.  leader 将消息写入本地磁盘
4.  follower 从 leader 拉取消息数据
5.  follower 将消息写入本地磁盘后向 leader 发送 ACK
6.  leader 收到所有的 follower 的 ACK 之后向生产者发送 ACK

## 选择 partition 的原则

1.  写入指定 partition
2.  没有指定 partition，但是设置了 key，则会根据 key 的值 hash 出一个 partition
3.  如果没指定 partition，也没设置 key，则轮询

## ACK 应答机制

*   0 代表 producer 往集群发送数据不需要等到集群的返回，不确保消息发送成功。
*   1 代表producer 往集群发送数据只需要 leader 应答就可以发送下一条，只确保 leader 发送成功。
*   all 代表 producer 往集群发送数据需要所有的 follower 都完成从 leader 的同步才会发送下一条，确保 leader 发送成功和所有的副本都完成备份。

## Topic 和数据日志

topic 是同一类别的消息记录（record）的集合。在 kafka 中，一个主题通常有多个订阅这。对于每个主题，kafka集群维护了一个分区数据日志文件的结构如下： ![file](https://www.52ynn.top/wp-content/uploads/2022/05/627f73d5c6f48.png) 每个 partition 都是一个有序并且不可变的消息记录集合。当心的数据写入时，就被追加到 partition 的末尾。在每个 partition 中，每条消息都会被分配一个顺序的唯一标识，这个标识被称为 offset，即偏移量。 Kafka 可以配置一个保留期限，用来标识日志会在 Kafka 集群内保留多长时间。Kafka 集群会保留在保留期限内所有发布的消息，超过期限会被清空。

## 消费数据

多个消费者实例可以组成一个消费者组，并用一个标签来标识这个消费者组。一个消费者组中的不同消费者实例可以运行在不同的进程甚至不同的服务器上。 ![file](https://www.52ynn.top/wp-content/uploads/2022/05/627f750fde275.png) 消费者可以消费多个 partition 但一个 partition 只能被一个Consumer group 中的一个消费者消费。

# 应用场景

消息队列 追踪网站活动 Metrics