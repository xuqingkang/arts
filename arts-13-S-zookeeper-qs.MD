# ***介绍
  Zookeeper是一个高性能的分布式应用协调服务，其提供名称、配置管理、同步和组服务等。可以用zookeeper实现共识、组管理、领导人选举(比如kafka的topic partition存在多复制的时候选择leader的场景)、和到场服务(因为原文：You can use it off-the-shelf to implement consensus, group management, leader election, and presence protocols)。非常多的开源软件默认由zookeeper实现配置管理和leader选举服务。
  
  今天我们初步来看一下zookeeper的基本安装和使用

# ***入门步骤
  我们还是直接使用192.168.32.150~152共三台机器组建zookeeper集群。
  根据官网的说明，zookeeper的安装和使用几乎是开源界最简单的了。截止目前(2019.3.23)的稳定版本为：3.4.13
## 1,下载
我们通过zookeeper官网或者是镜像服务器下载最新版本，https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper

## 2，修改配置
### 2.1 主配置文件
在zookeeper的conf目录，把zoo_sample.cfg拷贝为zoo.cfg文件，然后修改文件内容如下：主要修改dataDir参数，还有通过server.150这样的配置把三台机器集群配置好。

注意：每台机器均采用同样的配置。
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
### 2.2 pid文件
我们在上一节中指定了dataDir为/root/zookeeper/zookeeper-3.4.13/data目录，如果这个目录不存在可以先创建，然后在改目录下新建一个名为"myid"的文件，内容为上面zoo.cfg文件中对应的id。
比如上面zoo.cfg文件为

```
server.150=rhel150:2888:3888
server.151=rhel151:2888:3888
server.152=rhel152:2888:3888
```
这里边的server.xxx后面的xxx就是id，所以在192.168.32.150机器上myid文件内容应该设置为150，其他两个机器的myid文件同理。

如果不设置myid文件的话，我们会在启动时在日志文件（默认为运行zkServer.sh脚本目录下zookeeper.out）中看到类似如下错误：

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

## 3，启动
我们在每台机器上准备一个启动脚本，脚本非常简单，调用zookeeper的zkServer.sh命令就可以。
```
#!/bin/sh

ZK_HOME=/root/zookeeper/zookeeper-3.4.13
$ZK_HOME/bin/zkServer.sh start
```
## 4，确认和使用
  
### 4.1,检查进程
  zookeeper是java开发的，所以我们使用ps -ef | grep java可以看到zookeeper的进程。如果使用netstat查看2181端口会看到其被正常监听。
  
```
[root@rhel150 ~]# ps -ef | grep java
root       3845      1  5 08:49 ?        00:00:03 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /root/zookeeper/zookeeper-3.4.13/bin/../build/classes:/root/zookeeper/zookeeper-3.4.13/bin/../build/lib/*.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/slf4j-log4j12-1.7.25.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/slf4j-api-1.7.25.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/netty-3.10.6.Final.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/log4j-1.2.17.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/jline-0.9.94.jar:/root/zookeeper/zookeeper-3.4.13/bin/../lib/audience-annotations-0.5.0.jar:/root/zookeeper/zookeeper-3.4.13/bin/../zookeeper-3.4.13.jar:/root/zookeeper/zookeeper-3.4.13/bin/../src/java/lib/*.jar:/root/zookeeper/zookeeper-3.4.13/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /root/zookeeper/zookeeper-3.4.13/bin/../conf/zoo.cfg
```

### 4.2,key操作
  zookeeper存储的数据以树结构保存，可以使用命令创建、获取节点数据。
  我们使用zookeeper的zkCli.sh连接服务，比如150机器的zookeeper
```
./zkCli.sh -server 192.168.32.150:2181
```

如下采用create /zktest zkvalue创建节点和设置值，并通过get获取节点信息(包括值)
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

扩展一下：我们使用zkCli连接其他任何一台机器上的zk都能查询到刚刚新建的节点信息。