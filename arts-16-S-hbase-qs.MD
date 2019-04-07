# ***HBase
  HBase��Hadoop��̬Ȧ�ֲ�ʽ�����ݴ洢���ݿ⣬���չٷ����Ƶ�Ŀ�꣺HBase��Ŀ�����йܷǳ�������ݱ�������ģΪ��ǧ�������������ڼ������������ͬBigTableʹ�ùȸ�GFS�ļ�ϵͳһ����HBaseʹ��Hadoop��HDFS��Ϊ�ײ�洢��
  
  ���¸���HBase2.1.4�汾��������˵����ͬ��ʹ��192.168.32.150~152��̨�����齨HBase��Ⱥ��32.150��Ϊmaster��32.151��32.152��Ϊregionserver��

## 1����װ˵��
### 1.1,��װǰ׼��
  **JDK׼��**��HBase 1.x�汾��ҪJDK1.7�汾���ϣ���HBase 2.x�汾����ҪJDK1.8�汾���ϣ������Ҫ��������װ���ʵ�JDK�汾��
  
  **ZooKeeper׼��**��hbase����ʱ�����������õ�ZooKeeper��������Ҳ�������Ȱ�װ������ZooKeeper����HBaseʹ���������е�ZooKeeper��2.1.4�汾��ҪZookeeper�汾Ϊ3.4.x��
  
  **HDFS**���ڼ�Ⱥ��֤�У����ǲ���HDFS��Ϊ�ײ�洢�� ��Ȼ��Ҳ����ʹ�ñ����ļ�ϵͳ��
  
  �ӹ���https://hbase.apache.org���Ժ����׽����������ӡ�
  
  ����hbase-2.1.4-bin.tar.gz��ֱ�Ӷ�����н�ѹ����ѹ���hbase-2.1.4��ΪHBase��Ŀ¼���������
```
$ tar -zxvf hbase-2.1.4-bin.tar.gz
$ cd cd hbase-2.1.4/
```

## 2������
### 2.1��������������
  �޸�conf/hbase-env.sh�ļ�����ȷ����JDK·����
  
  ���⣬HBASE_MANAGES_ZKĬ��ֻΪtrue(Ҳ����HBase����ʱ����ZK)����Ϊ����������ZK��������������Ϊfalse(ʹ���ⲿ�Ѿ�������ZK)��
```
vi conf/hbase-env.sh 

# The java implementation to use.  Java 1.8+ required.
# export JAVA_HOME=/usr/java/jdk1.8.0/
export JAVA_HOME=/usr/java/jdk1.8.0_77

# Tell HBase whether it should manage it's own instance of ZooKeeper or not.
# export HBASE_MANAGES_ZK=true
export HBASE_MANAGES_ZK=false
```

### 2.2������������conf/hbase-site.xml
  ���£���Ҫ������HDFS�ĵ�ַ������ZK��Ӧ�������Ͷ˿ڡ�
  ������"hbase.cluster.distributed"Ϊtrue���������������Ҫ��������hbase������ʱ���Զ�����regionserver��������ֻ����master��
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://rhel150:9000/hbase</value>
  </property>

  <property>
    <name>hbase.tmp.dir</name>
    <value>/root/hbase-2.1.4/data/hbase/tmpdata</value>
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

</configuration>
```
### 2.3������
  ��32.150(master)����������bin/start-hbase.sh�ű�������master��regionserver��
  �״��������ܳ������´�����ҪΪhtrace��jar��Ĭ����hbase��libĿ¼û�У���Ҫ��hadoop1.x�汾���ҵ�htrace��3.x�汾(����htrace-core-3.1.0-incubating.jar)��ע�ⲻ����4.x�汾��
```
2019-03-28 21:42:18,991 ERROR [main] regionserver.HRegionServer: Failed construction RegionServer
java.lang.NoClassDefFoundError: org/apache/htrace/SamplerBuilder
        at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:644)
        at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:628)
        at org.apache.hadoop.hdfs.DistributedFileSystem.initialize(DistributedFileSystem.java:149)
        at org.apache.hadoop.fs.FileSystem.createFileSystem(FileSystem.java:2667)
        at org.apache.hadoop.fs.FileSystem.access$200(FileSystem.java:93)
        at org.apache.hadoop.fs.FileSystem$Cache.getInternal(FileSystem.java:2701)
        at org.apache.hadoop.fs.FileSystem$Cache.get(FileSystem.java:2683)
        at org.apache.hadoop.fs.FileSystem.get(FileSystem.java:372)
        at org.apache.hadoop.fs.Path.getFileSystem(Path.java:295)
        at org.apache.hadoop.hbase.util.CommonFSUtils.getRootDir(CommonFSUtils.java:362)
        at org.apache.hadoop.hbase.util.CommonFSUtils.isValidWALRootDir(CommonFSUtils.java:411)
        at org.apache.hadoop.hbase.util.CommonFSUtils.getWALRootDir(CommonFSUtils.java:387)
        at org.apache.hadoop.hbase.regionserver.HRegionServer.initializeFileSystem(HRegionServer.java:704)
        at org.apache.hadoop.hbase.regionserver.HRegionServer.<init>(HRegionServer.java:613)
        at org.apache.hadoop.hbase.master.HMaster.<init>(HMaster.java:489)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at org.apache.hadoop.hbase.master.HMaster.constructMaster(HMaster.java:3093)
        at org.apache.hadoop.hbase.master.HMasterCommandLine.startMaster(HMasterCommandLine.java:236)
        at org.apache.hadoop.hbase.master.HMasterCommandLine.run(HMasterCommandLine.java:140)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
        at org.apache.hadoop.hbase.util.ServerCommandLine.doMain(ServerCommandLine.java:149)
        at org.apache.hadoop.hbase.master.HMaster.main(HMaster.java:3111)
Caused by: java.lang.ClassNotFoundException: org.apache.htrace.SamplerBuilder
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
```

### 2.4,ȷ��
�����ɹ���ͨ������master������16010�˿ڿ��Կ���������ͼ����ʾ���е�regionserver��

![image](./images/arts-16-S-HBase-ok.png)

hbase shell���ԣ�ͨ��"hbase shell"����hbase shell�����С�ʹ��create��describe���Դ������Ͳ鿴����Ϣ��


```
hbase(main):004:0> create 'table1','fa1'
Created table table1
Took 1.4340 seconds                                                                                                  
=> Hbase::Table - table1

hbase(main):002:0> describe 'table1'
Table table1 is ENABLED                                                                                              
table1                                                                                                               
COLUMN FAMILIES DESCRIPTION                                                                                          
{NAME => 'fa1', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELL
S => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', R
EPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON
_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '655
36'}                                                                                                                 
1 row(s)
Took 0.1227 seconds
```