# ***����
  Envoy��HaProxy��Nginx����ͬ�����������ṩ������Envoy��ServiceMesh�����������Ϊ�ײ����������ʵ�֣���kubernetes��ϵúܺá�
  
  �������ǳ�������һ��Envoy�Ļ�����װ��ʹ��
# ***��������
  Envoy�����Ŀ��ٿ�ʼ�ֲᣨhttps://www.envoyproxy.io/docs/envoy/latest/start/start�� ���ᵽ���ṩ����õĶ����Ʊ���װ���Ƽ�ֱ��ͨ��docker�������������顣
# ***���Ų���-����
  ���ݹ�����˵��������ֻ��Ҫ�򵥵ĵ�������
  
```
$ docker pull envoyproxy/envoy:latest
$ docker run --rm -d -p 10000:10000 envoyproxy/envoy:latest
$ curl -v localhost:10000
```
���е�һ����������ȡ���񣬵ڶ�������������envoy����������������ӳ����10000�˿ڵ�������10000�˿ڡ�

ʵ�����ڹ��ڵĻ�����Ϊqiang��ԭ���������ǵķ��ʽ�����£����ǵõ���503����
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
�������ͨ��docker exec -it $containerid /bin/sh��cat�����ڵ�/etc/envoy/envoy.yaml�ļ������Կ���������������з��͵�envoy�˿�(Ĭ��10000)������ȫ����ת����google.com
# ***���Ų���-�Ƽ�
�������Ƽ������Ų������£�
## 1����ȡ����

```
$ docker pull envoyproxy/envoy:latest
```
## 2���½�һ��Ĭ������
�����������½�һ��envoy�����ļ������磺envoy-baidu.yaml
�����ļ���������ǹ����ṩ�ľ����е�Ĭ���ļ���ֻ�����������˼����޸ģ�

1��admin.address.socket_address.address����127�޸�Ϊȫ0��ַ��

2�����ص�google.com�����ñ��޸�Ϊ���ص�baidu.com��

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

## 3����������

������ʱ��ָ��-v�������������ϱ༭��yaml�ļ�ӳ�䵽������/etc/envoy/envoy.yaml��
�������£�����ʹ��-v��������������/root/envoy/envoy-baidu.yamlӳ�䵽�����и���/etc/envoy/envoy.yaml
```
docker run --rm -d -p 10000:10000 -p 10001:9901 -v /root/envoy/envoy-baidu.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy:latest
```
## 4����֤
ͨ�������������� curl localhost:10000��֤�����Ƿ�����������ֱ��ͨ�������������������10000�˿ڿ�ҳ���Ƿ�ת����baidu��ҳ��

������������������һ���˿�ӳ�䣬��������10001��������9901(envoy�����˿�)����Ϊ����Ҳ����ͨ�������������������10001�˿ڲ鿴envoy�Ĺ������档����������ӣ�

![image](./images/12-S-envoy-admin.png)