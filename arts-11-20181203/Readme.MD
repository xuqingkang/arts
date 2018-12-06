In this ARTS, I introduces the FileBeat + Redis + Logstash +��Kibana's install and configration.

# 1.Algorithm
### Question: 



### My solution in python3 is:


# 2.Review
// TODO: I will complete it later.

# 3.Tips���docker˽�о���ֿ⣩

�������Լ������docker˽�еľ���ֿ��������ᷢ�����ǵ�docker push��docker pull�޷��������У������������£����������Ҫ����Ϊdocker�����������Ҫ��ssl���ܣ�������û����ȷ����ssl֤�顣

```
[root@rhel152 ~]# docker pull 192.168.32.150:5000/build_repo/tomcat7
Using default tag: latest
Error response from daemon: Get https://192.168.32.150:5000/v2/: http: server gave HTTP response to HTTPS client
```

�ڲ��Ի���ֱ�Ӵ�ǰ�ȫ��registry��ɣ�����������������--insecure-registryѡ��������õ���˼�����������ڱ���ʹ��docker������ȡ����push�����ʱ�����192.168.32.150:5000�������Բ��÷�SSL���ܵķ�ʽ���ʡ�  ����192.168.32.150:5000��������׼�����е�registry�ĵ�ַ��
```
[root@rhel150 tomcat]# vi /usr/lib/systemd/system/docker.service

[Unit]
Description=Docker Application Container Engine

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
#ExecStart=/usr/bin/dockerd
ExecStart=/usr/bin/dockerd --insecure-registry 192.168.32.150:5000
```
֮������docker����
```
systemctl daemon-reload
systemctl restart docker
```
����registry�������ַ��Ӧ�ϱߵ�--insecure-registry���õĵ�ַ������Ϊ��������������һ��5000�˿ڵ�registry���ϱ����õ�--insecure-registry������Ҫ��Ӧ�����registry��
```
[root@rhel150 tomcat]# docker run -d -p 5000:5000 -v `pwd`/data:/var/lib/registry --restart=always --name registry registry
96dae80878a58e7a7971af4ada1834bf5898933ace17a017ac1d5f2f26c8547c
```
ʹ��docker �ѱ��صľ�������½���registry��ı�ǩ

```
docker tag build_repo/tomcat7 192.168.32.150:5000/build_repo/tomcat7
```
ͨ��docker push�ѱ��ؾ���push���½���registry�ֿ���

```
[root@rhel150 cplus]# docker push 192.168.32.150:5000/build_repo/tomcat7
The push refers to repository [192.168.32.150:5000/build_repo/tomcat7]
edd1867448c0: Pushed 
a097b0127601: Pushed 
0f01533e7e53: Pushed 
0aba197c9276: Pushed 
17ca8aebcfdf: Pushed 
0e20f4f8a593: Pushed 
1633f88f8c9f: Pushed 
46c98490f575: Pushed 
8ba4b4ea187c: Pushed 
c854e44a1a5a: Pushed 
latest: digest: sha256:0f9de34882c7f63504ca41fa4188baa100e6d6f0360207f91fd310da44bad9e8 size: 2413
```



# 4.Share(FLK�����)
����FileBeat��Ϊ��־�ɼ����ɼ������ݷŵ�Redis����(����������kafka)��Logstash+ElasticSearch���ӷ�����Kibana����ҳ��չʾ��

## 4.1 ����˵��
���������תͼ������ʾ��
```
graph LR
Tomcat������־-->FileBeat
FileBeat-->Redis
Redis-->LogStash
LogStash-->ElasticSearch
ElasticSearch-->Kibana

```
���У�

32.150�����������������־��tomcat��FileBeat��Redis

32.152��������LogStash��ElasticSearch

32.1������������������Kibana��
���ݻ�����תͼ������ʾ��
```
graph LR
Tomcat:32.150-->FileBeat:32.150
FileBeat:32.150-->Redis:32.150
Redis:32.150-->LogStash:32.152
LogStash:32.152-->ElasticSearch:32.152
ElasticSearch:32.152-->Kibana:32.1

```

## 4.2 ��װ˵��
### 4.2.1 Redis
Redis�İ�װû��ʲô����ģ�ע�����������ֶξͿ���,��Ҫ�ǿ�����İ�ȫ�������,��Ȼ��������������ʹ����ô�򵥵����롣

```
requirepass 123456
```

### 4.2.2 ElasticSearch
tar -xvf elasticsearch-6.5.1.tar.gz
�༭config/elastic.yml

```
path.data: /home/xuqingkang/ELK/elasticsearch-6.5.1/data
path.logs: /home/xuqingkang/elasticsearch-6.5.1/logs
#network.host: 192.168.0.1
network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
#http.port: 9200
http.port: 9200
```

ֱ������"ELK/elasticsearch-6.5.1/bin/elasticsearch &"
ͨ�������http://192.168.32.152:9200���Բ鿴��Ϣ

### 4.2.3 FileBeat
  �ڸ�ҵ�������ϲ���192.168.32.150~152���������Ŀ�Ľ���32.150�����ϲ���
  tar -xvf filebeat-6.5.1-linux-x86_64.tar.gz
  ��root���еģ���ΪҪ��ȡϵͳ��־��Щ��
  /root/ELK/filebeat-6.5.1-linux-x86_64
  
```
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - - /root/tomcat-6.0.35/logs/localhost_access_log*.txt
#----------------------------- Redis output --------------------------------
output.redis:
  enabled: true
  hosts: ["192.168.32.150"]
  db: "0"
  port: "6379"
  password: "123456"
  key: filebeat
  worker: 1
  timeout: 5s
  max_retries: 3
```
����

```
/root/ELK/filebeat-6.5.1-linux-x86_64/filebeat -e -c /root/ELK/filebeat-6.5.1-linux-x86_64/filebeat.yml &
```

### 4.2.3 Logstash
192.168.32.152 /home/xuqingkang/ELK/logstash-6.5.1
�������redis��ȡ���ݲ������elasticsearch

```
[root@rhel152 config]# vim redis-logstash.conf 

input {
    redis {
        data_type => "list"
        host => "192.168.32.150"
        db => "2"
        port => "6379"
        password => "123456"
        key => "filebeat"

    }
}

output {
    elasticsearch {
        hosts => ["192.168.32.152:9200"]
        index => "filesystem-log-5612-%{+YYYY.MM.dd}"
    }
}
~
```

### 4.2.4 Kibana
windows �޸�config/kibana.yml�ļ� ��Ҫ����elasticsearch��ַ����32.152�����ϵ�elasticsearch������־��ȡ
```
#server.port: 5601
server.port: 5601

#server.host: "localhost"
server.host: 0.0.0.0

#elasticsearch.url: "http://localhost:9200"
elasticsearch.url: "http://192.168.32.152:9200"
```

### 4.2.5��֤
ͨ�� http://192.168.32.1:5601 �鿴Kibana�Ƿ�������ʾ����������־ɾѡ��ͼ�꽨����