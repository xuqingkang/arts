# ***介绍
  Kafka作为海量消息队列。
  
  今天我们初步来看一下Kafka的基本安装和使用。同样的我们采用192.168.32.150~152机器进行集群部署。

# ***入门步骤
## 1，下载&解压
  可以在镜像服务下载kafka。
  http://mirrors.shu.edu.cn/apache/kafka/2.1.0/kafka_2.11-2.1.0.tgz 
  
  使用tar命令解压kafka_2.11-2.1.0.tgz,比如解压到/root/kafka/kafka_2.11-2.1.0目录
  
## 2，zookeeper准备
  可以使用kafka自带的zookeeper，也可以自行预先安装zookeeper。
  
## 3，修改配置文件
  conf/server.properties
  
  其中，broker.id要求每台机器设置得都不一样，设置为全局唯一值。
  zookeeper.connect设置为zk的地址，本例中指定了"/kafka"作为父结点。
```
# The id of the broker. This must be set to a unique integer for each 
broker.id=151
# root directory for all kafka znodes.
zookeeper.connect=192.168.32.150:2181,192.168.32.151:2181,192.168.32.152:2181/kafka
```

## 4，启动
通过kafka-server-start.sh脚本启动，并指定配置文件为server.properties,需要每台机器均启动。
```
#!/bin/sh

KAFKA_HOME=/root/kafka/kafka_2.11-2.1.0
$KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties &
```
## 5，验证和使用
### 5.1 创建主题
使用kafka的kafka-topics.sh脚本创建队列，在创建的时候需要指定--zookeeper选项设置zk地址，并指定消息复制数和partition数量(同一个topic均分到多台机器)，比如我们采用如下的脚本（createTopic.sh）固化创建Topic的操作，可根据实际情况调整。
并运行createTopic.sh testTopic创建主题
```
#!/bin/sh

if [ $# -lt 1 ]; then
  echo "The topic Name must be specified."
  exit
fi

topic_name=$1
zk_options=192.168.32.150:2181,192.168.32.151:2181,192.168.32.152:2182/kafka

KAFKA_HOME=/root/kafka/kafka_2.11-2.1.0
$KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper $zk_options --replication-factor 1 --partitions 3 --topic $topic_name
```
### 5.2 查看主题
我们仍然采用类似的方法实现对Topic的查看操作，脚本(descTopic.sh)内容如下：
```
#!/bin/sh

if [ $# -lt 1 ]; then
  echo "The topic Name must be specified."
  exit
fi

topic_name=$1
zk_options=192.168.32.150:2181,192.168.32.151:2181,192.168.32.152:2182/kafka

KAFKA_HOME=/root/kafka/kafka_2.11-2.1.0
$KAFKA_HOME/bin/kafka-topics.sh --describe --zookeeper $zk_options --topic $topic_name
```

采用descTopic.sh testTopic可以查看到主题信息如下：
其中testTopic有三个Partition分布于150~152三台机器。
```
Topic:testTopic PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: testTopic        Partition: 0    Leader: 151     Replicas: 151   Isr: 151
        Topic: testTopic        Partition: 1    Leader: 152     Replicas: 152   Isr: 152
        Topic: testTopic        Partition: 2    Leader: 150     Replicas: 150   Isr: 150
```
### 5.3 生产者发送消息
使用kafka-console-producer向Topic发消息，运行成功后直接在界面输入消息即可
```
kafka-console-producer.sh --broker-list 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --topic testTopic
```
### 5.4 消费者消费消息
使用kafka-console-consumer从Topic接收消息
```
kafka-console-consumer.sh --bootstrap-server 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --topic testTopic
```