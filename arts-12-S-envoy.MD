# ***介绍
  Envoy和HaProxy、Nginx属于同类别的软件，提供代理。Envoy在ServiceMesh网格计算中作为底层请求代理的实现，与kubernetes结合得很好。
  
  今天我们初步来看一下Envoy的基本安装和使用
# ***关于入门
  Envoy官网的快速开始手册（https://www.envoyproxy.io/docs/envoy/latest/start/start） 中提到不提供编译好的二进制报安装，推荐直接通过docker快速运行来体验。
# ***入门步骤-官网
  根据官网的说明，运行只需要简单的单个命令
  
```
$ docker pull envoyproxy/envoy:latest
$ docker run --rm -d -p 10000:10000 envoyproxy/envoy:latest
$ curl -v localhost:10000
```
其中第一条命令是拉取镜像，第二天命令是运行envoy容器，本地宿主机映射了10000端口到容器的10000端口。

实际上在国内的环境因为qiang的原因，所以我们的访问结果如下：我们得到到503错误。
```
* About to connect() to localhost port 10000 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 10000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:10000
> Accept: */*
> 
< HTTP/1.1 503 Service Unavailable
< content-length: 91
< content-type: text/plain
< date: Wed, 20 Mar 2019 12:22:58 GMT
< server: envoy
< 
* Connection #0 to host localhost left intact
upstream connect error or disconnect/reset before headers. reset reason: connection failure
```
如果我们通过docker exec -it $containerid /bin/sh后cat容器内的/etc/envoy/envoy.yaml文件，可以看到里边配置了所有发送到envoy端口(默认10000)的请求全部被转发到google.com
# ***入门步骤-推荐
所以我推荐的入门步骤如下：
## 1，拉取镜像

```
$ docker pull envoyproxy/envoy:latest
```
## 2，新建一个默认配置
在宿主机上新建一个envoy配置文件，比如：envoy-baidu.yaml
以下文件大多数都是官网提供的镜像中的默认文件，只不过我们做了几点修改：

1，admin.address.socket_address.address被从127修改为全0地址；

2，负载到google.com的配置被修改为负载到baidu.com。

```
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: www.baidu.com
                  cluster: service_baidu
          http_filters:
          - name: envoy.router
  clusters:
  - name: service_baidu
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_baidu
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.baidu.com
                port_value: 443
    tls_context:
      sni: www.baidu.com
```

## 3，运行容器

启动的时候指定-v参数把宿主机上编辑的yaml文件映射到容器的/etc/envoy/envoy.yaml。
命令如下，我们使用-v参数把宿主机的/root/envoy/envoy-baidu.yaml映射到容器中覆盖/etc/envoy/envoy.yaml
```
docker run --rm -d -p 10000:10000 -p 10001:9901 -v /root/envoy/envoy-baidu.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy:latest
```
## 4，验证
通过宿主机上运行 curl localhost:10000验证请求是否正常。或者直接通过浏览器访问宿主机的10000端口看页面是否转发到baidu首页。

由于上面我们新增了一个端口映射，宿主机的10001到容器的9901(envoy管理端口)，因为我们也可以通过浏览器访问宿主机的10001端口查看envoy的管理界面。像下面的样子：

![image](../images/12-S-envoy-admin.png)
