This is my second ARTS, i feel lighter than the first time.
# **1.Algorithm**
**Question��**

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

����ȱ��ǿ��Ǵ��~_~

# **3.Tips**
������Docker�����»�ȡ����ϵͳ��¼�û��������⣬getlogin()��getuid()�Ƚϡ�Docker��������Ϊ��yjjy0921/redhat7.2

### **3.1 ��������**
��Ǩ��ĳ��Ӧ�õ�Docker����ʱ������Ϊ���ò���ϵͳ��getlogin()����(sys/types.h)��ȡ������¼�û����õ���strcpy�����ĵ���ֱ��core�ˡ�
��ʱ�Ǵ�Redhat7.2������Ǩ�Ƶ�����Redhat7.2��Docker�����С�

������Ĵ������£�getlogin() �ĵ��ûᵼ��core
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
### **3.2 ԭ�����**
 ������/var/run/utmp�ļ��йأ�ʹ��docker cp����������������ϵ�/var/run/utmp�ļ���������Ӧ��/var/run/utmpҲ���У����Ľ���������������ҵ�һ��redhat7.2������/var/run/utmp�ļ��������е��Խ����
���ⲻ�����ս�������������stackoverflow�Ϸ������һ����Ҫ��ȡ��¼�û��Ļ�����������Ϊͨ��getuid������ȡ��getlogin()��һ������ȫ�ķ��������Ƽ��������https://stackoverflow.com/questions/4785126/getlogin-c-function-returns-null-and-error-no-such-file-or-directory��

### **3.3 �������**
����������������������������getuid�������Ժ���Ч�Ĺ��getlogin������/var/run/utmp�ļ�Ӱ�죬�Լ����ڵİ�ȫ�����⣺
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

# **4.ǳ̸���л�����**
### **4.1 ���л��Ƚ�**
�����JavaΪ����̸һ�����л��ĳ�����

���л�����������������������л���һ������ֽ��롢һ����Ը�ʽ�����ַ�����

1���ֽ������л���Protobuf��Hessian��Java���л�

2��Json��xml���л���jackjson��xstream��

����ͼ��ʾ���ڷֲ�ʽ�����пͻ��˺ͷ�������ͨ���漰�������л��������������ڵ�SOAP WebServiceʵ���д���ʹ��XML��ʽ������Restful�ӿڵ�Web
Service��ʹ��JSON��ʽ��δ����Docker��K8S��YAML��ʽ�᲻���п�����������Զ�̵��ã�  ������������Զ�̵��ÿ��gRPC��Thrift��ICE��Dubbo�ȵײ���ò�ͬ�����л���������һЩ���õ��м���ڷֲ�ʽ�����������Լ��ĵ���Э��(��Weblogic T3)���ײ�����л������ܿ��ܲ��õľ��������ֽ������л������е�һ�֡�

![avater](../images/2-distribute-serialize-summary.png)

### **4.2 �������л���ܵ�ʹ�÷���**
#### **4.2.1 ���л��������**
���Ǳ���ʹ�����л���������Java����ʾ����User�����������ơ����롢���䡢��ַ(�ɶ��)���ԡ�

```java
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

public class User implements Serializable{
	private static final long serialVersionUID = 1L;
	
	private String name;
	private String password;
	private int age;
	private List<String> address = new ArrayList<String>();
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public List<String> getAddress() {
		return address;
	}
	public void setAddress(List<String> address) {
		this.address = address;
	}
}
```

���ǵĲ��Դ����У�����ʹ�þ߱���ͬ����ֵ�Ķ��󣬻�򵥱Ƚ����л��ֽ�����С��ʹ�÷�����
```java
		// ������Զ���
		User user = new User();
		user.setName("bes");
		user.setPassword("BESDev");
		user.setAge(8);
		
		user.getAddress().add("BeiJing ChaoYang");
		user.getAddress().add("SuZhou JingChang");
		user.getAddress().add("XiAn GaoXin");
```

#### **4.2.2 JDK���л�**
JDK�����л�ʹ������򵥵ģ�ֻ��Ҫ��֤��Ҫ���л�����ʵ��java.io.Serializable�ӿں�ʹ��java.io.ObjectOutputStream���writeObject��readObject�����ʵ�����л��ͷ����л�������ʾ�����룺
```java
package serialize;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;

import hessian.User;

public class JDKSerializeMain {
	public static void main(String[] args) throws Exception {
		// ������Զ���
		User user = new User();
		user.setName("bes");
		user.setPassword("BESDev");
		user.setAge(8);
		
		user.getAddress().add("BeiJing ChaoYang");
		user.getAddress().add("SuZhou JingChang");
		user.getAddress().add("XiAn GaoXin");

		ByteArrayOutputStream output = new ByteArrayOutputStream();
		ObjectOutputStream objectOut = null;
		try {
			objectOut = new ObjectOutputStream(output);
			objectOut.writeObject(user);
			objectOut.close();
			output.close();

			System.out.write(output.toByteArray());
			System.out.println();
			System.out.println("���л�����ֽڴ�С��" + output.toByteArray().length);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}

```
#### **4.2.3 Hessian���л�**
Hessian���л���JDK�Դ������л�����Ϊ����Ҫ����Hessian��Input��Outputʵ��(��)���л���ʾ����������Ҳ�ܼ򵥡�
```java
package hessian;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;

import com.caucho.hessian.io.Hessian2Input;
import com.caucho.hessian.io.Hessian2Output;

public class HessianMain {
	public static void main(String[] args) {
		//������Զ���
		User user = new User();
		user.setName("bes");
		user.setPassword("BESDev");
		user.setAge(8);
		
		user.getAddress().add("BeiJing ChaoYang");
		user.getAddress().add("SuZhou JingChang");
		user.getAddress().add("XiAn GaoXin");

		try {
			//���л�����
			ByteArrayOutputStream os = new ByteArrayOutputStream();
			Hessian2Output ho = new Hessian2Output(os);
			ho.writeObject(user);
			//Hessian�Դ����壬��Ҫ����flush����
			ho.flush();
			byte[] bytes = os.toByteArray();

			System.out.write(bytes);
			System.out.println();
			System.out.println("���л�����ֽڴ�С:" + bytes.length);

			//�����л�����
			ByteArrayInputStream is = new ByteArrayInputStream(bytes);
			Hessian2Input hi = new Hessian2Input(is);
			User deUser = null;
			deUser = (User) hi.readObject();
			
			System.out.println(
					"User:[name:" + deUser.getName() + ",password:"
			      + deUser.getPassword() + ",age:" + deUser.getAge() + "]");
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
}

```

#### **4.2.4 protobuf���л�**
protobuf��ܵ�ʹ������Ҫ����protobufЭ�鼰�涨�ĸ�ʽ������Ҫ���л��Ķ����ʽ��

1��ʹ��protobuf�ĵ�һ�����Ƕ������µĸ�ʽ(����ΪMessage)��ͬǰ���ڵ�User�ࡣ
```docker
package bes;
option java_package="myprotobuf";
option java_outer_classname="UserProto";

message User{
    required int32 ID=1;
    required string userName=2;
    required string Password=3;
    required int32 age=4;
    repeated string address=5;
}
```
2��ʹ��ptotoc�������proto�ļ�����������Ե�Message�༰������

protoc --java_out ./ User.proto

3������protoc���������UserProto�࣬��Ҫ����builder�ͷ����л��ȵĹ����࣬��UserProto.User.Builder����������ʹ��protoc���ɵĴ���������л��ͷ����л���
```java
  /**
   * Protobuf type {@code bes.User}
   */
  public  static final class User extends
      com.google.protobuf.GeneratedMessageV3 implements
      // @@protoc_insertion_point(message_implements:bes.User)
      UserOrBuilder {
    // Use User.newBuilder() to construct.
    private User(com.google.protobuf.GeneratedMessageV3.Builder<?> builder) {
      super(builder);
    }
    private User() {
      iD_ = 0;
      userName_ = "";
      password_ = "";
      age_ = 0;
      address_ = com.google.protobuf.LazyStringArrayList.EMPTY;
    }
```
4������ʹ��protobuf��ʾ���������£�
```java
package protobuf;

import java.util.ArrayList;
import java.util.List;

import com.google.protobuf.InvalidProtocolBufferException;

import myprotobuf.UserProto;

public class UserSerial {
    public static void main(String[] args)
            throws Exception {

        //����Builder��������Ϣ����User������
        UserProto.User.Builder builder = UserProto.User.newBuilder();
        builder.setID(1);
        builder.setUserName("bes");
        builder.setPassword("BESDev");
        builder.setAge(8);
        List<String> address = new ArrayList<String>();
        address.add("BeiJing ChaoYang");
        address.add("SuZhou JingChang");
        address.add("XiAn GaoXin");
        builder.addAllAddress(address);
        
        //ʹ��protoc���ɵ���build��User����
        UserProto.User user = builder.build();
        
        //���л�User����
        byte[] encodedBytes = user.toByteArray();
        System.out.write(encodedBytes);
        System.out.println();
		System.out.println("���л�����ֽڴ�С��" + encodedBytes.length);
        
        //�����л�
        UserProto.User decodedUser = UserProto.User.parseFrom(encodedBytes);
        System.out.println("decodedUser:[" + decodedUser +"] "
        		+ "is equals(user):" + decodedUser.equals(user));

    }

}

```
#### **4.2.5 ���л����ܽ�**
�Ƚ����л�ǰ��Ķ����ڱ�֤�������л�ǰ��һ�µ�����£����Դ����л��ӡÿ�����л��Ĵ�С��

����ͼ�Ǽ򵥵ıȽ�
![avater](../images/2-distribute-serialize-compare.png)

�������л��ļ�˼����
1�����л��ڷֲ�ʽϵͳ�й㷺ʹ�ã���gRPCԶ�̵��ÿ��ʹ�õľ���protobuf��Thrift�ֲ�ʽ���ÿ�������Լ������л���ܣ������dubbo��ʹ��hessian���л�����(Ҳ֧��json����protobuf������չ��)��
2�����л����������շֲ�ʽ������˵�Ǻ���Ҫ�ģ����Ҷ���������������Ҫ��
3����ĿǰӦ�����ܼ��APM��Ʒ�У�����̸�Ķ˵��˼�أ��ڲ��û���HTTPЭ��ͨ�ŵ�ϵͳ֮��ʵ������ǱȽ����׵ġ�

����ʵ�кܶ�ϵͳ֮��Ķ˵�����·�����Ϊ���ڲ�ͬ�ķֲ�ʽͨ��Э�飬Э�鱨�ĵ����л���ʽ��ͬ����Щϵͳʹ�����м��������ͨ��Э�飬���������ʹ��EJB������ϵͳ����Ϊͻ��������Ҫ֧��Weblogic�µ�Ӧ����Ҫ����T3Э�顣���ⲻͬϵͳ���ò�ͬ�ķֲ�ʽ��ܣ��˵��˼�ؿ����Ҫ�������ķֲ�ʽ��ܽ���֧�֡�
�������Ҫʵ�ֶ˵��˼�صĻ�����Ҫ�Ը��ֲַ�ʽ������ͨ��Э�鶼�����˲���ʵ�ֶ˵��˼�ء�����ܶ�