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

**Topic与分区**

&emsp;&emsp;Kafka中的消息是以主题为单位进行归类的，生产者负责将消息发送到特定的主题，而消费者负责订阅主题并进行消费。主题是一个逻辑上的概念，它还可以细分为多个分区，一个分区只属于单个主题，同一主题下的不同分区包含的消息是不同的，分区在存储层面可以看做一个可追加的日志文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量(offset)。offset是消息在分区中的唯一标识，Kafka通过它来保证消息在分区内的顺序性，但是Kafka不保证消息在主题内有序。

![image](https://github.com/zhangqw2/Kafka/blob/main/%E6%B6%88%E6%81%AF%E8%BF%BD%E5%8A%A0%E5%86%99%E5%85%A5.png)

&emsp;&emsp;如上图所示，主题中有4个分区，消息被顺序追加到每个分区日志文件的尾部。Kafka中的分区可以分布在不同的服务器(broker)上，亦即，一个主题可以横跨多个broker，以此来提供比单个broker更强大的性能。
&emsp;&emsp;每一条消息被发送到broker之前，会根据分区规则选择存储到哪个具体的分区，如果分区规则设定的合理，所有消息都可以均匀地分配到不同的分区中。*如果一个主题只对应一个文件，那么这个文件所在的机器I/O将会成为这个主题的性能瓶颈，而分区解决了这个问题*。在创建主题的时候可以通过指定的参数来设置分区的个数，当然也可以在主题创建完成之后修改分区的数量，通过增加分区的数量可以是实现水平扩展。
&emsp;&emsp;Kafka为分区引入了多副本(Replica)机制， 通过增加副本数量可以提升容灾能力。同一
分区的不同副本中保存的是相同的消息（在同一时刻，副本之间并非完全一样），副本之间是“一主多从”的关系，其中leader副本负责处理读写请求，follower副本只负责与leader副本的消息同步。副本处于不同的 broker，当leader副本出现故障时，从follower副本中重新选举新的leader本对外提供服务。 Kafka 通过多副本机制实现了故障的自动转移，当Kafka集群中某个broker失效时仍然能保证服务可用
