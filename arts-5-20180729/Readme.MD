This is 5th ARTS, The Algorithm is still difficult for me again and again.

# 1.Algorithm
### Question: 

Given a 32-bit signed integer, reverse digits of an integer.

Example 1:
```
Input: 123
Output: 321
```

Example 2:
```
Input: -123
Output: -321
```

Example 3:
```
Input: 120
Output: 21
```
### My solution in python3 is:
In this solution,  we use the str and int function to reverse the type among int and string. Also we should deal with the sign, and delete the pre-ordered 0 in reversed string.
```
class Solution:
    def reverse(self, x):
        """
        :type x: int
        :rtype: int
        """
        if x == 0:
            return 0
        
        cx = abs(x)
        sx = str(cx)
        # the reversed string of input int
        rsx = sx[::-1]
        
        count = len(rsx)
        zIndex = -1
        # we must delete the pre-ordered 0
        for i in range(0,count):
            if rsx[i] != "0":
                break
            zIndex = i
        
        resultStr =  rsx[zIndex + 1:]

        if x < 0:
            resultStr = "-" + resultStr
        
        # Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [?231,  231 ? 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows
        rInt = int(resultStr)
        lValue = -(2**31)
        rValue = 2**31
        if rInt > rValue or rInt < lValue:
            return 0
        return rInt
```

# 2.Review
Today, I read this article [link](https://itnext.io/how-we-avoided-if-else-and-wrote-extendable-code-with-strategy-pattern-256e34b90caf) which about the Strategy Pattern for extendable code.

If we construct the offer in user class in coding,  many if/else statement is necessary. we needed to modify the user class when a new offer type was introduced. To avoiding this, import a new class which named UserOffer, and introduce setOffer method for User class, this will help us for decouple offer and user. improve the flexibility of code.

# 3.Tips
说说一次"**奇葩**"的乱码之旅。

**背景**：我们在生产环境对应用进行测试，架构相对比较复杂，涉及到很多个业务系统。运行了几百个中间件实例，大的架构方面：**浏览器**<->**web实例的中间件**(类似Tomcat)<->**app实例的中间件**(类似Tomcat)，这里边有大量的AJAX请求。

![image](../images/5-T-request-stream.png)

**下面是上图的markdown源码，发现浏览器不支持，网易笔记上是支持的。可不予理会**

  ```
  sequenceDiagram
  浏览器->>web实例: http请求获取
  web实例->>app实例: 构造http请求!
  ```

采用了常规的负载均衡，请求在70多台机器的700多个实例上流转，类如下图，**Bx**是web实例集群，**Cx**是app实例集群。
![image](../images/5-T-request-lb.png)

**下面是上图的markdown源码，发现浏览器不支持，网易笔记上是支持的。可不予理会**
  ```
  graph TD
  浏览器-->负载均衡器
  负载均衡器-->B1
  负载均衡器-->B2
  B1-->C1
  B1-->C2
  B1-->C3
  B2-->C1
  B2-->C2
  B2-->C3

  ```

**防乱码准备工作**：我们给Tomcat中间件配置了uriEncoding=UTF-8、useBodyEncodingForURI=true、默认的contentType为text/html;charset=UTF-8等。并**抽查了**多数中间件实例的JVM参数，放心地以为全是file.encoding=UTF-8。此外，应用中也有很多配置请求参数和相应参数编码的配置。

**问题点**：如图片所示，部分页面的部分字段出现乱码，然而如果对乱码页面进行刷新发现，只有某几次请求是乱码的。

![image](../images/5-T-page-encodingError.png)

**诊断过程**：我们很容易想到这个问题出错的只是后端的部分应用实例，当负载均衡请求被发送到有问题的实例时页面就会乱码，这是偶现问题。那么我们前面已经做了很多跟编码相关的配置，为什么还会出现乱码？在后端几十台主机和几百个应用实例中怎么准确找出问题实例成为首要的任务。当然肯定有人会问为什么测试不能针对具体的主机(这个主要是网络环境和数据原因)。

通过httpwatch抓取，频繁刷新页面直到乱码页面产生。在前面问题浮现的图中最下面我贴出了HTTP相应头，还好，应用会把后端应用实例的名称放到http头中，所以乱码页面很容易就定位到主机。然后接下来就是仔细查看主机上对应的Web实例日志，查找乱码时刻前后的日志进行分析，最后发现了web实例日志中**构造请求时的请求体就是乱码的**。这个乱码是web实例读取本地文件模板构造请求发往app实例，从而从数据库中获取数据：


```
checkContracNew:{"SUBMIT_TYPE":"submit","listener":"onTradeSubmit","DEPOSIT_FEE":"0","GIFT_FEE":"2500","CHANGEPRODUCT_INFO":"{\"PRODUCT_ID\":\"99813102\",\"YY\":[{\"IS_EXIST\":\"TRUE\",\"ELEM_KEY\":\"99813112_D_0\",\"PRODUCT_ID\":\"99
813102\",\"PACKAGE_ID\":\"VOICE\",\"ELEMENT_ID\":\"99813112\",\"ELEMENT_TYPE_CODE\":\"D\",\"ELEMENT_NAME\":\"???????99 ??\",\"PRICE\":\"9900\",\"PACKAGE_KIND_CODE\":\"YY\",\"MODIFY_TAG\":\"0\"}]}","SERIAL_NUMBER":"18817036279","MONTHS":"
24","EXEC_ACTION":"undefined","CONSUME_KMNUM":"99","REMARK":"","CHECK_MODE":"0","IS_CASH_VOUCHER_CREATE":"1","service":"ajax"}
```

然后跟应用厂商沟通，确认了确实存在读取文件的操作，进而很快想到可能是file.encoding配置问题，就很快检查出现乱码是处理请求的实例的JVM参数确实跟没有问题的有区别。

```
file.encoding = ANSI_X3.4-1968 //产生乱码的应用中间件实例
file.encoding = UTF-8 //正常的应用中间件实例
```
检查主机上的local和LANG均是相同的zh_CN.UTF-8,运行简单的Java程序发现-Dfile.encoding也是UTF-8,而中间件上则有区别，这个感觉比较诡异，但没有细查了。

**最后为了屏蔽配置差异，给700多个中间件实例强行设置-Dfile.encoding=UTF-8后解决**。

# 4.Share（关于IPV6）
## 4.1 IPV6基础
1,相比IPV4地址是32位(xxx.xxx.xxx.xxx)而言，IPV6用128位标识可以代表足够多的地址,IPV6用冒号(:)分隔(X:X:X:X:X:X:X:X)

2,IPV6地址中每一位的前导0可以省略，地址中可以(最多)出现两个冒号(::)来标识一段0，比如我们熟知的IPV4 loopback地址“127.0.0.1”对应的IPV6地址为“::1”以及IPV4的全0地址“0.0.0.0”在IPV6则为“::”

3，IPV6有FE80::/10和FEC0::/10的两个地址分别为链路本地和站点本地地址，以及对应IPV4公网地址的全球单播地址(global)如"2001:abcd:1000::130/64"，组播地址则以FF打头(如节点本地组播地址：FF01::1)。每个接口除一个链路本地地址外还可以配置一个站点本地地址和一个或多个可聚集全球地址。一般我们在主机上启动IPV6模块均有一个FE80打头的地址。
## 4.2 IPV6配置
操作系统：目前我们使用的大多数操作系统都已经很好的支持了IPV6，Linux内核2.4版本就很好的支持了，windows xp sp1及windows 2003就支持。
就Java而言，在Linux操作系统上JDK1.4就支持了IPV6，WIndows上则是从JDK1.5开始才支持。

以Redhat6为例，IPV6配置方法：

### 4.2.1，操作系统配置(Redhat6以上版本为例)需要注意：
**1，/etc/sysctl.conf**中的net.ipv6.conf.all.disable_ipv6选项，为1则表示禁用IPv6

**2，/etc/sysconfig/network**中配置NETWORKING_IPV6为yes和no分别表示IPV6的启动和禁用

**3，/etc/sysconfig/network-scripts/ifcfg-ethx**中配置IPV6INIT为yes或no

4，假如测试ipv4地址通了IPV6地址不通时则注意检查**chkconfig ip6tables off**关闭ipv6防火墙

**5，/proc/sys/net/ipv6/bindv6only**文件默认值为0表示Socket绑定支持V4和V6，如果设置为1则表示只绑定V6地址。

### 4.2.2，应用程序

1 Java的-Djava.net.preferIPv4Stack设置为true则表示Socket绑定到V4地址，为false则为V4和V6均支持。

2 在操作系统已经配置了IPV4和IPV6的情况下，绑定socket不管传递V4的"0.0.0.0"还是V6的"::"均可以支持V4和V6的通信

## 4.3 IPV6测试(以Java为例实现一个简单的聊天)

**4.3.1 服务端实现**

```
import java.io.*;
import java.net.*;

public class TalkServer {
	public static void main(String args[]) {
		
		/*使用方法为java TalkServer 监听地址 端口*/
		if(args.length < 2){
			System.out.println("Usage:java TalkServer $host $port");
			System.exit(2);
		}

		ServerSocket server = null;
		try {
		    server = new ServerSocket(Integer.parseInt(args[1]),1,InetAddress.getByName(args[0]));

			Socket socket = null;
			while(true){
				try {
					//接收到来自客户端的请求，打印客户端地址和端口
					socket = server.accept();
					System.out.println("Accepted connection from[" + socket.getInetAddress().getHostAddress() + ":" + socket.getPort() + "].");
				} catch (Exception e) {
					System.out.println("Error." + e);
				}
				System.out.println("");
				String line;
				BufferedReader is = new BufferedReader(new InputStreamReader(
						socket.getInputStream()));
				PrintWriter os = new PrintWriter(socket.getOutputStream());
				
				//从客户端获取信息
				while ((line = is.readLine()) != null && !"bye".equals(line)) {
					System.out.println("Received message from Client:" + line);
					os.println("Message Replied by Server:[" + line + "]");
					os.flush();
				}
				
				try{
					os.close();
					is.close();
					socket.close();
				}catch(Exception e){
					
				}
			}
		} catch (Exception e) {
			System.out.println("Error:" + e);
		}finally{
			try {
				server.close();
			} catch (Exception e2) {
			}
		}
	}
}
```

**4.3.2 客户端实现**


```
import java.io.*;

import java.net.*;

public class TalkClient {

	public static void main(String args[]) {
		Socket socket = null;
		BufferedReader is = null;
		PrintWriter os = null;
		try {
			/*使用方法为java TalkClient ip地址 端口*/
			if(args.length < 2){
				System.out.println("Usage:java TalkClient $host $port");
				System.exit(2);
			}
			
			//连接服务器端，这里可以通过IPV4和IPV6地址进行连接
			socket = new Socket(args[0], Integer.parseInt(args[1]));
			BufferedReader sin = new BufferedReader(new InputStreamReader(
					System.in));
			
			os = new PrintWriter(socket.getOutputStream());
			is = new BufferedReader(new InputStreamReader(
					socket.getInputStream()));

			String readline;
			readline = sin.readLine();
			
			//连接上服务器端后可以一直输出信息
			while (!readline.equals("bye")) {
				System.out.println("Will Send Message:" + readline);
				os.println(readline);
				os.flush();
				System.out.println(is.readLine());
				readline = sin.readLine();
			}

		} catch (Exception e) {
			System.out.println("Error" + e);
		}finally{
			try{
				os.close(); 
				is.close();
				socket.close();
			}catch(Exception e){
				
			}
		}
	}
}
```
**4.3.3 测试**

服务器的IP地址：

```
[root@rhel6130 ~]# ifconfig -a
eth0      Link encap:Ethernet  HWaddr 00:0C:29:D2:18:5F  
          inet addr:192.168.32.130  Bcast:192.168.32.255  Mask:255.255.255.0
          inet6 addr: 2001:abcd:1000::130/64 Scope:Global
          inet6 addr: fe80::20c:29ff:fed2:185f/64 Scope:Link
```

服务的前端通过

"java TalkServer :: 6666"  或者

"java TalkServer 0.0.0.0 6666"

运行后通过netstat查看均是全地址监听

```
[root@rhel6130 ~]# netstat -anp | grep 6666
tcp        0      0 :::6666                     :::*                        LISTEN      3173/java
```

客户机IP地址：

```
C:\Users\xuqk>ipconfig
Windows IP 配置

   连接特定的 DNS 后缀 . . . . . . . :
   IPv6 地址 . . . . . . . . . . . . : 2001:abcd:1000::100
   本地链接 IPv6 地址. . . . . . . . : fe80::6c4d:c260:8631:b199%18
   IPv4 地址 . . . . . . . . . . . . : 192.168.32.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
```

客户端可以通过IPv4和IPv6地址进行连接请求

"java TalkClient 2001:abcd:1000::130 6666" 或者

"java TalkClient 192.168.32.130 6666",

需要注意：

1，如果服务器端运行时加了 -Djava.net.preferIPv4Stack=true参数，则客户端智能通过IPV4地址进行连接。

2，遇到过一个问题，linux服务器正确配置了IPV6的global单播地址了，但是windows客户端无法通过这个地址连接，后来发现是需要给客户端主机也配置IPV6 global地址才可以，比如上面的“2001:abcd:1000::100”地址就是我测试用的IPV6地址。

## 4.4 关于IPV6的几点感悟
1，IPV6自2003年推出，至今没有太大规模商用

2，2018年6月，三大运营商联合阿里云宣布，将全面对外提供IPv6服务，并计划在2025年前助推我国互联网真正实现“IPv6 Only”（仅支持IPv6）。这一点我在电信行业感悟很深，现在各省公司已经要求中间件和业务系统必须做IPv6的支持。实际上很多中间件很早就支持了IPV6的。对于业务系统的IPV6支持实际上主要是测试验证的工作。这个宣言将推广IPv6真正提上了日程。

3，关于业务系统的IPV6全面测试总结事项，有如下总结：

3.1 URL所有连接服务器的URL地址，包含IPV6地址的需要用中括号括起来。
  
如浏览器直接通过IPV6地址访问的话，http://[2001:abcd:1000::130]:8080/webTest/index.html这样的地址。 

如JMX连接地址:service:jmx:rmi://[2001:abcd:1000::130]:6600

3.2 对于全面测试应该包括：

a，Web容器NIO和BIO模式的IPV6地址访问

b，商用中间件EJB容器的IIOP协议或者私有协议(如weblogic t3协议)的访问，通常为JNDI连接串需要使用IPv6地址进行验证

c，JMS连接及JDBC连接的ipv6验证

d，web服务器nginx和apache的IPV6验证，如果使用使用apache + tomcat的架构时需要验证AJP协议的IPV6支持，而apache的JK插件很有可能不支持IPV6，则需要使用mod_ajp_proxy这样的模块进行转发

e，中间件或者各类软件如果是分布式部署的话，需要验证各管理组件之间的IPV6通信

f，各软件的客户端通过IPV6地址连接服务器的验证
