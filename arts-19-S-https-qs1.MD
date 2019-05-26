# *** HTTPS���úͱ������
��ƪ������Ҫ����HTTPS�������˵����úͿͻ��˷��ʷ����ģ�ֻ�漰��������֤��Ҳ���ǿͻ������η�������

���л��漰��java��keytool����������ǩ��֤�鼰����֤��Ȳ������Լ�tomcat��ô����֤�����Ϣ

 ## 1��֤�����
 ### 1.1 ���ɷ�����֤�顾����˲�����
 ��������"keytool -genkeypair"Ϊ���ɷ�������֤����������-alias��ʾ����(һ��֤����ļ����Դ�����Կ���ñ�������)��-keystore��ʾ��Կ���ļ�������ļ���û�е�����»��½�������������Կ���ȡ���Ч�ڵȲ�����
```
keytool -genkeypair -alias xuqk -keysize 2048 -keyalg RSA -validity 3650 -keystore tomcatserver.jks -storetype JKS
```
����Ϊ��ɵĲ������ر�ע����Ҫ������Կ��������Կ����(˳��˵һ��JDKĬ�ϵ���Կ����$JAVA_HOME/jre/lib/security/cacert�ļ��У�����Ϊchangeit)��ÿһ����Կ���Ե������ò�ͬ�Ŀ��������Կ��������Ժ����Կ���ļ����в���ʱ����Ҫ�ṩ�ġ�
```
keytool -genkeypair -alias xuqk -keysize 2048 -keyalg RSA -validity 3650 -keystore tomcatserver.jks -storetype JKS
    ������Կ�����:
    �ٴ������¿���:
    ����������������ʲô?
      [Unknown]:  xuqk
    ������֯��λ������ʲô?
      [Unknown]:  bes
    ������֯������ʲô?
      [Unknown]:  bessystem
    �����ڵĳ��л�����������ʲô?
      [Unknown]:  chongqing
    �����ڵ�ʡ/��/������������ʲô?
      [Unknown]:  chongqing
    �õ�λ��˫��ĸ����/����������ʲô?
      [Unknown]:  CN
    CN=xuqk, OU=bes, O=bessystem, L=chongqing, ST=chongqing, C=CN�Ƿ���ȷ?
      [��]:  ��
    
    ���� <xuqk> ����Կ����
            (�������Կ�������ͬ, ���س�):
```
 ### 1.2 ������Կ������˲�����
 �ո����ɵ���Կ���ļ������˹�Կ��˽Կ��������Ҫ�ѹ�Կ��������Է�����Ҫ���ʷ���Ŀͻ��ˡ��еķ������"***�����������ֱ�ӷ���***"�Ļ���ĳЩ�ͻ�����ԱҲ���������������֤�顣
 ����������Ӹո����ɵ���Կ���ļ�tomcatserver.jks�е�����Կ�������浽xuqk.crt�У�-file���������ļ��ġ�����ļ����Է�����Ӧ�Ŀͻ��ˣ��ͻ�����Ҫ�����֤�鵼�뵽���ε�֤����С�
```
keytool -export -alias xuqk -keystore tomcatserver.jks -file xuqk.crt
    ������Կ�����:
    �洢���ļ� <xuqk.crt> �е�֤��
```

 ### 1.3 �����������˹�Կ֤�鵽�ͻ�������֤��⡾�ͻ��˲�����
 ��������Ϊ�Ѵӿͻ��˻�ȡ���Ĺ�Կ֤��<xuqk.crt>���뵽�ͻ��˵�����֤����У�-keystore�ڲ����ڵ�������ǻ����ɵģ���-keystoreָ�����ļ�������ʱ��ȫ��������Կ�����룬���ļ�����ʱ���ṩ֮ǰ���õ���Կ�����롣���⣬Ϊ�����ֲ�ͬ��֤�飬����ͨ��-alias��ָ��������ͨ������Ҫ���õġ�
```
keytool -importcert -alias xuqk-from-server -file xuqk.crt -keystore trust.jks
```

 ## 2������������
 ���������м��ΪTomcat�����conf/server.xml�ļ����ҵ�8443�˿ڶ�Ӧ��connector��������н�ע�Ͳ���(Ҳ���ǿ���)��Ȼ������keystoreFileΪ���ɵ�֤����ļ�·����keystorePassΪ��Կ����
```
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS"
			   keystoreFile="E:\docs\https\tomcat-keyfile\tomcatserver.jks"  
               keystorePass="12345678"  />
```
�������ú�֮��Ϳ�������tomcat�ˡ�
���������Ļ����������������Ĵ��ڻ�������־�п�����Starting ProtocolHandler ["http-bio-8443"]��������˵��https�����Ӧ��8443���������ˡ�
```
��Ϣ: Deploying web application directory D:\tools\tomcat-7.0.42\webapps\ROOT
���� 25, 2019 3:05:36 ���� org.apache.coyote.AbstractProtocol start
��Ϣ: Starting ProtocolHandler ["http-bio-8088"]
���� 25, 2019 3:05:36 ���� org.apache.coyote.AbstractProtocol start
��Ϣ: Starting ProtocolHandler ["http-bio-8443"]
���� 25, 2019 3:05:36 ���� org.apache.catalina.startup.Catalina start
��Ϣ: Server startup in 716 ms
```

 ## 3���ͻ��˷���
  ### 3.1 ���������
  ͨ�����������8443�˿ڣ����յ���ȫ���棬����ͨ���߼�ֱ�ӷ��ʡ�
  
  ![image](./images/arts-19-S-https-qs1-browser.png)
  
  ������˳��˵һ�£�ǰ���ᵽһ���ؼ���"***�����������ֱ�ӷ���***",���������ĳЩREST��API������Բ��ܴ������ֱ�ӷ��ʡ������ֿ���ֱ����������ʵķ����ڷ������˲��ܵ����ṩ��Կ֤�������£�����ͨ�����������֤�鵽�Լ�������֤���(��Ȼǰ����ȷ�Ϸ����ǰ�ȫ��)������Ϊ���ȴ��������ַ��ֱ�ӵ��������"ͼ��鿴֤�飬Ȼ������򵼾Ϳ��Ե����������˹�Կ��
  
  ֮�����"1.3��"�Ϳ��԰������������֤�鵼�뵽����֤��⣬��������ʷ�ʽʹ�á�
  ### 3.2 ͨ���������
  ͨ�����ǿ�����Ҫͨ���������������Ϸ���https����
  ����Ϊһ���򵥵�http�ͻ��ˣ���������httpclient4�����������apache���أ���Ҫע����ǻ��������jar��Ҳ��Ҫ���أ�����common-codec��common-beanutils�ȡ�
  
  ͨ�����������ʵ��https����ķ��ʣ�
```
java -cp ./ com.bes.test.ClientCustomSSL https://localhost:8443
```
  ���ǣ���������ǻ�ʧ�ܵģ�������Ϣ�лῴ���ؼ��֣�javax.net.ssl.SSLHandshakeException���������Ϊ�ͻ���û�����η���ˡ�
  
```
Exception in thread "main" javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
        at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
        at sun.security.ssl.SSLSocketImpl.fatal(SSLSocketImpl.java:1949)
        ......
Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
        at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:387)
```
���ϴ������İ취�ǣ�����javax.net.ssl.trustStore��javax.net.ssl.trustStorePassword�����������ֵ�Ϳ�"1.3"������ָ��������֤����Կ���ļ�·�������

```
java -Djavax.net.ssl.trustStore=./trust.jks -Djavax.net.ssl.trustStorePassword=12345678 -cp ./ com.bes.test.ClientCustomSSL https://localhost:8443/
```

httpclient��java����������ʾ��
```
package com.bes.test;

import java.io.File;
import java.io.FileInputStream;
import java.security.KeyStore;

import javax.net.ssl.SSLContext;

import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.conn.ssl.SSLContexts;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.TrustSelfSignedStrategy;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class ClientCustomSSL 
{
    public final static void main(String[] args) throws Exception {
        if(args.length < 1){
            System.out.println("Usage: java com.bes.test.ClientCustomSSL $https_url");
            System.exit(1);
        }

        CloseableHttpClient httpclient = HttpClients.createDefault();
        try {
            HttpGet httpget = new HttpGet(args[0]);
            System.out.println("executing request" + httpget.getRequestLine());

            CloseableHttpResponse response = httpclient.execute(httpget);
            String result="";
            try {
                HttpEntity entity = response.getEntity();

                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                if (entity != null) {
                    System.out.println("Response content length: " + entity.getContentLength());
                    if (response.getStatusLine().getStatusCode() == 200) {
                        result= EntityUtils.toString(response.getEntity(),"utf-8");
                        System.out.println("Response content: " + result);
                    }
                }
                EntityUtils.consume(entity);
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }
}

```
 ## 3.3 ��������
 ��ʹ�ó������httpsʱ��֤�������Ľ����˵��룬������������Ҳ���������ã���������������Ĵ���
```
Exception in thread "main" javax.net.ssl.SSLException: hostname in certificate didn't match: <172.18.132.73> != <xxx.xxx.com>
        at org.apache.http.conn.ssl.AbstractVerifier.verify(AbstractVerifier.java:238)
        at org.apache.http.conn.ssl.BrowserCompatHostnameVerifier.verify(BrowserCompatHostnameVerifier.java:54)
        at org.apache.http.conn.ssl.AbstractVerifier.verify(AbstractVerifier.java:159)
        at org.apache.http.conn.ssl.AbstractVerifier.verify(AbstractVerifier.java:140)
        at org.apache.http.conn.ssl.SSLConnectionSocketFactory.verifyHostname(SSLConnectionSocketFactory.java:301)
```
�����������Ϊ���ʵ�������֤���е�CN��Ӧ���ϣ�ͨ�������ֽ��������

1�����CN��������������ô�����ڿͻ�����������������ʵ��IP��ַ��ӳ��

2���ڴ���������ͨ��ʹ��SSLConnectionSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER��ʵ�֡�
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(...,SSLConnectionSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER)
��ʵ��"����������������֤"��

����ͨ��keytool -v -list�鿴֤����ָ����CN��ͨ��CN��������˵��������
```
*******************************************
keytool -v -list -keystore ./trust.jks
...
����: esight-local-32150
��������: 2019-5-25
��Ŀ����: trustedCertEntry

������: CN=192.168.32.150, OU=bes, O=bessystem, L=chongqing, ST=chongqing, C=CN
������: CN=192.168.32.150, OU=bes, O=bessystem, L=chongqing, ST=chongqing, C=CN
���к�: 783dc37f
��Ч�ڿ�ʼ����: Sat May 25 09:31:53 CST 2019, ��ֹ����: Tue May 22 09:31:53 CST 2029
```
