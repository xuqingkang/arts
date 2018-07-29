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

there are several useful advice, for example ��Do not use magic numbers or magic strings����"Do not use else-statements if you do not need to","Use and so on. meaningful names for your methods, variables and tests" and so on.

and the one of "Always use {} within if-statements in this article", this is mainly in language like Java, in other language it means we should Always keep our code clean with the readable format.

# 3.Tips
����������˵˵��Դ��APM����pinpoint�ı������⡣
��ֹ������(2018-07-21),pinpoint�������ȶ��汾��v1.7.3�����о�pinpoint��ʱ�����ֱ�����عٷ�����õķ����汾�����ҽ��鰴�չٷ����ٿ�ʼָ��(http://naver.github.io/pinpoint/quickstart.html)�Լ��������С�����������п����ڱ����޸�һЩ�����Խ��д��뼶����о���

���ڱ���Ļ���׼����
1. windows 10�°�װgit������ȡԴ�룺git clone https://github.com/naver/pinpoint.git
2. ��װ�����汾��JDK������JAVA_HOME/JAVA6_HOME/JAVA7_HOME/JAVA8_HOME/JAVA9_HOME��������������
3. ��װmaven 3

ֱ�ӱ�����ʹ��������У�./mvnw install -Dmaven.test.skip=true

Ȼ�����ڱ���Ĺ����о����������Ҳ����Ĵ���ȣ����η�����ô�����������п������������⡣
- ����1��profileģ�����ʧ�ܣ�������־����:
```
Failed to execute goal on project pinpoint-profiler: Could not resolve dependencies for project com.navercorp.pinpoint:pinpoint-profiler:jar:1.8.0-SNAPSHOT: Could not find artifact com.navercorp.pinpoint:pinpoint-rpc:jar:tests:1.
8.0-SNAPSHOT in cloudera (https://repository.cloudera.com/artifactory/cloudera-repos/) -> [Help 1]
org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal on project pinpoint-profiler: Could not resolve dependencies for project com.navercorp.pinpoint:pinpoint-profiler:jar:1.8.0-SNAPSHOT: Could not find artifact
com.navercorp.pinpoint:pinpoint-rpc:jar:tests:1.8.0-SNAPSHOT in cloudera (https://repository.cloudera.com/artifactory/cloudera-repos/)
```
**�������**��

ʵ���ϳ����������ʧ�ܵ�ʱ�򣬷����ҵ�maven���زֿ��Ǵ���pinpoint-rpc-1.8.0-SNAPSHOT-tests.jar�ģ��������ڱ�����ȥ����"-Dmaven.test.skip=true"ѡ�֮���ٴ� ��mvn install -Dmaven.test.skip=true�������ģ��ı���ͳɹ���

- ����2��Ȼ���ַ�����webģ�����ʧ�ܣ������Debug������־����:

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
**�������**��

����һ������һ����ʵ�����Ҳ鿴���ұ��ص�mavenģ�飬org\apache\commons\commons-compress���кܶ���汾����������κΰ汾������GzipCompressorInputStream��ġ�����Ҿ������������������⡣
�ҵ�com\github\eirslett\frontend-maven-plugin\1.0\frontend-maven-plugin-1.0.pom�ļ���������������������õ�������о��ٷ���frontend-maven-plugin��Ȼû�ж�Ӧ�����������ȷʵ�е���졣

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

��maven�����������Ҳ������Ƶ����⣬�����ȼ�鱾��maven���ǲ������û�ж�Ӧ��ģ����࣬���û�еĻ���Ҫ���Ǿ�����ַ��  ����еĻ�����Ҫ����ģ�鱾����pom�����ļ��������ǲ�������©��

# 4.Share
��̸̸zookeeper�ʹ����ݴ洢��pinpoint�е�ʹ�á�Pinpoint�ǱȽ����еĿ�ԴAPM��Ӧ�����ܹ�������Ʒ�������APM��Ʒpinpoint�������Zookeeper��Hbase��Hadoop(HDFS)���齨��ط������ļ�Ⱥ��

Pinpoint������HBase���м�ش����ݴ洢�������ڹ�����װָ������֮������HBaseʹ�ò����������֮�ϵ�HDFSʵ�ֵײ�洢���������ԴZooKeeper��Ϊ�ﵽ���ݵĸ߿��á�
## 4.1����˵��

����ͼ��ʾ��ҵ������192.168.32.1�ǰ�װ��vmware������������������pinpoint��collector��web�����������һ�������õ�tomcat��������ɣ���е����������192.168.32.150~152��Ҫ����ZooKeeper��Hbase��Hadoop��32.1�����ϵļ�����ݴ洢���������HBase��Zookeeper��HDFSҲͬHBaseЭͬ������
```
graph TD
D[ҵ������:192.168.32.1] -->A
A[Master: rhel150-192.168.32.150 HBase/Hadoop master]
A --> B[Slave1: rhel151-192.168.32.151]
A --> C[Slave2: rhel152-192.168.32.152]
```
## 4.2����˵��
��̨������hosts�ļ��������������ã��Ը��Ե���������ip��ַ����ӳ�䡣

```
192.168.32.152 rhel152
192.168.32.150 rhel150
192.168.32.151 rhel151
```

### 4.2.1 Zookeeper
cat zookeeper/zookeeper-3.4.9/conf/zoo.cfg��ÿ̨������zoo.cfg���þ���ͬ��������Ҫ��zookeeper-3.4.9/data/myid����Ϊ��ͬ����������150��151��152�ֱ���IP��ַ�������λ��ΪΨһid��֤���ظ���
```
dataDir=/root/zookeeper/zookeeper-3.4.9/data
# the port at which the clients will connect
clientPort=2181
server.150=rhel150:2888:3888
server.151=rhel151:2888:3888
server.152=rhel152:2888:3888
```

### 4.2.2 Hadoop
Ҳ����̨����������ͬ�������ã���������������ʾ��

cat /root/hadoop/hadoop-2.8.4/etc/hadoop/core-site.xml,����ļ�������HDFS�ķ��ʵ�ַ����hdfs://rhel150:9000
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

cat /root/hadoop/hadoop-2.8.4/etc/hadoop/hdfs-site.xml������HDFS���ļ��ĸ�������Ŀ¼����Ϣ��
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
cat /root/hadoop/hadoop-2.8.4/etc/hadoop/slaves����Ҫ����Ҫ����datanode��������ȫ���뵽����ļ��С�
```
rhel151
rhel152
```

���ú��������ļ�������ͨ��hadoopĿ¼sbin/start-dfs.sh����HDFS

### 4.2.3 HBase
HBaseҲһ������̨����������ͬ�����á�
cat hbase-1.2.1/conf/hbase-env.sh����ΪĬ�������HBase���������õ�zookeeper����Ϊ�������ǲ����������õ�zookeeper��������Ҫ��HBASE_MANAGES_ZK����Ϊfalse����Ҫע���������ϵͳû������JAVA_HOME���������Ļ���Ҫ�ڸ��ļ��н������á�
```
# Tell HBase whether it should manage it's own instance of Zookeeper or not.
# export HBASE_MANAGES_ZK=true
export HBASE_MANAGES_ZK=false
```

cat hbase-1.2.1/conf/hbase-site.xml��������ļ�������HDFS��zookeeper��Ϣ��
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
 cat hbase-1.2.1/conf/regionservers������Hadoop��slaves���ã���Ҫ����regionserver������������Ҫ����������ļ��С�
```
rhel150
rhel151
rhel152
```
�������ú�֮��ʹ��hbase��bin/start-hbase.sh������������л����ϵ�hbase������HMaster��RegionServer��

## 4.3 Pinpoint�������

### 4.3.1 pinpoint collector����
pinpoint��collector.war��������tomcat�������м���к�ֱ���޸�WEB-INF/classes/hbase.properties�ļ�
```
# example
hbase.client.host=192.168.32.150
hbase.client.port=2181
```
�Լ�pinpoint-collector.properties

```
cluster.enable=true
cluster.zookeeper.address=192.168.32.150
cluster.zookeeper.sessiontimeout=30000
cluster.listen.ip=
cluster.listen.port=
```

### 4.3.2 pinpoint web����
pinpoint��web.war�����WEB-INF/classes/hbase.properties��pinpoint-web.properties�����ú�collector��������ͬ����Ҫ����hbase.client.host��cluster.zookeeper.address����ѡ�

## 4.4 �ܽ�˼��
1������Ϊֹ������zk��hadoop��hdfs��hbase��Ϊ�洢��pinpoint����Ⱥ�ʹ����ˡ�ʵ����zookeeperĿǰ��Ϊ�ֲ�ʽ���ù���ʵ�ֵ����������ںܶ���ҵ��������������ʹ�á�

2��������Ĳ���ܹ�֮�£������������collector��ʵ��collector�ĺ�����չ�������Ǽ�ص�Ӧ�ù�ģ����֮�󣬿���ʹ�ö��pinpoint��collectorʵ�����ݴ�������Ϊpinpoint���вɼ����ݵ�agent����zk�в��ҿ��Դ������ݵ�collector���Ӷ������ݷ���ĳ��collector��ʵ����pinpoint��collector��Ϊ�����ṩ�ߣ�agent��Ϊ���������ߣ���zk�ж�̬��ȡ�����ṩ���ĵ�ַ��΢����ܹ��еķ���ע��ͷ���Ҳ���Բ��������ļܹ�ʵ�֡�

3��Ŀǰ��APM��Ʒ���ѻ���Ŀǰ�������������ݽ��д洢�������ڻ��ڼ�ش�����֮�Ͻ���һ������ָ����ά��HBase+HDFS�ĺ��������ݴ洢��ܱ��㷺���á�