This is 4th ARTS, The Algorithm is still difficult for me again and again.

# 1.Algorithm
### Question: 
Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

Example1:
```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```
Example2:
```
Input: "cbbd"
Output: "bb"
```

### My solution in python3 is:

In this solution, we process the string which less than 3 simply.
and it wasn't accepted until the 9th submission, it alse took me about more than ten hours.

We must process the even string and odd string and use the "type" variable to specify which way we should process the next character like "aa","aba" sequence and so on.
```
class Solution:
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """
        
        ''' It's neccessary to process the string which be empty or all space'''
        strLen = len(s)
        if strLen == 0:
            return ""
        if strLen == 1:
            return s
        elif strLen == 2:
            if s[0] == s[1]:
                return s
            else:
                return s[0]
        elif strLen == 3:
            if s[0] == s[2]:
                return s
            elif s[0] == s[1]:
                return s[0:2]
            elif s[1] == s[2]:
                return s[1:3]
        
        tmpDict = {'length':1,"range":(0,1)}
        isEvenSeq = (strLen % 2 == 0)

        maxLengthOfPalindromic = 1

        for i in range(1,strLen):
            lIndex = i - 1
            rIndex = i + 1
            '''
            type means the way we should process following characters in current 'i' loop. string sequence is lIndex->i->rIndex.
             0: default, espcilly means the first one loop;
             1: means that we should process the same triple-string like "aaa"; 
             2: means that we should process the standard palindromic string like "aba", in next loop we should also compare the lIndex and rIndex character orderly; 
             3: means that we should process only the rIndex character which same as 'i' character, eg:"bbbb" in "xsrdbbbb" when i is 3(sequence number of character 'd')
            '''
            type = 0 
            while lIndex >= 0 and rIndex < strLen:
                # process the median index when string length is even
                if (i+1) == strLen//2 and isEvenSeq and type == 0:
                    if s[rIndex] == s[i]:
                        tmpLenth = (rIndex - i) + 1
                        if tmpLenth > maxLengthOfPalindromic:
                            maxLengthOfPalindromic = tmpLenth
                            tmpDict = {'length':tmpLenth,"range":[i,rIndex + 1]}
                        type = 2
                        rIndex += 1
                    elif s[rIndex] == s[lIndex]:
                        type = 2
                    else:
                        break

                # process the conterminal same characters like the "bbb" in "cdbbbrt"
                elif s[lIndex] == s[i] and s[rIndex] == s[i] and (type == 0 or type == 1): 
                    if rIndex+1 < strLen and lIndex-1 > -1 and s[lIndex-1] == s[rIndex+1]:
                        type = 2
                    else:
                        type = 1
                        while  rIndex+1 < strLen:
                            if s[rIndex + 1] == s[i]:
                                rIndex += 1
                            else:
                                break
                        tmpLenth = (rIndex - lIndex) + 1
                        if tmpLenth > maxLengthOfPalindromic:
                            maxLengthOfPalindromic = tmpLenth
                            tmpDict = {'length':tmpLenth,"range":[lIndex,rIndex + 1]}
                        break

                # process the standard palindromic string like "aba"
                elif s[lIndex] == s[rIndex] and (type == 0 or type == 2):
                    type = 2
                    tmpLenth = (rIndex - lIndex) + 1
                    if tmpLenth > maxLengthOfPalindromic:
                        maxLengthOfPalindromic = tmpLenth
                        tmpDict = {'length':tmpLenth,"range":[lIndex,rIndex + 1]}
                
                    lIndex -= 1
                    rIndex += 1

                # process the conterminal same characters after current 'i' index
                elif s[rIndex] == s[i] and (type == 0 or type == 3):
                    if rIndex+1 < strLen and s[rIndex+1] == s[lIndex]:
                        rIndex += 1
                        type = 2
                    else:
                        type = 3
                        while  rIndex+1 < strLen:
                            if s[rIndex + 1] == s[i]:
                                rIndex += 1
                            else:
                                break
                        tmpLenth = (rIndex - i) + 1
                        if tmpLenth > maxLengthOfPalindromic:
                            maxLengthOfPalindromic = tmpLenth
                            tmpDict = {'length':tmpLenth,"range":[i,rIndex + 1]}
                        
                        break

                else: 
                    break
        return s[tmpDict["range"][0]:tmpDict["range"][1]]
```

# 2.Review
Today i read this article about better coding skills:
https://medium.com/@maladdinsayed/advanced-techniques-and-ideas-for-better-coding-skills-d632e9f9675

there are several useful advice, for example “Do not use magic numbers or magic strings”，"Do not use else-statements if you do not need to","Use and so on. meaningful names for your methods, variables and tests" and so on.

and the one of "Always use {} within if-statements in this article", this is mainly in language like Java, in other language it means we should Always keep our code clean with the readable format.

# 3.Tips
今天我们来说说开源的APM软件pinpoint的编译问题。
截止到今天(2018-07-21),pinpoint发布的稳定版本是v1.7.3。在研究pinpoint的时候可以直接下载官方编译好的发布版本，但我建议按照官方快速开始指南(http://naver.github.io/pinpoint/quickstart.html)自己编译运行。这样，如果有可能在本地修改一些代码以进行代码级别的研究。

对于编译的环境准备：
1. windows 10下安装git，并获取源码：git clone https://github.com/naver/pinpoint.git
2. 安装各个版本的JDK并设置JAVA_HOME/JAVA6_HOME/JAVA7_HOME/JAVA8_HOME/JAVA9_HOME几个环境变量，
3. 安装maven 3

直接编译则使用命令进行：./mvnw install -Dmaven.test.skip=true

然而，在编译的过程中经常发现类找不到的错误等，本次分享怎么解决编译过程中可能遇到的问题。
- 问题1：profile模块编译失败，错误日志如下:
```
Failed to execute goal on project pinpoint-profiler: Could not resolve dependencies for project com.navercorp.pinpoint:pinpoint-profiler:jar:1.8.0-SNAPSHOT: Could not find artifact com.navercorp.pinpoint:pinpoint-rpc:jar:tests:1.
8.0-SNAPSHOT in cloudera (https://repository.cloudera.com/artifactory/cloudera-repos/) -> [Help 1]
org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal on project pinpoint-profiler: Could not resolve dependencies for project com.navercorp.pinpoint:pinpoint-profiler:jar:1.8.0-SNAPSHOT: Could not find artifact
com.navercorp.pinpoint:pinpoint-rpc:jar:tests:1.8.0-SNAPSHOT in cloudera (https://repository.cloudera.com/artifactory/cloudera-repos/)
```
**解决方案**：

实际上出现这个编译失败的时候，发现我的maven本地仓库是存在pinpoint-rpc-1.8.0-SNAPSHOT-tests.jar的，所以我在编译中去掉了"-Dmaven.test.skip=true"选项，之后再次 “mvn install -Dmaven.test.skip=true”后这个模块的编译就成功。

- 问题2：然后又发现下web模块编译失败，具体的Debug错误日志如下:

```
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.0:install-node-and-npm (install node and npm) on project pinpoint-web: Execution install node and npm of goal com.github.eirslett:frontend-maven-plugin:1.0:install-node-and-npm failed: A required class was missing while executing com.github.eirslett:frontend-maven-plugin:1.0:install-node-and-npm: org/apache/commons/compress/compressors/gzip/GzipCompressorInputStream
[ERROR] -----------------------------------------------------
[ERROR] realm = plugin>com.github.eirslett:frontend-maven-plugin:1.0
[ERROR] strategy = org.codehaus.plexus.classworlds.strategy.SelfFirstStrategy
[ERROR] urls[0] = file:/E:/devInstaller/maven/repository/com/github/eirslett/frontend-maven-plugin/1.0/frontend-maven-plugin-1.0.jar
[ERROR] urls[1] = file:/E:/devInstaller/maven/repository/com/github/eirslett/frontend-plugin-core/1.0/frontend-plugin-core-1.0.jar
[ERROR] urls[2] = file:/E:/devInstaller/maven/repository/org/apache/maven/plugin-tools/maven-plugin-annotations/3.2/maven-plugin-annotations-3.2.jar
[ERROR] urls[3] = file:/E:/devInstaller/maven/repository/org/sonatype/plexus/plexus-build-api/0.0.7/plexus-build-api-0.0.7.jar
[ERROR] urls[4] = file:/E:/devInstaller/maven/repository/org/codehaus/plexus/plexus-utils/1.5.8/plexus-utils-1.5.8.jar
[ERROR] Number of foreign imports: 1
[ERROR] import: Entry[import from realm ClassRealm[maven.api, parent: null]]
```
**解决方案**：

跟第一个问题一样，实际上我查看了我本地的maven模块，org\apache\commons\commons-compress下有很多个版本，里边随意任何版本都是有GzipCompressorInputStream类的。因此我觉得是依赖库设置问题。
找到com\github\eirslett\frontend-maven-plugin\1.0\frontend-maven-plugin-1.0.pom文件，增加如下依赖后问题得到解决。感觉官方的frontend-maven-plugin居然没有对应的依赖，这点确实有点诡异。

```
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-plugin-api</artifactId>
        <version>3.1.0</version>
    </dependency>

    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-compress</artifactId>
        <version>1.10</version>
    </dependency>
```

在maven编译出现类库找不到类似的问题，首先先检查本地maven库是不是真的没有对应的模块和类，如果没有的话需要考虑镜像库地址。  如果有的话则需要考虑模块本身的pom配置文件依赖库是不是有遗漏。

# 4.Share
来谈谈zookeeper和大数据存储在pinpoint中的使用。Pinpoint是比较流行的开源APM（应用性能管理）产品，将结合APM产品pinpoint如何利用Zookeeper、Hbase、Hadoop(HDFS)等组建监控服务本身的集群。

Pinpoint采用了HBase进行监控大数据存储，本文在官网安装指导基础之上配置HBase使用部署与虚拟机之上的HDFS实现底层存储，并采用自搭建ZooKeeper作为达到数据的高可用。
## 4.1环境说明

如下图所示：业务主机192.168.32.1是安装了vmware的宿主机，其上运行pinpoint的collector和web组件，并运行一个测试用的tomcat。宿主机桑运行的三个虚拟机192.168.32.150~152主要部署ZooKeeper、Hbase、Hadoop。32.1主机上的监控数据存储于虚拟机的HBase，Zookeeper和HDFS也同HBase协同工作。
```
graph TD
D[业务主机:192.168.32.1] -->A
A[Master: rhel150-192.168.32.150 HBase/Hadoop master]
A --> B[Slave1: rhel151-192.168.32.151]
A --> C[Slave2: rhel152-192.168.32.152]
```
## 4.2配置说明
三台主机的hosts文件均采用如下配置，对各自的主机名和ip地址进行映射。

```
192.168.32.152 rhel152
192.168.32.150 rhel150
192.168.32.151 rhel151
```

### 4.2.1 Zookeeper
cat zookeeper/zookeeper-3.4.9/conf/zoo.cfg，每台主机的zoo.cfg配置均相同，但是需要把zookeeper-3.4.9/data/myid配置为不同的整数，如150、151、152分别用IP地址的最后三位作为唯一id保证不重复。
```
dataDir=/root/zookeeper/zookeeper-3.4.9/data
# the port at which the clients will connect
clientPort=2181
server.150=rhel150:2888:3888
server.151=rhel151:2888:3888
server.152=rhel152:2888:3888
```

### 4.2.2 Hadoop
也是三台主机均采用同样的配置，具体样例如下所示。

cat /root/hadoop/hadoop-2.8.4/etc/hadoop/core-site.xml,这个文件中配置HDFS的访问地址，如hdfs://rhel150:9000
```
<configuration>
<property>
   <name>hadoop.tmp.dir</name>
        <value>/root/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
   </property>
   <property>
        <name>fs.default.name</name>
        <value>hdfs://rhel150:9000</value>
   </property>
</configuration>
```

cat /root/hadoop/hadoop-2.8.4/etc/hadoop/hdfs-site.xml，配置HDFS中文件的复制数及目录等信息。
```

<configuration>

<property>
   <name>dfs.name.dir</name>
   <value>/root/hadoop/dfs/name</value>
   <description>Path on the local filesystem where theNameNode stores the namespace and transactions logs persistently.</description>
</property>
<property>
   <name>dfs.data.dir</name>
   <value>/root/hadoop/dfs/data</value>
   <description>Comma separated list of paths on the localfilesystem of a DataNode where it should store its blocks.</description>
</property>
<property>
   <name>dfs.replication</name>
   <value>2</value>
</property>
<property>
      <name>dfs.permissions</name>
      <value>false</value>
      <description>need not permissions</description>
</property>

</configuration>
```
cat /root/hadoop/hadoop-2.8.4/etc/hadoop/slaves，需要把需要启动datanode的主机名全加入到这个文件中。
```
rhel151
rhel152
```

配置好这两个文件后便可以通过hadoop目录sbin/start-dfs.sh启动HDFS

### 4.2.3 HBase
HBase也一样，三台主机采用相同的配置。
cat hbase-1.2.1/conf/hbase-env.sh。因为默认情况下HBase会启动内置的zookeeper，因为这里我们采用了自配置的zookeeper，所以需要把HBASE_MANAGES_ZK设置为false。需要注意如果操作系统没有设置JAVA_HOME环境变量的话需要在该文件中进行设置。
```
# Tell HBase whether it should manage it's own instance of Zookeeper or not.
# export HBASE_MANAGES_ZK=true
export HBASE_MANAGES_ZK=false
```

cat hbase-1.2.1/conf/hbase-site.xml，在这个文件中配置HDFS和zookeeper信息。
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://rhel150:9000/hbase</value>
  </property>

  <property>
    <name>hbase.tmp.dir</name>
    <value>/root/hbase-1.2.1/data/hbase/tmpdata</value>
  </property>
 
  <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
  </property>


    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>rhel150,rhel151,rhel152</value>
    </property>

  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/root/hbase-1.2.1/data/zookeeper</value>
  </property>
</configuration>
```
 cat hbase-1.2.1/conf/regionservers，类似Hadoop的slaves配置，需要启动regionserver的主机名均需要配置在这个文件中。
```
rhel150
rhel151
rhel152
```
以上配置好之后使用hbase的bin/start-hbase.sh便可以启动所有机器上的hbase，包括HMaster和RegionServer。

## 4.3 Pinpoint相关配置

### 4.3.1 pinpoint collector配置
pinpoint的collector.war部署到类如tomcat这样的中间件中后，直接修改WEB-INF/classes/hbase.properties文件
```
# example
hbase.client.host=192.168.32.150
hbase.client.port=2181
```
以及pinpoint-collector.properties

```
cluster.enable=true
cluster.zookeeper.address=192.168.32.150
cluster.zookeeper.sessiontimeout=30000
cluster.listen.ip=
cluster.listen.port=
```

### 4.3.2 pinpoint web配置
pinpoint的web.war部署后WEB-INF/classes/hbase.properties和pinpoint-web.properties的配置和collector的配置雷同，主要就是hbase.client.host和cluster.zookeeper.address两个选项。

## 4.4 总结思考
1，到此为止，采用zk、hadoop的hdfs、hbase作为存储的pinpoint服务集群就搭建完成了。实际上zookeeper目前作为分布式配置管理实现的主流方案在很多企业的生产环境都有使用。

2，在这里的部署架构之下，可以启动多个collector，实现collector的横向扩展，当我们监控的应用规模增大之后，可以使用多个pinpoint的collector实现数据处理。因为pinpoint所有采集数据的agent均从zk中查找可以处理数据的collector，从而把数据发给某个collector。实际上pinpoint的collector作为服务提供者，agent作为服务消费者，从zk中动态获取服务提供方的地址，微服务架构中的服务注册和发现也可以采用这样的架构实现。

3，目前的APM产品均把基于目前分析的所有数据进行存储，期望在基于监控大数据之上进行一定分析指导运维，HBase+HDFS的海量大数据存储框架被广泛采用。