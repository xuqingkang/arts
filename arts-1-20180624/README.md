# **1.Algorithm**
I'm familar with Java, but planned to code with Python for my ARTS's A(Algorithm). Although it means i will spend more time than usual in the earlier stage, but it also means that maybe i will master the Python language.

**Question：**

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

本周缺，强烈谴责~_~

# **3.Tips**
分享一个技巧，在解决Java类冲突、内存泄漏等问题时经常需要查找某个类在哪个jar包中，我在很长一段时间里总需要使用如下的脚本。

**## 我的原始方法**

其中这个脚本的用法为：findclass.sh $根路径 $类名称。

**思路为**：查找目录（含子目录）下的jar包，然后使用jar -tvf查找类

这有一个**很明显的缺点**，就是每次你总要用这么长一个文件，有些客户机器不能传文件的时候通常需要手动敲这个脚本。
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
**## 简化版本**

后来有一天，一个同事看到我远程执行，顺便鄙视了我一把，给出下面的简化命令。
优点：实际上find出来的结果就可以直接用grep去查找类，并且管道的后面三条命令组合可以让找到的结果高亮显示
```shell
find . -name "*.jar" | xargs grep JavaMain | cut -f2 -d " " | xargs less | grep JavaMain --color
```

# **4.Share**
## 优化docker镜像文件大小
docker目前已经很火热了，在我们对应用进行容器话运行的第一步便是把应用打包到镜像中，通常我们可能需要打包一些基础工具(redhat\centos这样的平台就是用yum进行安装)、中间件，最后把我们的应用拷贝进容器，还很有可能需要对文件权限进行更改。
### **4.1 在制作docker镜像时我们很可能构造如下的目录结构**
```shell
DOCKERFILE
dockerimages/oracle.libs.tar.gz
dockerimages/saleapp.war
dockerimages/tomcat.zip
dockerimages/deps/libaio-0.3.109-13.el7.x86_64.rpm
```
### **4.2 优化前dockerfile**
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
这个dockerfile是可以制作出镜像来的，其通过DOCKERFILE的ADD命令解压oracle的依赖库和和应用包到容器镜像，然后还使用了chmod修改文件权限，以及使用rpm安装aio包。
其存在问题：因为docker镜像分层，而且按照Copy On Write的思路，当需要对文件进行修改是就会进行拷贝。这带来的问题是比如oracle.libs.tar.gz文件本身比较大，ADD之后又使用chmod命令对其权限修改为755，这个chmod会导致一次文件拷贝，就无形中增加了docker镜像大小。

## **4.3 优化版本**

### 优化思路：构建文件服务器
docker镜像文件的优化，尽量在RUN 命令中执行所有文件操作，减少文件层多余的拷贝。少用ADD和COPY命令，因此建议搭建一个文件服务器，在RUN命令中通过curl wget等命令获取文件，之后对文件进行操作。
甚至可以把YUM源也配置为文件服务器，这样的话dockerfile中RUN命令中的yum安装也相当于从文件服务器获取文件。

### **4.3.1 优化后dockerfile**
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
### **4.3.2 优化后的dockerfile有几点改进**
1，没有使用ADD或者COPY进行文件拷贝，这样的话就算RUN命令中有chmod等命令对文件进行操作也不会导致多一级的拷贝

2，配置了yum源也从文件服务器获取安装包，这样的话yum install命令可以直接运行，不需要提前拷贝安装包(xxx.rpm)

3，直接通过curl获取文件并解压，进行权限变更

4，直接通过curl获取rpm包之后进行安装

我相信很多人都遇到过制作后的镜像会比想象中大的问题，也许这是一种简化的思路！

# **5.总结**
本期是我的ARTS,用了蹩脚的英语写完了Algorithm实现思路，而且是一个很简单的算法题！

缺了Review文章点评，英文这周确实没有看文章，也实在补不了了!  

我相信一年后写出来的ARTS会进步很大很大！