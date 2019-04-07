# ***Redis
  Redis�ǹ㷺��ʹ�õ��ڴ����ݿ��ˣ���ͨ���������������ݿ⣬��Ҳ���������ݿ��������Ϣ����(message broker)�ġ����Ǽ��ö���ǽ�WebӦ�õ�Session��Ϣ�洢��Redis��ʵ�ֻỰ����״̬�����졣
  
  �������ǻ������°汾��Redis(5.0.4)������װ�ͼ�Ⱥ���ã����ǻ��ǶԲ��Ի�������˵����192.168.32.115��116��117��̨������

## 1����װǰ׼��
  ��Ҫʵ�ְ�װgcc��������û�еĻ���ǰ��װ��ɡ�

## 2����װ˵��
  ֱ����redis�������أ�https://redis.io/download��
  
  ���غ�ֱ�ӽ�ѹredis-5.0.4.tar.gz,Ȼ�����jemalloc�Ϳ��Ա���redis�ˡ�
###  2.1 ����jemalloc
  ����depsĿ¼���б��룬�������£�
  
```
[root@rhel7 redis-5.0.4] cd deps
[root@rhel7 deps] make jemalloc
```
  ���������jemallocֱ�ӱ���redis�Ļ��ͻ���������Ĵ���
  
```
�Ҳ���jemalloc.h��
cc: ����../deps/hiredis/libhiredis.a��û���Ǹ��ļ���Ŀ¼
cc: ����../deps/lua/src/liblua.a��û���Ǹ��ļ���Ŀ¼
```

  redis�ٷ�˵���ĵ���jemalloc���������͵ģ�

```
Selecting a non-default memory allocator when building Redis is done by setting
the `MALLOC` environment variable. Redis is compiled and linked against libc
malloc by default, with the exception of jemalloc being the default on Linux
systems. This default was picked because jemalloc has proven to have fewer
fragmentation problems than libc malloc
```
### 2.2 ����redis
  �ص�redis��Ŀ¼ֱ��make����
  
```
[root@rhel7 deps] cd ../
[root@rhel7 redis-5.0.4] make
```
### 2.3 ��װȷ�ϺͶ����ƿ���
  ����ɹ�����srcĿ¼�������¼����������ļ���==redis-server==Ϊ����ˣ�redis-cli�������й��ߡ�
```
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-check-aof
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-check-rdb
-rwxr-xr-x. 1 root root 4365440 4?   4 22:52 redis-benchmark
-rwxr-xr-x. 1 root root 4805784 4?   4 22:52 redis-cli
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-sentinel
-rwxr-xr-x. 1 root root 8100855 4?   4 22:52 redis-server
```
## 3����Ⱥ����
### 3.1 �����ļ�׼��
  redis����Ҫ�����ļ�����==redis.conf==�ļ����޸ĺ�����ļ���Ȼ������redis-server��ʱ��ָ����������ļ���ɡ�
  redis.conf����Ҫ�������£����ǿ��������еļ�Ⱥ����
  
```
# ȫ��ַ����,Ĭ�ϼ�������127.0.0.1��ַ
bind 0.0.0.0
# ����˿�,��Ⱥͨ�Ŷ˿ڻ��ڴ˻���֮�ϼ�10000�õ�������6379��16379
port 6379
daemonize yes
pidfile /opt/redis-instance1/redis.pid
logfile "/opt/redis-instance1/logs/redis.log"
dir /opt/redis-instance1
# ���ºͼ�Ⱥ����...
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
# ����������ƽ�"cluster-slave-validity-factor"
cluster-replica-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage no
```
### 3.2 ����Ŀ¼׼��
  ���Ǽƻ�����6����Ⱥʵ��(3��master��3��slave)��ÿ̨��������2��ʵ����
  ���ԣ�������ÿһ̨������׼����������Ŀ¼��
  
```
/opt/redis-instance1
/opt/redis-instance2
```
 redis.conf�ֱ𿽱�����������Ŀ¼������Ϊͬһ̨����������Ⱥʵ���������Ҫ������֮һ(����redis-instance2)��redis.conf�Ķ˿ڽ����޸ı����ظ�����Ҫpidfile��logfile��dir������Ҳ��Ҫ�������������Ա��ֲ��䡣
 
 ### 3.3 �ֱ�������Ⱥʵ��
   �ֱ���ÿ̨������������Ⱥʵ�������Բο����½ű���
   
```
#!/bin/sh

REDIS_HOME=/opt/redis-5.0.4/src
$REDIS_HOME/bin/redis-server /opt/redis-instance1/redis.conf
$REDIS_HOME/bin/redis-server /opt/redis-instance2/redis.conf
~
```

 ### 3.4 �齨��Ⱥ
   ����������6��ʵ��֮�����redis-cli�� "--cluster create"����������齨Ϊһ����Ⱥ��ֻ��Ҫ�����е�����ip��ַ�Ͷ˿������ϱ�ɡ�����"--cluster-replicas 1"��ʾ��Ⱥʵ������һ��slave�����6���ͻ��Զ��齨"3��master 3��slave"�ļ�Ⱥ��
```
./redis-cli --cluster create 192.168.32.115:6379 192.168.32.115:6380 192.168.32.116:6379 192.168.32.116:6380 192.168.32.117:6379 192.168.32.117:6380 --cluster-replicas 1
```
  ����ִ�к��һ�������ʾ����redis����ļ�Ⱥ����(˭��master ˭��slave��),ֱ������"yes"��ɡ�
  
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

## 4 ��Ⱥ����ȷ��
### 4.1 redis�ͻ��˲���
  ����ֱ��ͨ��redis-cli���Ӽ�Ⱥ��ͨ��set�½�key-value��ֵ�ԣ�ͨ��get��ȡֵ��
```
[root@rhel7 bin]# ./redis-cli -h 192.168.32.116 -p 6379
192.168.32.116:6379> set aaa "value-aaa"
OK
192.168.32.116:6379> get aaa
"value-aaa"
192.168.32.116:6379> 
```
### 4.2 jedis�ͻ���
  jedis�ǹٷ��Ƽ���java�ͻ��ˣ����ص�ַ��https://redis.io/clients#java��
  
  �����maven���̵Ļ�ֱ����pom.xml�����ӿͻ���������ɣ��������£�
```
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>3.0.1</version>
      <type>jar</type>
      <scope>compile</scope>
    </dependency>
```

  ����Redis�Ĵ���򵥲ο����¡���Ҫ��==redis.clients.jedis.Jedis==��������ֱ�ӵ���clusterNodes()������ȡ��Ⱥʵ���б���

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
==ע��==: �����Ҫͨ��set����redis��Ⱥ�����ݵĻ�����Ҫʹ��"redis.clients.jedis.JedisCluster"�ͻ��ˡ�
