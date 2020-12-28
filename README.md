# Kafka
What is Kaka?

&emsp;&emsp;Kafka 是一个多分区、多副本且基于 ZooKeeper 
协调的分布式消息系统，目前 Kafka 已经定位为一个分布式流式
处理平台，它以高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用。

主要用途：
* 消息系统:&emsp;Kafka和传统的消息系统都具备系统解耦、冗余存储、流量削峰、缓冲、异步通信、扩展性、可恢复性等功能。与此同时，Kafka还提供了大多数消息系统难以实现的消息顺序性保障及回溯消费的功能。
* 存储系统:&emsp;Kafka把消息持久化到磁盘，只需要把对应的数据保留策略设置为“永久”或启用主题的日志压缩功能即可。
* 流式处理平台:&emsp;Kafka不仅为每个流行的流式处理框架提供了可靠的数据来源，还提供了一个完整的流式处理类库，比如窗口、连接、变换和聚合等各类操作。

## 一、基本概念

&emsp;&emsp;一个典型的Kafka体系架构包括若干Producer、若干Broker、若干Consumer，以及一个ZooKeeper集群，其中ZooKeeper是Kafka用来负责集群元数据的管理、控制器的选举等操作的。Producer将消息发送到Broker,Broker负责将收到的消息存储到磁盘中，而Consumer负责从Broker订阅并消费消息

![image](https://github.com/zhangqw2/Kafka/blob/main/kafka%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84.png)

整个Kafka体系结构中引入以下3个术语：

 (1）Producer: 生产者，也就是发送数据的一方。生产者负责创建消息，然后将其投递到Kafka中.

 (2) Consumer: 消费者，也就是接收消息的一方。消费者连接到Kafka上并接收消息，进而进行相应的业务逻辑处理。
 
 (3) Broker: 服务代理节点。对于Kafka而言，Broker可以简单地看作一个独立的Kafka服务节点或Kafka服务实例。一个或多个Broker组成一个Kafka集群。一般习惯使用首字母小写的broker来表示服务代理节点。
