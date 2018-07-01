This is my second ARTS, i feel lighter than the first time.
# **1.Algorithm**
**Question：**

Given an array of integers, return indices of the two numbers such that they add up to a specific target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.

**Example:**
```javascript
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```
**My Solution in Python3:**
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """

        '''Transfer all the input l1 and l2 to list'''
        l1List = self.node2List(l1)
        l2List = self.node2List(l2)
        l1List.reverse()
        l2List.reverse()

        minLen = min(len(l1List),len(l2List))
        """ ex: 9+8=17,so carryDigit is 1; 5+4=9 so it no carryDigit """
        carryDigit = 0
        count = 0
        sumList = []
        ''' Compute the aligned digit of two numbers, it's neccessary to handle remainder of the sum of two Numbers.'''
        while(count < minLen):
            a = l1List.pop()
            b = l2List.pop()
            tmpSum = a + b + carryDigit
            sumList.append(tmpSum % 10)
            carryDigit = 1 if tmpSum >= 10 else 0
            count += 1

        ''' if length of two number list is equal, there will be no remaining digit, so just return calculated list '''
        if len(l1List) != len(l2List):
            """ Process the remaining digit, it has no any compute at all, just appending the digit to list """
            remainDigitsList = l1List if len(l1List) else l2List
            while(len(remainDigitsList)):
                tmpSum = remainDigitsList.pop() + carryDigit
                sumList.append(tmpSum % 10)
                carryDigit = 1 if tmpSum >= 10 else 0
                
        if carryDigit:
            sumList.append(carryDigit)

        return sumList

    '''
    Transfer the "ListNode" to list, ex:
        input: 2 -> 3 -> 5- > 9
        output: [2,3,5,9]
    '''
    def node2List(self,myNode):
        tmpNode = myNode
        lList = []
        while(tmpNode != None):
            lList.append(tmpNode.val)
            if tmpNode.next != None:
                tmpNode = tmpNode.next
            else:
                tmpNode = None
        return lList
```

## **Notice**
In this problem, we should notice the following:

we must process the carry digit correctly,it's too easy to ignore adding to the result list.
the code as following is ignored at my first time submission.
```python
        if carryDigit:
            sumList.append(carryDigit)
        return sumList
```

# **2.Review**

本周缺，强烈谴责~_~

# **3.Tips**
关于在Docker环境下获取操作系统登录用户名的问题，getlogin()和getuid()比较。Docker基础镜像为：yjjy0921/redhat7.2

### **3.1 具体问题**
在迁移某个应用到Docker环境时发现因为调用操作系统的getlogin()方法(sys/types.h)获取不到登录用户调用导致strcpy函数的调用直接core了。
当时是从Redhat7.2宿主机迁移到运行Redhat7.2的Docker容器中。

出问题的代码如下：getlogin() 的调用会导致core
```c++
#include <cstdio>
#include <iostream>
#include <string.h>
#include <sys/types.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    std::cout << "Start" << std::endl;
    std::cout << "Start2,Next is getlogin() function test." << std::endl;
    std::cout << getlogin() << std::endl;
    std::cout << "Start3, Invoked getlogin() function." << std::endl;

    return 0;
}
```
### **3.2 原因分析**
 经查与/var/run/utmp文件有关，使用docker cp命令拷贝内网宿主机上的/var/run/utmp文件到容器对应的/var/run/utmp也不行，最后的解决方法是在外网找到一个redhat7.2，拷贝/var/run/utmp文件到镜像中得以解决。
但这不是最终解决方案，最后在stackoverflow上发现如果一定需要获取登录用户的话建议代码调整为通过getuid方法获取。getlogin()是一个不安全的方法，不推荐，详见：https://stackoverflow.com/questions/4785126/getlogin-c-function-returns-null-and-error-no-such-file-or-directory。

### **3.3 解决方案**
建议参照如下样例代码调整，采用getuid方法可以很有效的规避getlogin方法受/var/run/utmp文件影响，以及存在的安全性问题：
```python

#include <unistd.h>
#include <stdio.h>
#include <pwd.h>
#include <sys/types.h>
int main() {
   char *name;
   struct passwd *pass;
   pass = getpwuid(getuid());
   name = pass->pw_name;
   printf("This is the login name: %s\n", name);
   return 0;
}
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