This is 3rd ARTS, The Algorithm is still difficult for me.
# **1.Algorithm**
**Question��**

Given a string, find the length of the longest substring without repeating characters.

**Example:**
```javascript
Given "abcabcbb", the answer is "abc", which the length is 3.
Given "bbbbb", the answer is "b", with the length of 1.
Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

**My Solution in Python3:**
```python
class Solution:
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        
        ''' It's neccessary to process the string which be empty or all space'''
        if len(s) == 0:
            return 0
        elif s.isspace():
            return 0
            
        strLen = len(s)
        '''tmpDict is the intermiation result, it story the length and the index of substring temporary'''
        tmpDict = {'length':1,"range":(0,1)}

        for i in range(0,strLen):
            for j in range(i+1,strLen):
                '''d
                If found the duplicated string or j was reached the biggest index, we should check the tmpDict.
                '''
                if s[j] in s[i:j]:
                    tmpLen = j - i
                    if tmpLen > tmpDict["length"]:
                        tmpDict = {'length':tmpLen,"range":(i,j)}
                    i = s.index(s[j],i,j) + 1
                    break
                elif j == strLen-1:
                    tmpLen = (j - i) + 1
                    if tmpLen > tmpDict["length"]:
                        tmpDict = {'length':tmpLen,"range":(i,j)}

        return tmpDict["length"]

```

## **Notice**
As an engineer which has decade-long experience with coding, i made the following mistake unexpectedly.

1,I neglectted processing the empty string unexpectedly at the first submission, so it will wrongly return 1 when the empty string as the input.

2,If the input string is wholly substring which has no any duplicated character, i return the number which be less than the string's length. eg: return the 1 when input string is "au".
the following code is neccessary
```python
                elif j == strLen-1:
                    tmpLen = (j - i) + 1
                    if tmpLen > tmpDict["length"]:
                        tmpDict = {'length':tmpLen,"range":(i,j)}
```

when i committed the right code to the leetcode, i found it's very inefficient. Many knowledge of algorithm is waiting for me...

# **2.Review**

I read this article about "the monitor metric data storing", The origin url is:https://blog.timescale.com/sql-nosql-data-storage-for-prometheus-devops-monitoring-postgresql-timescaledb-time-series-3cde27fd1e07

As metioned in this article, prometheus is the internal monitoring system Google developed for Borg, and along with the popularity of Kubernetes. Prometheus grabbed attention from so many developers.

As we know, the metric data is large, so we must use reliable data store system to dealing the challenge of storage. 

Tranditianal APM(Application monitor) software uses the Mysql to store statistical data and Hbase to store the detail data(e.g. the code invoking stack of one request).

If we use Docker for running my application, The more application instance, The heavier metric data is, use PostgreSQL + TimescaleDB(be packaged as a PostgreSQL extension) perhaps is one good choice. I will try it later by buiding test environment.

B.T.W:The grafana is good UI dashboard framework of monitor!

# **3.Tips**
����Mysql���ݿ��ַ�����Ϣ�洢��������

### **3.1 ��������**
������ά��Ʒ��һ�����ܾ�����ҵִ�У���ҵ���Ե���ĳ̨�����ϵĽű������ǽ�����Ҫ����ű���"��׼���"����ÿ����ҵ���ýű��ı�׼�������Ҫ�洢�����ݿ��й��Ժ�鿴�ģ�����ִ����4����ҵ���ȣ�����������һ����ҵ����������������濴������׼�����


### **3.2 ԭ�����**
ͨ��������ҵ��׼���������:ִ����ҵ(���ýű�)������->���ݿ�->���档�������뵽�鿴ִ����ҵ�����ϵĴ����˵���־��ͨ������һ�����ҵ�ԭ����־�������Ե��ֶγ��޴���

```python
MysqlDataTruncation: Data Truncation: Data too long for column "xxx" at row 1
```

### **3.3 �������**
��Ʒ�����������������ˣ�û���뵽��׼�����Ȼ�����ˣ���Ʒ�������ߺ����ݿ��ж�Ӧ���ֶ�����ΪText��������ϸ����Mysql���ֲᣬ����TEXT���ɽ��ܵ���2^16(65536)�ֽڣ���˾͵����Ͱ����ݿ��ֶ����ʹ�TEXT��ΪMEDIUMTEXT(2^24)
```python

ALTER TABLE xxx MODIFY COLUMN yyy MEDIUMTEXT BINARY NULL;
```

MYSQL�ַ������֡�ʱ���ֶδ洢�Ŀռ��СҪ��������ʾ��
![avater](../images/3-mysql-string-type-storageSize.png)

# **4.Share(ǳ̸���ù�������)**

���������ճ���ά��˵�������һ��̨���߼�̨������ά�Ļ�����������Ҫ������ʱ��ͨ����½ÿ̨�����������ã�������ͨ��sshȥ���������á����������������ʮ�ϰٳ�ǧ̨��ʱ�򻹲������ַ�ʽЧ�ʾ����������ˡ��������һ�㶼��Ҫ����ר�ŵ����ù���������
Ŀǰ���е���Puppet(ʹ��ruby����)��Chef(ʹ��ruby����)��Ansible(ʹ��python����)��SaltStack(ʹ��python����)�ȡ�
���ߵĹ�����Ϊ�д���(Agent)���޴���ģʽ(Agentless):

1,����(Agent)ģʽ����һ���������������ڱ���ضˣ����շ���˵�����������ù���������

2,�޴���(Agentless)ģʽ������Ҫ��Զ�������ϰ�װclient/agents����Ϊ�����ǻ���ssh����������(��Ϣ�м��)����Զ������ͨѶ�ġ�

### **4.1 Puppet**
Puppet�ǿ�Դ�Ļ���Ruby��ϵͳ���ù������ߣ�������C/S�Ĳ���ܹ���Puppet�������ļ����û�����������ϵͳ�����Ϊ����Դ����ÿ����Դ���ж�Ӧ�����ԣ����������а�װ����װ�����ԣ��ļ���Ȩ�����Եȡ�Puppet�Ĵ�����Ҫ����Щ��Դ����������ɡ�

#### **4.1.1 Puppet������������ʾ��
![avater](../images/3-S-puppet-dataFlow.png)

#### **4.1.2 Puppet�ļ�ʹ������ͼ��ʾ��

��Ҫչʾ�ļ�ͬ����ʵ��
![avater](../images/3-S-puppet-useFlow.png)

#### **4.1.3 Puppet���õ���Դ�����ඨ��
��Щ��Դ������Ҫ��д��".pp"Ϊ��׺�ļ��м���

-�ļ����Ͷ��壺����ΪҪ������������Ҫͬ���������ϵ�my.cf�ļ���/etc/mysql/my.cnf
```python
file { 
    "/etc/mysql/my.cnf": 
    source => "puppet:///mysql-server/my.cnf", 
    owner => root, 
    group => root, 
    mode => 0644;        
}
```
-��������װ��
```python
package { 
      ["nginx","mysql-server"]: 
      ensure => installed; 
      ["vim"]: 
      ensure => absent; 
}
```
-ִ��cron��ʱ����
```python
cron{  
 "ntpdate":  
  command => "/usr/sbin/ntpdate pool.ntp.org",  
  user => root,  
  hour => 0,  
  minute => 0,  
  } 
```
-ִ�й̶�����
```python
exec {  
  "/tmp/nginx_install.sh":  
  cwd => "/tmp",  
  user => root,  
  path => ["/usr/bin","/usr/sbin","/bin","/bin/sh"],  
  }
```


### **4.2 Chef**
����Puppet�����ù������ߣ�Ҳ�������ļ����û�����������ϵͳ�����Ϊ����Դ����

#### **4.2.1 Chef�ܹ��ͼ�˵��������ʾ��
![avater](../images/3-S-chef-dataFlow.png)

#### **4.2.2 Chef�ļ�ʹ������ͼ��ʾ��

����ͨ���鿴��ͼ���ļ����ú�����ִ�У�Ҳ�������Կ�����Puppet�м������ơ�
![avater](../images/3-S-chef-useFlow.png)


### **4.3 Ansible**
Ansible������Ϊһ��ģ�����������ù�������֧�ֶ�ڵ㷢����Զ������ִ�С�Ĭ��ʹ�� SSH ����Զ�����ӡ�
Ansible�����ڱ������ڵ��ϰ�װ������������ʹ�ø��ֱ�����Խ�����չ���������ܳ�ΪAnsible���е����ԭ��֮һ��

#### **4.3.1 ����˵���Ļ������
Host Inventory��ָ������������,��һ�������ļ����涨���ص���������Ҫ�����������������ڸ��ļ��С�

PlayBooks �� �������������ansible�Զ�ִ��һϵ������������PlayBook

Module��Ping,command,user,yum�ȵ�ģ�飬ͨ��ģ�������Ӧ����������

���ӷ�ʽ��������(Ҳ��������ansible����Ļ���)ͨ��local ��ssh��zeromq ���ַ�ʽ���ӱ������ˣ�Ĭ��ʹ�û���ssh�����ӡ�

#### **4.3.2 Ansible����ʹ�ã�

Ansible��ʹ������򵥣��ڰ�װ���֮��Ĭ�Ͼ���command������ģ����ڣ�����ֻ��Ҫ�ѱ����������ӽ��������Զ��ִ�����
�����൱���㣬Ҫ�����ϸ�֪�Ļ�ֻ��Ҫ�����������

1������hosts�ļ�,���������Ӧ�����������ڹ��������϶���������������
```python
vi /etc/ansible/hosts
---------------------
[webservers]
rhel7161
rhel7160
[dummy_group]
192.168.1.111
192.168.1.112
```

2���ڹ�������ͨ��ansibleִ��Զ���������ͼ��ʾ�����������ϲ�ѯjdk�İ汾��Ϣ��
![avater](../images/3-S-ansible-useFlow.png)

#### **4.3.3 Ansible��ȶ��ư�װ���ã�
�����Ҫִ�а�װ�����Ļ������Ը�ϸ���ȵı�������ɣ�����ͼΪ����tomcat�İ�װ��
![avater](../images/3-S-ansible-deepUsage.png)

### **4.4 Salt**
��ζ�Salt���ñȽ�ǳ��Ҳû�д�ɹ����Ի����������������ͼʾչʾSalt��ʹ�ã����Կ���ʹ��Ҳ�ǱȽϼ򵥵ġ�
![avater](../images/3-S-salt-useFlow.png)

#### **4.5 �ֲ�ʽ�����ܽ�**
������������õĹ��������Ʋ�ansible��ʹ������Ӧ���������еġ�

�ںܶ๫˾�����Լ������ÿ�ܣ���Щ�ǻ������Ͽ�Դ��Ʒ����֮һ���д��
Ҳ�����п���Զ�����ÿ��������¹����ģ�

1��������ҵִ��

2����������(�ر����м��)��װ������

3���Զ�Ѳ��(����Զ����ҵʵ�������洢�ռ䣬CPU���ڴ�ȵ��Զ�Ѳ��)

����Ҳ����Ѷ����������ƽ̨��