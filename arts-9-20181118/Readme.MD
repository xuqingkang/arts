In this ARTS, I introduces the SNMP's configration and program sample.

# 1.Algorithm
### Question: 



### My solution in python3 is:


# 2.Review
// TODO: I will complete it later.

# 3.Tips



# 4.Share(SNMP���ʵս)

## 4.1 �����е�SNMP
2013���ʱ����Ϊ�м��Ͷ�꣬�ںܶ��б��ļ����Ҫ���м��֧��SNMPЭ��Ĺ����ͼ�أ����Ե�ʱ��ͬ��һ�𿪷����м����SNMPģ�顣��������ͨ��SNMP����м�����̳߳ء���Դ�ء�WEB����ͳ�ƣ�����200��400��404����Ӧ��Ӧ�ı���ͳ�ƣ��ȣ�Ҳ��������һЩ��ֵ����ָ�곬����ֵʱͨ��SNMP TRAP������Ϣ��NMS(����ϵͳ)��

��ʱ���νӴ�SNMP,��SNMP PDU��OID��MIB��Community��Agent�ȸ������ʾ���Ҫ�񲹡������м���ĽǶȣ��м��ʵ����һ������Agent����Ҫ����NMSϵͳ���м����ָ���أ���鿴�ڴ�ʹ��������̳߳�ʹ������ȡ�NMS���м������SNMP GET��Ϣ��Ȼ��ʹ�����Ƕ����OID��ȡ��Ӧ����Ϣ��

������ʾ�����Ǹ��Լ��Ĳ�Ʒ������һ��oid��root�ڵ㣬Ϊ1.3.6.1.4.1.218��Ȼ��˾�����Ʒ�����м��appserver��Ʒ������218�����321�ڵ㣬֮��jvmRuntimeTable���ǹ���JVM�ڴ�ʹ�õ�OID��Ϣ������ļ��Ƚϼ򵥣���������������ΪMIB�ļ����Ƕ���OID������Ĺ�ϵ��Ȼ���м���Ľ����У������յ���ͬ��OIDʱ��Ҫ�Ӽ�����з��ض�Ӧ��ֵ��

```
--  GENERATED MIB FILE  --
BES-MIB DEFINITIONS ::=
BEGIN
  IMPORTS
    DisplayString FROM RFC1213-MIB
    OBJECT-TYPE, MODULE-IDENTITY,Counter64, NOTIFICATION-TYPE
      FROM SNMPV2-SMI;
      
  appserver       MODULE-IDENTITY
    LAST-UPDATED  "Fri Mar 08 11:24:31 CST 2013"
    ORGANIZATION  "BES Software Corporation"
    CONTACT-INFO  "www.bessystem.com"
    DESCRIPTION   "BES"
    ::=           {bes 321}

  bes       OBJECT IDENTIFIER ::= { iso 3 6 1 4 1 218 }

  appserver       OBJECT IDENTIFIER ::= { bes 321 }
  
  jvmRuntimeTable       OBJECT-TYPE
      SYNTAX          SEQUENCE OF JVMRuntimeEntryType
      MAX-ACCESS      not-accessible
      STATUS          current
      DESCRIPTION     "This table contains runtime information about java virtual machine(JVM)"
      ::=             { appserver 2 }

  jvmRuntimeEntry       OBJECT-TYPE
      SYNTAX          JVMRuntimeEntryType
      MAX-ACCESS      not-accessible
      STATUS          current
      DESCRIPTION     "Entry of jvmRuntimeTable"
      INDEX           { jvmRuntimeHeapsizeCurrent }
      ::=             { jvmRuntimeTable 1 }

  JVMRuntimeEntryType ::=
      SEQUENCE {
          jvmRuntimeHeapsizeCurrent  Counter64,
          jvmRuntimeHeapsizeHighWatermark  Counter64,
          jvmRuntimeHeapsizeLowWatermark  Counter64,
          jvmRuntimeHeapsizeUnit  OctetString,
          jvmRuntimeHeapsizeLowerBound  Counter64,
          jvmRuntimeHeapsizeUpperBound  Counter64,
      }

  jvmRuntimeHeapsizeCurrent       OBJECT-TYPE
      SYNTAX          Counter64
      MAX-ACCESS      read-only
      STATUS          current
      DESCRIPTION     "The current JVM heap size"
      ::=             { jvmRuntimeEntry 2 }
```

## 4.2 ���ڻع���SNMP
������Ϊ��MIB��OID��һЩ��⣬������1.3.6.1.4.1������һ��218�ڵ����Ϊ��˾��SNMP OID���ڵ��ˣ����������������Ҫ��ΪNMS�Ӹ��������豸��ͨ��SNMP��ȡ��Ϣ����֪����ȷ�Ķ����Լ���˾��SNMP OID���ڵ�������Ƿ��ʼ���iana-mib@isi.edu�������롣

### 4.2.1 NMS����ģ��
�����Ҫģ��NMSϵͳ����SNMP GET���󣬿���ʹ��SNMP4J��Դ�齨���С���������ԵĻ���Ҫ���ݲ�ͬ�������豸�����ã�������H3C S2610������Ϊ����

1,��ʼ��������SNMPģ�飬����ͼ��ʾ:

![image](../images/9-S-snmp-h3c-openSNMP.png)

2����Ҫע��֧��SNMPV3ʱ��Ҫ������û���Ϣ��SNMPV3��ǿ���ʰ�ȫ��Ҳ��һ�����ԣ�
SNMPV3�е���֤�ͼ��ܶ��ǿ�ѡ�

![image](../images/9-S-snmp-h3c-snmpv3-user.png)

3������ʹ�����µ�snmpwalk����ȿ��Ի�ȡ��������Ϣ��
snmpwalk -v 3 -u xuqingkang -l authPriv -a MD5 -A xuqingkang -x DES -X xuqingkang 192.168.0.234
����-A����֤���룬-X�Ǽ������룬-u���û�����

4��MIB��Ϣ(Ҳ����OID��Ӧ��������ʲô����)��Ҫ��ѯ�豸�����ۺ��ȡ

