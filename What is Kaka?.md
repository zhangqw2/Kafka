# Kafka

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

**副本**

&emsp;&emsp;Kafka为分区引入了多副本(Replica)机制， 通过增加副本数量可以提升容灾能力。同一
分区的不同副本中保存的是相同的消息（在同一时刻，副本之间并非完全一样），副本之间是“一主多从”的关系，其中leader副本负责处理读写请求，follower副本只负责与leader副本的消息同步。副本处于不同的 broker，当leader副本出现故障时，从follower副本中重新选举新的leader本对外提供服务。 Kafka 通过多副本机制实现了故障的自动转移，当Kafka集群中某个broker失效时仍然能保证服务可用

![image](https://github.com/zhangqw2/Kafka/blob/main/%E5%A4%9A%E5%89%AF%E6%9C%AC%E6%9E%B6%E6%9E%84.png)

&emsp;&emsp;如上图所示，Kafka集群中有4个broker，某个主题3个分区，3个副本因子，如此每个分区便有1个leader副本和2个follower副本，生产者和消费者只与leader副本进行交互，而follower副本只负责消息同步，很多时候follower副本中的消息相对于leader副本而言会有一定滞后。

&emsp;&emsp;Kafka消费端也具备一定的容灾能力。Consumer使用拉(Pull)模式从服务端拉取消息并且保存消费的具体位置，当消费者宕机后恢复上线时可以根据之前保存的消费位置重新拉取需要的消息进行消费，这样就不会造成消息丢失。

**AR、ISR、OSR**
&emsp;&emsp;分区中的所有副本统称为AR(Assigned Replicas)。所有域leader副本保持一定程度同步的副本(包含leader副本在内)组成ISR(In-Sync Replicas),ISR集合是AR集合中的一个子集。消息会先发送到leader副本，然后follower副本才能从leader副本拉取消息进行同步，同步期间内follower副本相对于leader副本而言会有一定程度的滞后。可以通过参数进行配置，限制滞后范围。与leader副本同步滞后过多的副本(不包括leader副本)组成OSR(Out-of-Sync Replicas)，由此可见，AR=ISR+OSR。在正常情况下，所有的follower副本都应该与leader副本保持一定程度的同步，即AR=ISR，OSR集合为空。

![image](https://github.com/zhangqw2/Kafka/blob/main/AR-ISR-OSR.png)

&emsp;&emsp;leader副本负责维护和跟踪ISR集合中所有follower副本的滞后状态，当follwer副本落后太多或失效时，leader副本会把它从ISR集合中剔除。如果OSR集合中有follwer副本”追上“了leader副本，那么leader副本会把它从OSR集合转移至ISR集合。默认情况下，当leader副本发生故障时，只有在ISR集合中的副本才有资格被选举为新的leader，而在OSR集合中副本则没有任何机会(可以通过修改相应的参数配置来修改此原则)。

**ISR、HW、LEO**

&emsp;&emsp;HW是High Watermark的缩写，俗称高水位，它标识了一个特定的消息偏移量(offset)，消费者只能拉取到这个offset之前的消息。

![image](https://github.com/zhangqw2/Kafka/blob/main/%E5%88%86%E5%8C%BA%E5%81%8F%E7%A7%BB%E9%87%8F%E8%AF%B4%E6%98%8E.png)

&emsp;&emsp;如上图所示，它代表一个日志文件，这个日志文件中有9条消息，第一条消息的offset为0，最后一条offset为8，offset为9的消息用虚线框表示，代表下一条待写入的消息。日志文件HW为6，表示消费组只能拉取到offset在0~5之间的消息，而offset为6的消息对消费者而言是不可见的。

&emsp;&emsp;LEO是Log End Offset的缩写，它标识当前日志文件中下一条待写入消息的offset，LEO的大小相当于当前日志分区中最后一条消息的offset值加1。分区ISR集合中每个副本都会维护自身的LEO，而ISR集合中最小的LEO即为分区的HW，对消费者而言只能消费HW之前的消息。

&emsp;&emsp;由此可见，Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制。事实上，同步复制要求所有能工作的follower副本都复制完，这条消息才会被确认为已成功提交，这种复制方式极大地影响了性能。而在异步复制方式下，follower副本异步地从leader副本中复制数据，数据只要被leader副本写入就被认为已经成功提交。在这种情况下，如果follower副本都还没有复制完而落后于leader副本，突然leader副本着机，则会造成数据丢失。 Kafka 使用的这ISR的方式则有效地权衡了数据可靠性和性能之间的关系。

## 二、安装与配置


### 2.1  JDK1.8

&emsp;&emsp;以Linux服务器为例,下载jdk-8ul8l-linux-x6.tar.gz,解压至/opt
```
 vim /etc/profile ,在末尾添加如下配置:
 
export JAVA_HOME=/opt/jdk1.8.0_181 
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin 
export CLASSPATH= ./://$JAVA_HOME/lib:JRE_HOME/lib
 
然后,执行source /etc/profile 命令时配置生效。
执行java -version命令，验证是否安装成功，输出java版本信息
```
### 2.2 ZooKeeper安装与配置

&emsp;&emsp;ZooKeeper是安装Kafka集群的必要组件，Kafka通过ZooKeeper来实施对元数据信息的管理，包括集群、broker、主题、分区等内容。在ZooKeeper中共有3个角色:leader、follower和observer,同一时刻ZooKeeper集群中只会有一个leader，其他都是follower和observer。observer不参与投票，默认情况下ZooKeeper中只有leader和follwer两个角色。

```
第一步,下载 Zookeeper-3.4.12.tar.gz 安装包，并解压至/opt目录下。
第二步,向/etc/profile配置文件中添加如下内容，并执行source /etc/profile命令使配置生效:
export ZOOKEEPER_HOME=/opt/zookeeper-3.4.12 
export PATH=$PATH:$ZOOKEEPER_HOME/bin
第三步,修改Zookeeper的配置文件。首先进入$ZOOKEEPER_ HOME/conf 目录,并将
zoo_sample.cfg 文件修改为 zoo.cfg:
 cp zoo_sample.cfg zoo.cfg
 修改zoo.cfg配置文件,zoo.cfg文件的内容参考如下:
 
 # ZooKeeper服务器心跳时间， 单位为 ms
tickTime=2000
＃投票选举新 leader 的初始化时间
initLimit=lO
# leader foll ower 跳检测最大容忍时间，响应超过 syncLimit*tickTime leader 认为
# follower “死掉 ，从服务器列表中删除 follower
syncLimit=5
＃数据目录
dataDir=/tmp/zookeeper/data 
日志目录
dataLogDir=/tmp/zookeeper/log 
# ZooKeeper 对外服务端口
clientPort=2181

默认情况下,Linux系统中没有/tmp/zookeeper/data和tmp/zookeeper/log这个两个目录,所以需要手动创建。
第四步,在${dataDir}(/tmp/zookeeper/data)目录下创建一个myid文件，并写入一个数值，比如0。myid文件里存放的是服务器编号。
第五步,启动Zookeeper服务
zkServer.sh start
可以通过zkServer.sh status命令查看Zookeeper服务状态。

以上是Zookeeper的单机配置模式

对于集群模式:
假设有三台Linux服务器，首先修改3台的服务器的/etc/hosts文件，示例如下:
192.168.0.2 node1
192.168.0.3 node2
192.168.0.4 node3

然后在这3台机器的zoo.cfg文件中添加以下配置:

server.0=192.168.0.2:2888:3888
server.1=192.168.0.3:2888:3888
server.2=192.168.0.4:2888:3888

为了便于说明，可以抽象出一个公式,即server.A=B:C:D。其中A是一个数字，代表服务器的编号，就是myid文件里面的值。
集群中每台服务器的编号必须唯一，所以要保证每台服务器中的myid文件中的值不同。B代表的是服务器的IP地址。
C表示服务器与集群中的leader服务交换信息的端口。D表示选举时服务器相互通信的端口。

配置完成后在3台机器上各自执行zkServer.sh start启动服务。
```
### 2.3 Kafka安装与配置

```
下载kafka_2.11-2.0.0.tgz，解压至/opt目录下
参考zookeeper将KAFKA_HOME添加到/etc/pfofile文件中

修改broker的配置文件$KAFKA_HOME/conf/server.properties,主要关注以下几个配置参数即可:
# broker的编号，如果集群中有多个broker ，则每个broker的编号需要设置的不同
broker.i d=O 
# broker对外提供服务入口地址
listeners= PLAINTEXT//localhost:9092
＃存放消息日志文件的地址
log.dirs= /tmp/kafka-logs
# Kafka 所需的ZooKeeper集群地址，为了方便演示，我们假设Kafka和ZooKeeper 都安装在本机
zookeeper.connect=localhost:2181/kafka

如果是单机模式，那么修改完上述配置就可以启动了。
如果是集群模式，确保集群中每个broker的broker.id 配置数的值不一样，
以及 listeners 配置参数也需要修改为与broker对应的IP地址域名，之后就可以各自启动服务 。
注意 ，在启动Kafka服务之前同样需要确保zookeeper.connect参数所配置的ZooKeeper服务正确启动。

```

&emsp;&emsp;可以使用以下命令启动Kakfa
```
后台启动:
bin/kafka-server-start.sh -daemon config/server.properties
bin/kafka-server-start.sh config/server.properties &
界面启动
bin/kafka-server-start.sh config/server.properties

通过jps -l 命令查看kafka服务进程是否已经启动
```
