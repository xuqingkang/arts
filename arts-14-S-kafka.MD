# ***����
  Kafka��Ϊ������Ϣ���С�
  
  �������ǳ�������һ��Kafka�Ļ�����װ��ʹ�á�ͬ�������ǲ���192.168.32.150~152�������м�Ⱥ����

# ***���Ų���
## 1������&��ѹ
  �����ھ����������kafka��
  http://mirrors.shu.edu.cn/apache/kafka/2.1.0/kafka_2.11-2.1.0.tgz 
  
  ʹ��tar�����ѹkafka_2.11-2.1.0.tgz,�����ѹ��/root/kafka/kafka_2.11-2.1.0Ŀ¼
  
## 2��zookeeper׼��
  ����ʹ��kafka�Դ���zookeeper��Ҳ��������Ԥ�Ȱ�װzookeeper��
  
## 3���޸������ļ�
  conf/server.properties
  
  ���У�broker.idҪ��ÿ̨�������õö���һ��������Ϊȫ��Ψһֵ��
  zookeeper.connect����Ϊzk�ĵ�ַ��������ָ����"/kafka"��Ϊ����㡣
```
# The id of the broker. This must be set to a unique integer for each 
broker.id=151
# root directory for all kafka znodes.
zookeeper.connect=192.168.32.150:2181,192.168.32.151:2181,192.168.32.152:2181/kafka
```

## 4������
ͨ��kafka-server-start.sh�ű���������ָ�������ļ�Ϊserver.properties,��Ҫÿ̨������������
```
#!/bin/sh

KAFKA_HOME=/root/kafka/kafka_2.11-2.1.0
$KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties &
```
## 5����֤��ʹ��
### 5.1 ��������
ʹ��kafka��kafka-topics.sh�ű��������У��ڴ�����ʱ����Ҫָ��--zookeeperѡ������zk��ַ����ָ����Ϣ��������partition����(ͬһ��topic���ֵ���̨����)���������ǲ������µĽű���createTopic.sh���̻�����Topic�Ĳ������ɸ���ʵ�����������
������createTopic.sh testTopic��������
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
### 5.2 �鿴����
������Ȼ�������Ƶķ���ʵ�ֶ�Topic�Ĳ鿴�������ű�(descTopic.sh)�������£�
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

����descTopic.sh testTopic���Բ鿴��������Ϣ���£�
����testTopic������Partition�ֲ���150~152��̨������
```
Topic:testTopic PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: testTopic        Partition: 0    Leader: 151     Replicas: 151   Isr: 151
        Topic: testTopic        Partition: 1    Leader: 152     Replicas: 152   Isr: 152
        Topic: testTopic        Partition: 2    Leader: 150     Replicas: 150   Isr: 150
```
### 5.3 �����߷�����Ϣ
ʹ��kafka-console-producer��Topic����Ϣ�����гɹ���ֱ���ڽ���������Ϣ����
```
kafka-console-producer.sh --broker-list 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --topic testTopic
```
### 5.4 ������������Ϣ
ʹ��kafka-console-consumer��Topic������Ϣ
```
kafka-console-consumer.sh --bootstrap-server 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --topic testTopic
```