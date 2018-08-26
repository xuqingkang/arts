In this ARTS, I introduces the Spark's installation,configration and program sample.

# 1.Algorithm
### Question: 
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.

```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```
For example, two is written as II in Roman numeral, just two one's added together. Twelve is written as, XII, which is simply X + II. The number twenty seven is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

I can be placed before V (5) and X (10) to make 4 and 9. 
X can be placed before L (50) and C (100) to make 40 and 90. 
C can be placed before D (500) and M (1000) to make 400 and 900.
Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

Example1

```
Input: "III"
Output: 3
```
Example2

```
Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```


### My solution in python3 is:

```
class Solution:
    N_ENUM = {"I":1,"V":5,"X":10,"L":50,"C":100,"D":500,"M":1000}
    def romanToInt(self, s):
        """
        :type s: str
        :rtype: int
        """
        strLen = len(s)
        if strLen == 0:
            return 0
        
        s = s[::-1] #reverse the string
        resultVal = self.N_ENUM[s[0]]
        preStrVal = self.N_ENUM[s[0]]

        i = 1
        while i < strLen:
            currentStrVal = self.N_ENUM[s[i]]
            if currentStrVal < preStrVal: # process the IV/IX,XL/XC,CD/CM's reverted string
                resultVal -= currentStrVal
            else:
                resultVal += currentStrVal
            preStrVal = currentStrVal
            i += 1
        
        return resultVal
        

def main():

    print(Solution().romanToInt("MCMXCIV"))

if __name__ == "__main__":
    main()
```
In this solution, I reverted the origin string for processing the IV/IX,XL/XC,CD/CM with easier way.

# 2.Review
// TODO: I will complete it later.

# 3.Tips
��ʱ��һ̨���������˺ܶ��Java���̣�������Ҫ��ȡÿ���������õ�HeapDumpPath·������ʱ������ps����awk��forѭ����������
1. ������ps�ҳ���ǰ�û�������java����
2. ʹ��forѭ��(����forѭ��������־λawk���õ�����¼��NF)��ͨ��substr($i,5,12)��ȡ��HeapDumpPath�ؼ����С���Ϊ���õ�HeapDumpPathΪ"-XX:HeapDumpPath=yyy"���������substr������Ҫ��5��ʼ
3. Ȼ����ͨ��һ�ֵȺŷָ���awk�������ҳ�HeapDumpPath���õ�����·����

�������£�
```
ps -ef|grep java|grep [whoami] |awk -F " " '{for (i=1;i<NF;i++) {if (substr($i,5,12)=="HeapDumpPath") print $i}}' | awk -F "=" '{print $2}'
```


# 4.Share(Spark ����˼��)

## 4.1 Spark��������
Spark�����Ľ�����˵����Spark��һ�����ٺͶ���;�ļ�Ⱥ����ϵͳ�����ṩ��Java��Scala��Python��R���ԵȵĿ����ӿڣ�ʹ����Щ���Խ���SparkӦ�õĿ������Ǻ����׵ģ�Spark�����˵ײ�����֮���ͨ�ź�ͬ�������⡣���һ�������Spark SQL���ڴ���SQL�ͽṹ�����ݴ�����MLlibģ�����ڻ���ѧϰ��GraphX����ͼ�δ����Լ�Spark������ȡ�

Spark��������Ӧ�����գ�

**Master**����Ϊ�������㼯Ⱥ�Ĺ����ˣ���Ϊ�ύ����Ľӿڣ���ͨ��web����Master����ϵͳ��Ϣ�鿴(����������Ⱥ�ļ���ڵ㡢ִ�������)

**Worker**������ڵ㣬����ִ�м������������

**executor**:spark�����ִ�е�Ԫ
**driver**��������������򣬿ɼ�����Ϊ��������ʹ��Java���Կ�������ʱ��main�࣬����Ҫ���������ڼ�Ⱥ�ڲ���ִ�С�

**application**:���Ǿ��������(���ݴ���ҵ���߼�)

**RDD(resilient distributed dataset)**:������Ϊ����Ҫ���������ݼ������ԴӸ����ļ�ϵͳ�д������ճ�����кܴ�Ĺ��������Ǵ���RDD��

spark�����ִ���߼�������driver��master����ִ�У�master��worker������Դ����worker����executorִ������������driver���ܴ�����

**��Ⱥ������(cluster manager)**:spark��Ⱥģʽ��Standalone(spark������Ⱥ�������)��Mesos��Hadoop YARN��Kubernetes���֡�����������Ҫ������װȻ������spark����һ����ֻ�谲װspark�������ɡ�

**�߿���**��Spark���Խ�����ZooKeeper����master�߿��������ã���֤Master��Standby Master�������ֵ�ʵ�����ϵ�����������������ǳ��

## 4.2 Spark��Ⱥ�

### 4.2.1 ���Ի���׼��
��192.168.32.150~152��̨��������spark��Ⱥ������spark2.3.1������ʾ˵������д��������Բ���Java����������ʹ��Spark�Դ���Standalone��Ⱥ��������������master��standby��Ҳ��ʹ��hadoop�����ļ��洢�����Ҫ��ʾ��ʵ��ȫ���ǲ���worker�����ı���Ŀ¼���ļ�������ṹ������ʾ��

```
--master(ͬʱҲ��Ϊworker�ڵ�)
  192.168.32.150
--slave(worker�ڵ�)
  192.168.32.150
  192.168.32.151
  192.168.32.152
```
ǰ���������ڰ�װ������֮ǰ��Ҫ��֤���л���JDK8+

### 4.2.2 Spark��Ⱥ�������
- ��װ����֮ǰ�������û������ܵ�¼(������ʹ�õ�root�û���ʵ�ʻ������ǲ�����ʹ��root�û���)
- ���أ�http://spark.apache.org/downloads.html,
- ��ѹspark: ��ÿ̨���������н�ѹ��ͳһĿ¼�������������ǽ�ѹ��/root/sparkĿ¼������spark homeĿ¼Ϊ/root/spark/spark-2.3.0-bin-hadoop2.7
- spark���ã�
1. spark-env.sh�ļ�������JAVA_HOME,����JAVA_HOME=/usr/java/jdk1.8.0_77
2. conf/slaves�ļ�������������Ҫ����worker������������ip��ַ���磺
```
rhel150
rhel151
rhel152
```
- spark��Ⱥ������ֱ������sbin/start-all.sh������master��slave(worker)��������
������ɺ�ͨ�����������master��8080�˿ھ��ܿ���������Ⱥ��ȫò���������е�worker�ڵ���Ϣ������ͼ��ʾ��

![image](../images/7-S-spark-console.png)

### 4.2 Spark ������ʵ�ֺͲ���

#### 4.2.1 ��������
���´�����Ҫʵ�ִ�ָ���ļ��в���ָ���Ĺؼ��֣���Ӧ��������

���´����ǳ�ʼ��SparkContext��
```
SparkConf conf = new SparkConf().setAppName(appName);
JavaSparkContext sparkContext = new JavaSparkContext(conf);
```
���´���Ϊִ�еĺ����߼��������д�������������Ҫ��Ϊ�������ļ����������ǵ��߼��Ƿ����ļ��ֱ��޸ĺ����½���һ��ͳ�ơ��ر�˵��һ�����Ƕ�ȡ���ļ�Ϊworker�����ϵı����ļ�����˺��ִ�е�ʱ����Ҫ���������ļ�����̨��������ͬĿ¼��Ϊ�����֣����ǰѲ����ļ��ĵ�һ�а����˵�ǰ�������ƣ�Ȼ����������Ϊ����SEVERE�ؼ��ֵ��С�
```
while ((System.currentTimeMillis() - startTime) < sleepTime) {
			if (logFile.lastModified() > fileLastModified) {
				System.out.println("###############file[" + logFile + " Has been changed, we will count again. Now is:" + dateFormat.format(new Date()));
				JavaRDD<String> logData = sparkContext.textFile(logFilePath);
				System.out.println("###############file[" + logFile + "'s first line is:[" + logData.first() + "]");
				System.out.println("###############Spark Application is executed at machine[" + System.getenv("HOSTNAME") + "]");
				long numAs = logData.filter(s -> s.contains(key)).count();
				System.out.println("###############Lines with " + key + ": " + numAs);
				fileLastModified = logFile.lastModified();
			}
```
����������������£�
```

/* SimpleApp.java */
import java.io.File;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

public class SimpleRDDApp {
	/*args[0]: appName
	* args[1]: logFilePath
	* args[2]: key, the key for finding from file
	* args[3]: sleep, this task execute time.
	*/
	private static int ARG_COUNT = 3;
	private static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSZ");
	public static void main(String[] args) {
		
		if(args.length < ARG_COUNT){
			System.exit(255);
		}
		
		String appName = args[0];
		String logFilePath = args[1];
		String key = args[2];
		
		// sleep time(unit:second)
		int sleepTime = 60000;
		if(args.length > 3){
			sleepTime = Integer.parseInt(args[3])*1000;
		}
		
		SparkConf conf = new SparkConf().setAppName(appName);
		JavaSparkContext sparkContext = new JavaSparkContext(conf);
		
		long startTime = System.currentTimeMillis();
		File logFile = new File(logFilePath);
		long fileLastModified = 0;
		while ((System.currentTimeMillis() - startTime) < sleepTime) {
			if (logFile.lastModified() > fileLastModified) {
				System.out.println("###############file[" + logFile + " Has been changed, we will count again. Now is:" + dateFormat.format(new Date()));
				JavaRDD<String> logData = sparkContext.textFile(logFilePath);
				System.out.println("###############file[" + logFile + "'s first line is:[" + logData.first() + "]");
				System.out.println("###############Spark Application is executed at machine[" + System.getenv("HOSTNAME") + "]");
				long numAs = logData.filter(s -> s.contains(key)).count();
				System.out.println("###############Lines with " + key + ": " + numAs);
				fileLastModified = logFile.lastModified();
			}

			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		sparkContext.close();
		sparkContext.stop();
	}
}
```

#### 4.2.2 ������������
���Բ����κ�һ����ϲ���Ĺ��߰�����������jar�ļ���
Ȼ��Ѵ�����jar������̨��������ͬĿ¼�����籾���д�����̨������/root/spark/appsĿ¼��Ȼ��ÿ̨������/root/spark/testfile.logΪ�����ļ�������������ʾ��

���е�һ�������ڸ��������в�𣬷ֱ�����������������
```
This file is in rhel150
####|2018-08-22 21:51:52.782|SEVERE|javax.enterprise.system.std.logging|_ThreadID=499;_ThreadName=pool-1-app-11;|java.lang.IllegalArgumentException: ���ݿ�����������Ϊ��!
####|2018-08-23 08:28:31.841|SEVERE|javax.enterprise.system.std
```
�������½ű������ύSpark����

```
#!/bin/bash

/root/spark/spark-2.3.0-bin-hadoop2.7/bin/spark-submit --class SimpleRDDApp --master spark://192.168.32.150:7077 --deploy-mode cluster /root/spark/apps/spark-0.0.1-SNAPSHOT.jar SimpleRDDApp /root/spark/testfile.log SEVERE 6000000
```
#### 4.2.3 �鿴ִ�н��
�����ύ�ɹ��󣬿���ͨ����������鿴�����񱻷��䵽��һ̨������ִ�У���������ͼ��ʾ��
![image](../images/7-S-spark-console.png)

����ͼ��ʾ�����ǿ��Կ������ǵ�driver����(driver-20180824210025-0008)���ձ����䵽150���������У�Ȼ��������150�������޸�/root/spark/testfile.log����������count������

ͨ����/root/spark/spark-2.3.0-bin-hadoop2.7/work/driver-20180824210025-0008/stdout�ļ��ļ�飬��������ִ�гɹ���
��������ͨ���ؼ��ּ���������־��ÿһ�������Ӧ���ļ��ǻ�������ģ���ʱ��RDD���ݶ�Ӧ150�����ϵ��ļ�����ʱ���Ӧ151��152�����ϵ��ļ�����˵��ÿ�������ִ�����ڼ�Ⱥ�ж�̬���ȵģ����Իῴ�������count���ϱ仯��

```
[root@rhel150 driver-20180824210025-0008]# cat stdout | grep "######"
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:00:28.059+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel150]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 94
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:03:11.291+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel150]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 95
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:03:40.236+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel152]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 100
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:05:02.505+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel152]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 93
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:05:27.955+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel151]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 93
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:05:34.241+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel151]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 93
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:06:58.788+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel152]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 93
###############file[/root/spark/testfile.log Has been changed, we will count again. Now is:2018-08-24T21:07:04.960+0800
###############file[/root/spark/testfile.log's first line is:[This file is in rhel150]
###############Spark Application is executed at machine[rhel150]
###############Lines with SEVERE: 106
```


## 4.3 ����Spark�ļ���˼��
1. ����ZooKeeperʵ��Spark��master�߿��÷����������������Ƽ���
2. ������������һ���HDFS����HBase��HIVE�ȷֲ�ʽ�洢�л�ȡ����Դ���з�������
3. ����Kerbernetes���������в���Kubernetes��Ϊ��Ⱥ���ȹ���Ӧ��������
4. Spark2.0��ʹ��Datasetȡ����RDD��RDD���APIĿǰ���Ǻ��Ƽ�ʹ���ˡ�����Spark��������Storm������������ʲô�д��ں�߼����о���
5. ��һƪARTS���ᵽ��Storm��Spark�����𣬸��Ե����ó����д������о���
