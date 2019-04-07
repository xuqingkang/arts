# ***Redis
  Redis是广泛被使用的内存数据库了，其通常被用作缓存数据库，但也有用于数据库或者是消息代理(message broker)的。我们见得多的是讲Web应用的Session信息存储在Redis中实现会话的无状态化改造。
  
  以下我们基于最新版本的Redis(5.0.4)讲述安装和集群配置，我们还是对测试环境做下说明，192.168.32.115、116、117三台主机。

## 1，安装前准备
  需要实现安装gcc编译器，没有的话提前安装便可。

## 2，安装说明
  直接在redis官网下载：https://redis.io/download。
  
  下载后直接解压redis-5.0.4.tar.gz,然后编译jemalloc就可以编译redis了。
###  2.1 编译jemalloc
  进入deps目录进行编译，命令如下：
  
```
[root@rhel7 redis-5.0.4] cd deps
[root@rhel7 deps] make jemalloc
```
  如果不编译jemalloc直接编译redis的话就会遇到下面的错误：
  
```
找不到jemalloc.h等
cc: 错误：../deps/hiredis/libhiredis.a：没有那个文件或目录
cc: 错误：../deps/lua/src/liblua.a：没有那个文件或目录
```

  redis官方说明文档对jemalloc是这样解释的：

```
Selecting a non-default memory allocator when building Redis is done by setting
the `MALLOC` environment variable. Redis is compiled and linked against libc
malloc by default, with the exception of jemalloc being the default on Linux
systems. This default was picked because jemalloc has proven to have fewer
fragmentation problems than libc malloc
```
### 2.2 编译redis
  回到redis根目录直接make即可
  
```
[root@rhel7 deps] cd ../
[root@rhel7 redis-5.0.4] make
```
### 2.3 安装确认和二进制拷贝
  编译成功后在src目录会有如下几个二进制文件，==redis-server==为服务端，redis-cli是命令行工具。
```
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-check-aof
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-check-rdb
-rwxr-xr-x. 1 root root 4365440 4?   4 22:52 redis-benchmark
-rwxr-xr-x. 1 root root 4805784 4?   4 22:52 redis-cli
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-sentinel
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-server
```
## 3，集群创建
### 3.1 配置文件准备
  redis的主要配置文件就是==redis.conf==文件，修改好这个文件，然后运行redis-server的时候指定这个配置文件便可。
  redis.conf的主要配置如下：我们开启了所有的集群配置
  
```
# 全地址监听,默认监听的是127.0.0.1地址
bind 0.0.0.0
# 服务端口,集群通信端口会在此基础之上加10000得到，比如6379和16379
port 6379
daemonize yes
pidfile /opt/redis-instance1/redis.pid
logfile "/opt/redis-instance1/logs/redis.log"
dir /opt/redis-instance1
# 如下和集群配置...
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
# 早期这个名称叫"cluster-slave-validity-factor"
cluster-replica-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage no
```
### 3.2 服务目录准备
  我们计划启动6个集群实例(3个master和3个slave)，每台机器启动2个实例。
  所以，我们在每一台机器都准备如下两个目录。
  
```
/opt/redis-instance1
/opt/redis-instance2
```
 redis.conf分别拷贝到以上两个目录，但因为同一台主机两个集群实例，因此需要把其中之一(比如redis-instance2)的redis.conf的端口进行修改避免重复。还要pidfile和logfile、dir等属性也需要调整，其他属性保持不变。
 
 ### 3.3 分别启动集群实例
   分别在每台机器上启动集群实例，可以参考如下脚本：
   
```
#!/bin/sh

REDIS_HOME=/opt/redis-5.0.4/src
$REDIS_HOME/bin/redis-server /opt/redis-instance1/redis.conf
$REDIS_HOME/bin/redis-server /opt/redis-instance2/redis.conf
~
```

 ### 3.4 组建集群
   我们启动了6个实例之后采用redis-cli的 "--cluster create"命令把他们组建为一个集群，只需要把所有的主机ip地址和端口设置上便可。其中"--cluster-replicas 1"表示集群实例会有一个slave，因此6个就会自动组建"3个master 3个slave"的集群。
```
./redis-cli --cluster create 192.168.32.115:6379 192.168.32.115:6380 192.168.32.116:6379 192.168.32.116:6380 192.168.32.117:6379 192.168.32.117:6380 --cluster-replicas 1
```
  命令执行后过一会儿会提示我们redis建议的集群形势(谁是master 谁是slave？),直接输入"yes"便可。
  
```
[root@rhel7 bin]# ./redis-cli --cluster create 192.168.32.115:6379 192.168.32.116:6379 192.168.32.117:6379 192.168.32.115:6380 192.168.32.116:6380 192.168.32.117:6380 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.32.116:6380 to 192.168.32.115:6379
Adding replica 192.168.32.117:6380 to 192.168.32.116:6379
Adding replica 192.168.32.115:6380 to 192.168.32.117:6379
M: 8a108ef8e9957f105d5b3dc8d49dbf1f6fe3d85c 192.168.32.115:6379
   slots:[0-16383] (5461 slots) master
M: aa3fd49c80346535e2a2757bf0bd82fad291d3c2 192.168.32.116:6379
   slots:[5461-10922] (5462 slots) master
M: 85481be88ea44cf55867263e4ee6adbf2549fd52 192.168.32.117:6379
   slots:[10923-16383] (5461 slots) master
S: 0aa379a50e1cf5bc0cfdb130c7540220b2687385 192.168.32.115:6380
   replicates 85481be88ea44cf55867263e4ee6adbf2549fd52
S: 17c455f2d3655ce906f40f9cebede22ee4aa675f 192.168.32.116:6380
   replicates 8a108ef8e9957f105d5b3dc8d49dbf1f6fe3d85c
S: 313ef5431a158e24554275b8d0e8acdd692e26db 192.168.32.117:6380
   replicates aa3fd49c80346535e2a2757bf0bd82fad291d3c2
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 192.168.32.115:6379)
M: 8a108ef8e9957f105d5b3dc8d49dbf1f6fe3d85c 192.168.32.115:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 313ef5431a158e24554275b8d0e8acdd692e26db 192.168.32.117:6380
   slots: (0 slots) slave
   replicates aa3fd49c80346535e2a2757bf0bd82fad291d3c2
S: 17c455f2d3655ce906f40f9cebede22ee4aa675f 192.168.32.116:6380
   slots: (0 slots) slave
   replicates 8a108ef8e9957f105d5b3dc8d49dbf1f6fe3d85c
M: 85481be88ea44cf55867263e4ee6adbf2549fd52 192.168.32.117:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: aa3fd49c80346535e2a2757bf0bd82fad291d3c2 192.168.32.116:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 0aa379a50e1cf5bc0cfdb130c7540220b2687385 192.168.32.115:6380
   slots: (0 slots) slave
   replicates 85481be88ea44cf55867263e4ee6adbf2549fd52
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
---------------------------
```

## 4 集群操作确认
### 4.1 redis客户端操作
  可以直接通过redis-cli连接集群，通过set新建key-value键值对，通过get获取值。
```
[root@rhel7 bin]# ./redis-cli -h 192.168.32.116 -p 6379
192.168.32.116:6379> set aaa "value-aaa"
OK
192.168.32.116:6379> get aaa
"value-aaa"
192.168.32.116:6379> 
```
### 4.2 jedis客户端
  jedis是官方推荐的java客户端，下载地址：https://redis.io/clients#java。
  
  如果是maven工程的话直接在pom.xml中增加客户端依赖便可，类似如下：
```
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>3.0.1</version>
      <type>jar</type>
      <scope>compile</scope>
    </dependency>
```

  测试Redis的代码简单参考如下。主要是==redis.clients.jedis.Jedis==，还可以直接调用clusterNodes()方法获取集群实例列表。

```
package jedis;

import redis.clients.jedis.Jedis;
public class JedisPing {
    public static void main(String[] args) {
        if(args.length < 2){
            System.out.println("Usage: java JedisPing $ip $port");
            System.exit(1);
        }
        int port = Integer.parseInt(args[1]);
        Jedis jedis = new Jedis(args[0], port);
        
        try {
            jedis.connect();
            System.out.println("Redis connect successed.");
        } catch (Throwable localThrowable3) {
            System.out.println("connect to redis faild");
        }finally{
            jedis.quit();
        }
    }
    
}
```
==注意==: 如果需要通过set设置redis集群中数据的话，需要使用"redis.clients.jedis.JedisCluster"客户端。

