In this ARTS, I introduces the Spark's installation,configration and program sample.

# 1.Algorithm
### Question: 



### My solution in python3 is:


# 2.Review
// TODO: I will complete it later.

# 3.Tips



# 4.Share(Hadoop HDFS ����˼��)

## 4.1 Hadoop��������
### 4.1.1 HDFS
  ***Hadoop�ṩ�ķֲ�ʽ�ļ�ϵͳ��HDFS��Ⱥ��NameNode��DataNode��ɣ�NameNode��Ӧ����������һ��master server��������ļ�ϵͳ�����ռ䣬DataNode����������ļ��Ĵ洢��
  ***HDFSÿ���ļ����ᱻ�ֿ�洢��Ҳ����Block�����С�ǿ������õġ�Ϊ������ļ�ϵͳ�ɿ��ԣ�HDFS���ļ�����ڶ�̨�����ϱ��棬�鸴����(replicas)�ǿ������õġ�
### 4.1.2 MapReduce
 Ϊ�˲��д����������ݵ�һ����Ⱥ��ܡ�
 MapReduce Job����������ݷ�Ϊ���chunk��Ȼ����Reduce Task�������ݴ�����
 MapReduce��Ⱥ��һ��Resource Manager����Ⱥ���������ϣ��Ͷ��NodeManager����Ⱥҵ�������ϣ����
### 4.1.3 Yarn
  Yarn��Ϊ��һ����MapReduce��ܣ�����˼���ǰ���Դ������Resouce Manager����������ȼ�أ�Job Scheduling/monitoring���ֿ����ɲ�ͬ��Daemon������ɡ�
  Yarn��Ҳ��ResourceManager��NodeManager������Resource Manager��Scheduler��ApplicationManager���������Scheduler��Ҫ����ַ�����Դ��Ӧ�ã����������Ӧ�õ�״̬��أ�ApplicationManager���������ύ����Э���õ�һ�������������������ApplicationMaster��ApplicationMaster��������״̬�ĸ��١�
  
  ����ΪYarn�ļܹ�ͼ:
  ![image](../images/8-S-hadoop-yarn-architecture.gif)
������Ҫ��HDFS��ʹ��

## 4.2 Hadoop��Ⱥ�

### 4.2.1 ���Ի���׼��
��192.168.32.150~152��̨��������Hadoop��Ⱥ������ṹ������ʾ��

```
--namenode
  192.168.32.150
--datanode
  192.168.32.151
  192.168.32.152
```
ǰ��������
1. �ڰ�װ������֮ǰ��Ҫ��֤���л���JDK8+
2. ������֮������SSH���ܵ�¼

### 4.2.2 Hadoop��Ⱥ�������
- ��װ����֮ǰ�������û������ܵ�¼(������ʹ�õ�root�û���ʵ�ʻ������ǲ�����ʹ��root�û���)
- ���أ�https://hadoop.apache.org/releases.html,
- ��ѹhadoop: ��ÿ̨���������н�ѹ��ͳһĿ¼�������������ǽ�ѹ��/root/hadoopĿ¼������hadoop homeĿ¼Ϊ/root/hadoop/hadoop-2.8.4
- hadoop����(ȫ����master������Ҳ�������ǵ�namenode��resource manager�����Ͻ���)��
1. etc/hadoop/hadoop-env.sh�ļ�������JAVA_HOME,����"export JAVA_HOME=/usr/java/jdk1.8.0_77"
2. etc/hadoop/core-site.xml�ļ�������FS��namenode URI�ȣ��磺
```
<configuration>
<property>
   <name>hadoop.tmp.dir</name>
        <value>/root/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
   </property>
   <property>
        <name>fs.defaultFS</name>
        <value>hdfs://rhel150:9000</value>
   </property>
</configuration>
```
����fs.defaultFS��Ϊnamenode��URI�������ڰ汾��������Ϊfs.default.name

3. etc/hadoop/hdfs-site.xml�ļ�������HDFS�ĸ�������Ŀ¼�ȣ��磺
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

4.etc/hadoop/yarn-site.xml��Ҫ����Resource Manager����Ϣ���磺
```
<configuration>

<!-- Site specific YARN configuration properties -->

<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>rhel150</value>
   </property>
   <property>
        <description>The address of the applications manager interface in the RM.</description>
        <name>yarn.resourcemanager.address</name>
        <value>${yarn.resourcemanager.hostname}:8032</value>
   </property>
   <property>
        <description>The address of the scheduler interface.</description>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>${yarn.resourcemanager.hostname}:8030</value>
   </property>
   <property>
        <description>The http address of the RM web application.</description>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>${yarn.resourcemanager.hostname}:8088</value>
   </property>
   <property>
        <description>The https adddress of the RM web application.</description>
        <name>yarn.resourcemanager.webapp.https.address</name>
        <value>${yarn.resourcemanager.hostname}:8090</value>
   </property>
   <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>${yarn.resourcemanager.hostname}:8031</value>
   </property>
   <property>
        <description>The address of the RM admin interface.</description>
        <name>yarn.resourcemanager.admin.address</name>
        <value>${yarn.resourcemanager.hostname}:8033</value>
   </property>
   <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
   </property>
   <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>1024</value>
        <discription>ÿ���ڵ�����ڴ�,��λMB,Ĭ��8182MB</discription>
   </property>
   <property>
        <name>yarn.nodemanager.vmem-pmem-ratio</name>
        <value>2.1</value>
   </property>
   <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>1024</value>
</property>
   <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>

</configuration>
```
5.etc/hadoop/mapred-site.xml��Ҫ��������mapreduce����֮�����Ϣ��������ʾ�������µ���Դ���ȿ��yarn��Ϊmapreduce�����棺
```
<configuration>

 <property>
    <name>mapred.job.tracker</name>
    <value>rhel150:49001</value>
</property>
<property>
      <name>mapred.local.dir</name>
       <value>/root/hadoop/var</value>
</property>
<property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
</property>

</configuration>
```
6. etc/hadoop/slaves�ļ���������������̨������Ϊslave����
```
rhel151
rhel152
```

- hadoop��Ⱥ������
1. ��namenode����(192.168.32.150)��/root/hadoop/hadoop-2.8.4/binĿ¼��ִ�г�ʼ���ű���Ҳ����ִ�����
 ./hadoop  namenode  -format

2.ֱ������/root/hadoop/hadoop-2.8.4/sbin/start-all.sh������master��slave(datanode)��������
������ɺ�ͨ�����������master��50070�˿ھ��ܿ���������Ⱥ��ȫò���������е�datanode�ڵ���Ϣ������ͼ��ʾ��

![image](../images/8-S-hadoop-cluster-overview.png)


### 4.2 Hadoop HDFS��������
����hadoop�󣬱����ʹ��hadoop��fsģ������ļ�������
***����HDFS�ļ��� ./hadoop fs -mkdir /test

***��HDFS�ļ���
[root@rhel150 bin]# ./hadoop fs -fs hdfs://rhel150:9000 -ls /
Found 1 items
drwxr-xr-x   - root supergroup          0 2018-07-22 17:55 /test

***���������ļ���HDFS
./hadoop fs -fs hdfs://rhel150:9000 -copyFromLocal /root/hadoop/hadoop-2.8.4/etc/hadoop/core-site.xml /test/core-site.xml

�½����ļ��к��ļ���������������ϲ鿴���ļ�Ŀ¼�ṹ
![image](../images/8-S-hadoop-hdfs-fileview.png)

## 4.3 ����Hadoop�ļ���˼��
1. ����ZooKeeperʵ��Spark��master�߿��÷����������������Ƽ���
2. ������������һ���HDFS����HBase��HIVE�ȷֲ�ʽ�洢�л�ȡ����Դ���з�������
3. ����Kerbernetes���������в���Kubernetes��Ϊ��Ⱥ���ȹ���Ӧ��������
4. Spark2.0��ʹ��Datasetȡ����RDD��RDD���APIĿǰ���Ǻ��Ƽ�ʹ���ˡ�����Spark��������Storm������������ʲô�д��ں�߼����о���
5. ��һƪARTS���ᵽ��Storm��Spark�����𣬸��Ե����ó����д������о���
