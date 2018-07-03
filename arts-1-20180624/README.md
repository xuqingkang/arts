# **1.Algorithm**
I'm familar with Java, but planned to code with Python for my ARTS's A(Algorithm). Although it means i will spend more time than usual in the earlier stage, but it also means that maybe i will master the Python language.

**Question��**

Given an array of integers, return indices of the two numbers such that they add up to a specific target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.

**Example:**
```javascript
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```
**My Solution in Python3:**
```python
class Solution:
    def twoSum(self, nums, target):
        for i in range(0,len(nums)):
            for j in range(i + 1,len(nums)):
                if nums[i] + nums[j] == target:
                    return [i,j]
        return None
```

## **process**
I must share the process(three commit times until accepted) how i finished the algorithm question, because i spent more than one day for finishing it. It looks like a joke because it's so simple for most people, i hope i make great progress in the coming year.
![](https://pan.baidu.com/s/1skCkwImXOZyKpP_NMDzq1Q)

1. I use the "nums[$index] < target", but it's wrong because ignoring negative.
```python
class Solution:
    def twoSum(self, nums, target):
        for i in range(0,len(nums)):
            if nums[i] > target:
                continue
            for j in range(1,len(nums)):
                if nums[j] > target:
                    continue
                if nums[i] + nums[j] == target:
                    return [i,j]
        return None
```
2. the second occurence of "j" is wrong because it always start with "1"
```python
class Solution:
    def twoSum(self, nums, target):
        for i in range(0,len(nums)):
            if nums[i] > target:
                continue
            for j in range(1,len(nums)):
                if nums[j] > target:
                    continue
                if nums[i] + nums[j] == target:
                    return [i,j]
        return None
```
3. Finally i resolve it with lower efficiency. 
```python
class Solution:
    def twoSum(self, nums, target):
        for i in range(0,len(nums)):
            for j in range(i + 1,len(nums)):
                if nums[i] + nums[j] == target:
                    return [i,j]
        return None
```
# **2.Review**

����ȱ��ǿ��Ǵ��~_~

# **3.Tips**
����һ�����ɣ��ڽ��Java���ͻ���ڴ�й©������ʱ������Ҫ����ĳ�������ĸ�jar���У����ںܳ�һ��ʱ��������Ҫʹ�����µĽű���

**## �ҵ�ԭʼ����**

��������ű����÷�Ϊ��findclass.sh $��·�� $�����ơ�

**˼·Ϊ**������Ŀ¼������Ŀ¼���µ�jar����Ȼ��ʹ��jar -tvf������

����һ��**�����Ե�ȱ��**������ÿ������Ҫ����ô��һ���ļ�����Щ�ͻ��������ܴ��ļ���ʱ��ͨ����Ҫ�ֶ�������ű���
```java
#!/bin/sh

if [ $# != 2 ]; then
  echo "Usage:$0 \$rootPath \$ClassName"
  exit 2
fi

cl_root=$1
className=$2

function getJarFiles()
{
        fileList=`find $cl_root -name "*.jar"`;
        for fileName in $fileList;
        do
            if [ ! -r $fileName ]; then
               echo "Have no read permission for file:$fileName"
               continue;
            fi
            jar -tvf $fileName | grep $className
            if [ $? -eq 0 ]; then
               echo "Found $className in $fileName."
            fi
        done
}

getJarFiles
```
**## �򻯰汾**

������һ�죬һ��ͬ�¿�����Զ��ִ�У�˳���������һ�ѣ���������ļ����
�ŵ㣺ʵ����find�����Ľ���Ϳ���ֱ����grepȥ�����࣬���ҹܵ��ĺ�������������Ͽ������ҵ��Ľ��������ʾ
```shell
find . -name "*.jar" | xargs grep JavaMain | cut -f2 -d " " | xargs less | grep JavaMain --color
```

# **4.Share**
## �Ż�docker�����ļ���С
dockerĿǰ�Ѿ��ܻ����ˣ������Ƕ�Ӧ�ý������������еĵ�һ�����ǰ�Ӧ�ô���������У�ͨ�����ǿ�����Ҫ���һЩ��������(redhat\centos������ƽ̨������yum���а�װ)���м�����������ǵ�Ӧ�ÿ����������������п�����Ҫ���ļ�Ȩ�޽��и��ġ�
### **4.1 ������docker����ʱ���Ǻܿ��ܹ������µ�Ŀ¼�ṹ**
```shell
DOCKERFILE
dockerimages/oracle.libs.tar.gz
dockerimages/saleapp.war
dockerimages/tomcat.zip
dockerimages/deps/libaio-0.3.109-13.el7.x86_64.rpm
```
### **4.2 �Ż�ǰdockerfile**
```docker
# pull base image
# --------------
FROM 10.10.10.13:5000/gs.mobile.com/redhat7.2

# Maintainer
# --------------
MAINTAINER xuqingkang  <qingkang.xu@bessystem.com>

RUN echo 'root:root' | chpasswd

# Environment variables required for this build
# ---------------------------------------------
ENV APP_PKG=saleapp.war
ENV ORACLE_LIS_PKG=oracle.libs.tar.gz
ENV ORACLE_HOME_DIR=/app/oracle/product/12.1.0/db

ADD dockerimages/$ORACLE_LIS_PKG $ORACLE_HOME_DIR
ADD dockerimages/$APP_PKG /app/$APP_PKG

COPY dockerimages/deps/libaio-0.3.109-13.el7.x86_64.rpm /deps/libaio-0.3.109-13.el7.x86_64.rpm

# Setup filesystem and wjtpss user
#-----------------------------

RUN chmod a+xr -R /wjtpss && \
  useradd -d /wjtpss -s /bin/bash wjtpss && \
  echo wjtpss:wjtpss | chpasswd  && \
  chown wjtpss:wjtpss -R /wjtpss && \
  chmod -R 755 /usr/lib64 && \
  chmod -R 755 $ORACLE_HOME_DIR && \
  rpm -ivh /deps/libaio-0.3.109-13.el7.x86_64.rpm

USER wjtpss

WORKDIR $WJTPSS_HOME

ENTRYPOINT /bin/bash
```
���dockerfile�ǿ����������������ģ���ͨ��DOCKERFILE��ADD�����ѹoracle��������ͺ�Ӧ�ð�����������Ȼ��ʹ����chmod�޸��ļ�Ȩ�ޣ��Լ�ʹ��rpm��װaio����
��������⣺��Ϊdocker����ֲ㣬���Ұ���Copy On Write��˼·������Ҫ���ļ������޸��Ǿͻ���п�����������������Ǳ���oracle.libs.tar.gz�ļ�����Ƚϴ�ADD֮����ʹ��chmod�������Ȩ���޸�Ϊ755�����chmod�ᵼ��һ���ļ���������������������docker�����С��

## **4.3 �Ż��汾**

### �Ż�˼·�������ļ�������
docker�����ļ����Ż���������RUN ������ִ�������ļ������������ļ������Ŀ���������ADD��COPY�����˽���һ���ļ�����������RUN������ͨ��curl wget�������ȡ�ļ���֮����ļ����в�����
�������԰�YUMԴҲ����Ϊ�ļ��������������Ļ�dockerfile��RUN�����е�yum��װҲ�൱�ڴ��ļ���������ȡ�ļ���

### **4.3.1 �Ż���dockerfile**
```docker
# pull base image
# --------------
FROM 10.10.10.13:5000/gs.mobile.com/redhat7.2

# Maintainer
# --------------
MAINTAINER xuqingkang  <qingkang.xu@bessystem.com>

RUN echo 'root:root' | chpasswd

# Environment variables required for this build
# ---------------------------------------------
ENV FILE_SERVER_URL=http://10.251.50.113:8090/fileserver/dockerimages
ENV APP_PKG=saleapp.war
ENV ORACLE_LIS_PKG=oracle.libs.tar.gz
ENV ORACLE_HOME_DIR=/app/oracle/product/12.1.0/db

# Setup filesystem and wjtpss user
#-----------------------------

RUN curl -o /etc/yum.repos.d/rhel_fileserver.repo $FILE_SERVER_URL/rhel_fileserver.repo && \
  yum makecache && \
  yum -y install net-tools && \
  yum -y install telnet && \
  yum -y install glibc-common && \
  yum -y install iputils && \
  chmod a+xr -R /wjtpss && \
  useradd -d /wjtpss -s /bin/bash wjtpss && \
  chmod -R 755 /usr/lib64 && \
  curl -L $FILE_SERVER_URL/$ORACLE_LIS_PKG | tar zxf - && \
  chmod -R 755 $ORACLE_HOME_DIR && \
  curl -O $FILE_SERVER_URL/deps/libaio-0.3.109-13.el7.x86_64.rpm && \
  rpm -ivh /deps/libaio-0.3.109-13.el7.x86_64.rpm

USER wjtpss

WORKDIR $WJTPSS_HOME

ENTRYPOINT /bin/bash
```
### **4.3.2 �Ż����dockerfile�м���Ľ�**
1��û��ʹ��ADD����COPY�����ļ������������Ļ�����RUN��������chmod��������ļ����в���Ҳ���ᵼ�¶�һ���Ŀ���

2��������yumԴҲ���ļ���������ȡ��װ���������Ļ�yum install�������ֱ�����У�����Ҫ��ǰ������װ��(xxx.rpm)

3��ֱ��ͨ��curl��ȡ�ļ�����ѹ������Ȩ�ޱ��

4��ֱ��ͨ��curl��ȡrpm��֮����а�װ

�����źܶ��˶�������������ľ����������д�����⣬Ҳ������һ�ּ򻯵�˼·��

# **5.�ܽ�**
�������ҵ�ARTS,�������ŵ�Ӣ��д����Algorithmʵ��˼·��������һ���ܼ򵥵��㷨�⣡

ȱ��Review���µ�����Ӣ������ȷʵû�п����£�Ҳʵ�ڲ�������!  

������һ���д������ARTS������ܴ�ܴ�