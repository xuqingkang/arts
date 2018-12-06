In this ARTS, I introduces the FileBeat + Redis + Logstash +　Kibana's install and configration.

# 1.Algorithm
### Question: 



### My solution in python3 is:


# 2.Review
// TODO: I will complete it later.

# 3.Tips（搭建docker私有镜像仓库）

我们在自己环境搭建docker私有的镜像仓库是往往会发现我们的docker push和docker pull无法正常运行，场景错误如下：这个错误主要是因为docker镜像操作必须要走ssl加密，而我们没有正确配置ssl证书。

```
[root@rhel152 ~]# docker pull 192.168.32.150:5000/build_repo/tomcat7
Using default tag: latest
Error response from daemon: Get https://192.168.32.150:5000/v2/: http: server gave HTTP response to HTTPS client
```

在测试环境直接搭建非安全的registry便可，在启动参数中增加--insecure-registry选项。此项配置的意思是允许我们在本地使用docker命令拉取或者push镜像的时候，针对192.168.32.150:5000镜像库可以采用非SSL加密的方式访问。  这里192.168.32.150:5000就是我们准备运行的registry的地址。
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
之后重启docker服务
```
systemctl daemon-reload
systemctl restart docker
```
启动registry，这个地址对应上边的--insecure-registry设置的地址。正因为我们这里启动了一个5000端口的registry，上边配置的--insecure-registry参数需要对应到这个registry。
```
[root@rhel150 tomcat]# docker run -d -p 5000:5000 -v `pwd`/data:/var/lib/registry --restart=always --name registry registry
96dae80878a58e7a7971af4ada1834bf5898933ace17a017ac1d5f2f26c8547c
```
使用docker 把本地的镜像打上新建的registry库的标签

```
docker tag build_repo/tomcat7 192.168.32.150:5000/build_repo/tomcat7
```
通过docker push把本地镜像push到新建的registry仓库中

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



# 4.Share(FLK环境搭建)
采用FileBeat作为日志采集，采集的数据放到Redis缓冲(生产建议上kafka)，Logstash+ElasticSearch复杂分析，Kibana负责页面展示。

## 4.1 环境说明
数据组件流转图如下所示：
```
graph LR
Tomcat访问日志-->FileBeat
FileBeat-->Redis
Redis-->LogStash
LogStash-->ElasticSearch
ElasticSearch-->Kibana

```
其中：

32.150机器部署输出访问日志的tomcat、FileBeat和Redis

32.152机器部署LogStash和ElasticSearch

32.1机器是宿主机，部署Kibana。
数据机器流转图如下所示：
```
graph LR
Tomcat:32.150-->FileBeat:32.150
FileBeat:32.150-->Redis:32.150
Redis:32.150-->LogStash:32.152
LogStash:32.152-->ElasticSearch:32.152
ElasticSearch:32.152-->Kibana:32.1

```

## 4.2 安装说明
### 4.2.1 Redis
Redis的安装没有什么特殊的，注意配置如下字段就可以,主要是跨机器的安全密码访问,当然生产环境不建议使用这么简单的密码。

```
requirepass 123456
```

### 4.2.2 ElasticSearch
tar -xvf elasticsearch-6.5.1.tar.gz
编辑config/elastic.yml

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

直接启动"ELK/elasticsearch-6.5.1/bin/elasticsearch &"
通过浏览器http://192.168.32.152:9200可以查看信息

### 4.2.3 FileBeat
  在各业务主机上部署：192.168.32.150~152，这里测试目的仅在32.150机器上部署。
  tar -xvf filebeat-6.5.1-linux-x86_64.tar.gz
  用root运行的，因为要获取系统日志这些。
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
启动

```
/root/ELK/filebeat-6.5.1-linux-x86_64/filebeat -e -c /root/ELK/filebeat-6.5.1-linux-x86_64/filebeat.yml &
```

### 4.2.3 Logstash
192.168.32.152 /home/xuqingkang/ELK/logstash-6.5.1
配置其从redis获取数据并输出到elasticsearch

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
windows 修改config/kibana.yml文件 主要配置elasticsearch地址。从32.152机器上的elasticsearch进行日志获取
```
#server.port: 5601
server.port: 5601

#server.host: "localhost"
server.host: 0.0.0.0

#elasticsearch.url: "http://localhost:9200"
elasticsearch.url: "http://192.168.32.152:9200"
```

### 4.2.5验证
通过 http://192.168.32.1:5601 查看Kibana是否正常显示，并进行日志删选和图标建立。
