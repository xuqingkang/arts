# ***Kafka�������
  ������˵һ��Kafka����ķ����������������ʹ��192.168.32.150~152����̨�����齨��kafka��Ⱥ��

## 1������partition��־˵��
  ����Kafka���������Ź�����ͼʾ˵����
  ��������ͼ�ǹ���kafka�����ģ�һ��topic��Ϣ���ж��partition�ֲ��ڲ�ͬ������֮�ϣ�ÿ��partition�µ���Ϣ�������(offset)
  
  ![image](./images/ARTS-15-S-log_anatomy.png)
  
  ��һ��ͼ˵�������µ���Ϣ����ͬ��������ʹ��offset��λ����֪���Լ����ѵ���Ϣλ�á�
  
  ![image](./images/ARTS-15-S-log_consumer.png)

  �ڴ�������ʱ:./kafka-topics.sh  --create�����������������ã�һ��"--partitions"��Ҫ˵��һ������ּ�����������һ����"--replication-factor"Ϊ�������ӣ�ָ��partition�ĸ��Ʊ�������
  
  ��̨��Ϣ�ܱ������Ϣ�����޶ȵģ�������partition������չ�ķ�ʽ֧�ֺ�����Ϣ�ı��棬��һ��Ҳ����������������Ϣ����������ʱ�ĸ��ؾ������⡣��������������Ҫ����Ϣ���ݴ��ԣ����ڵ���partition����һ����νleader�Ľ�ɫ������Ϣ����ȡ�����follower���𱸷���Ϣ����leader崻�ʱ��follower�ӹ���Ϣ������
## 2����֤�Բ���
### 2.1�� ��������
  �����ǲ������������һ��testTopic���⣬����������Ϊ3��������Ϊ����Ϊ1
  kafka-topics.sh --create --zookeeper $zk_options --replication-factor 1 --partitions 3 --topic testTopic
  
  ����ͼ��ʾ���鿴kafka��̨�����ϵ�kafka-logsĿ¼���ῴ��������partition��Ŀ¼�������ˡ�
  ![image](./images/ARTS-15-2-topicOnMachine.png)

### 2.2��������Ϣ
����һ��kafka�����ߣ�ֱ�ӷ���������Ϣ�������ı��ֱ�Ϊ6��1��6��2��6��3

```
[root@rhel150 ~]# ./kafka/kafka_2.11-2.1.0/bin/kafka-console-producer.sh --broker-list 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --topic testTopic
>111111
>222222
>333333
>
```
�ֱ���ÿ��kafka����brokerĿ¼�鿴kafka-logsĿ¼�µ�partition��־�ļ����Կ�����Ϣȷʵ�Ǿ��ֵ���ͬ�Ļ�������ġ�

150������
```
[root@rhel150 kafka_2.11-2.1.0]# cat kafka-logs/testTopic-2/00000000000000000002.log 
>?Vr\i?R��?i?R���
            111111
```
151������
```
[root@rhel151 testTopic-0]# cat kafka-logs/testTopic-0/00000000000000000002.log 
>?i?R��i?R��
             333333
```
152������
```
[root@rhel152 testTopic-1]# cat kafka-logs/testTopic-1/00000000000000000002.log 
>?i?R��i?R��
             222222
```
### 2.3����Ϣ����
����һ��consumer��ָ��"--from-beginning"��ͷ������Ϣ���Ϳ��Խ���ǰ�����е���Ϣ��

```
[root@rhel150 bin]# ./kafka-console-consumer.sh --bootstrap-server 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --from-beginning --topic testTopic
333333
222222
111111
```

�����Ϣ��Ϣ��ʱ��ָ����--partition��������ֻ�����ѹ̶�partition��Ϣ�����µ�consumer��ֻ������partition 1����Ϣ(222222��������Ϣ���Բμ�������Ƕ�Ӧ152�����ϵ�partition)��
```
[root@rhel152 bin]# ./kafka-console-consumer.sh --bootstrap-server 192.168.32.150:9092,192.168.32.151:9092,192.168.32.152:9092 --from-beginning --partition 1 --topic testTopic
222222
```