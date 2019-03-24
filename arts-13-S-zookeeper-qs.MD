# ***����
  Zookeeper��һ�������ܵķֲ�ʽӦ��Э���������ṩ���ơ����ù�����ͬ���������ȡ�������zookeeperʵ�ֹ�ʶ����������쵼��ѡ��(����kafka��topic partition���ڶิ�Ƶ�ʱ��ѡ��leader�ĳ���)���͵�������(��Ϊԭ�ģ�You can use it off-the-shelf to implement consensus, group management, leader election, and presence protocols)���ǳ���Ŀ�Դ����Ĭ����zookeeperʵ�����ù�����leaderѡ�ٷ���
  
  �������ǳ�������һ��zookeeper�Ļ�����װ��ʹ��

# ***���Ų���
  ���ǻ���ֱ��ʹ��192.168.32.150~152����̨�����齨zookeeper��Ⱥ��
  ���ݹ�����˵����zookeeper�İ�װ��ʹ�ü����ǿ�Դ����򵥵��ˡ���ֹĿǰ(2019.3.23)���ȶ��汾Ϊ��3.4.13
## 1,���غͽ�ѹ
����ͨ��zookeeper���������Ǿ���������������°汾��https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper

ֱ��ͨ��tar�����ѹzookeeper-3.4.13.tar.gz��ĳ��Ŀ¼������/root/zookeeper/zookeeper-3.4.13

## 2���޸�����
### 2.1 �������ļ�
��zookeeper��confĿ¼����zoo_sample.cfg����Ϊzoo.cfg�ļ���Ȼ���޸��ļ��������£���Ҫ�޸�dataDir����������ͨ��server.150���������ð���̨������Ⱥ���úá�

ע�⣺ÿ̨����������ͬ�������á�
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/root/zookeeper/zookeeper-3.4.13/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.150=rhel150:2888:3888
server.151=rhel151:2888:3888
server.152=rhel152:2888:3888
```
### 2.2 pid�ļ�
��������һ����ָ����dataDirΪ/root/zookeeper/zookeeper-3.4.13/dataĿ¼��������Ŀ¼�����ڿ����ȴ�����Ȼ���ڸ�Ŀ¼���½�һ����Ϊ"myid"���ļ�������Ϊ����zoo.cfg�ļ��ж�Ӧ��id��
��������zoo.cfg�ļ�Ϊ

```
server.150=rhel150:2888:3888
server.151=rhel151:2888:3888
server.152=rhel152:2888:3888
```
����ߵ�server.xxx�����xxx����id��������192.168.32.150������myid�ļ�����Ӧ������Ϊ150����������������myid�ļ�ͬ����

���������myid�ļ��Ļ������ǻ�������ʱ����־�ļ���Ĭ��Ϊ����zkServer.sh�ű�Ŀ¼��zookeeper.out���п����������´���

```
2019-03-23 08:36:06,732 [myid:] - ERROR [main:QuorumPeerMain@88] - Invalid config, exiting abnormally
org.apache.zookeeper.server.quorum.QuorumPeerConfig$ConfigException: Error processing /root/zookeeper/zookeeper-3.4.13/bin/../conf/zoo.cfg
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parse(QuorumPeerConfig.java:156)
        at org.apache.zookeeper.server.quorum.QuorumPeerMain.initializeAndRun(QuorumPeerMain.java:104)
        at org.apache.zookeeper.server.quorum.QuorumPeerMain.main(QuorumPeerMain.java:81)
Caused by: java.lang.IllegalArgumentException: /root/zookeeper/zookeeper-3.4.13/data/myid file is missing
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parseProperties(QuorumPeerConfig.java:408)
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parse(QuorumPeerConfig.java:152)
```

## 3������
������ÿ̨������׼��һ�������ű����ű��ǳ��򵥣�����zookeeper��zkServer.sh����Ϳ��ԡ�
```
#!/bin/sh

ZK_HOME=/root/zookeeper/zookeeper-3.4.13
$ZK_HOME/bin/zkServer.sh start
```
## 4��ȷ�Ϻ�ʹ��
  
### 4.1,������
  zookeeper��java�����ģ���������ʹ��ps -ef | grep java���Կ���zookeeper�Ľ��̡����ʹ��netstat�鿴2181�˿ڻῴ���䱻����������
  
```
[root@rhel150 ~]# ps -ef | grep java
root       3845      1  5 08:49 ?        00:00:03 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zookeeper/zookeeper-3.4.13/bin/../build/classes:/root/zookeeper/zookeeper-3.4.13/bin/../build/lib/*.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/slf4j-log4j12-1.7.25.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/slf4j-api-1.7.25.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/netty-3.10.6.Final.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/log4j-1.2.17.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/jline-0.9.94.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/audience-annotations-0.5.0.jar:/root/zookeeper/zookeeper-3.4.13/bin/../zookeeper-3.4.13.jar:/root/zookeeper/zookeeper-3.4.13/bin/../src/java/lib/*.jar:/root/zookeeper/zookeeper-3.4.13/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zookeeper/zookeeper-3.4.13/bin/../conf/zoo.cfg
```

### 4.2,key����
  zookeeper�洢�����������ṹ���棬����ʹ�����������ȡ�ڵ����ݡ�
  ����ʹ��zookeeper��zkCli.sh���ӷ��񣬱���150������zookeeper
```
./zkCli.sh -server 192.168.32.150:2181
```

���²���create /zktest zkvalue�����ڵ������ֵ����ͨ��get��ȡ�ڵ���Ϣ(����ֵ)
```
[zk: 192.168.32.150:2181(CONNECTED) 1] create /zktest zkvalue
Created /zktest
[zk: 192.168.32.150:2181(CONNECTED) 2] get /zktest
zkvalue
cZxid = 0x100000005
ctime = Sat Mar 23 13:55:39 CST 2019
mZxid = 0x100000005
mtime = Sat Mar 23 13:55:39 CST 2019
pZxid = 0x100000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
```

��չһ�£�����ʹ��zkCli���������κ�һ̨�����ϵ�zk���ܲ�ѯ���ո��½��Ľڵ���Ϣ��