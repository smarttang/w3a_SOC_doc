---
layout: default
parent: 日志审计
title: 配置Nginx采集器
permalink: /日志审计/配置Nginx采集器
nav_order: 2
---

## 演进&洞察

现在经过几年的技术演进，我们不再自己造轮子自己写这个采集端，因为相对来讲不太稳健：

- 容易崩，一旦崩了会带来各种问题，还得维护、排错，也不知道哪个地方出问题，说不定是语法，或者各种奇奇怪怪的场景。
- 中间层不太稳健，量大的就出各种问题，取决于机器性能和日志处理的瓶颈。

最好的方式还是基于现成的采集来实现，选型中有Flume、Logstash，也有Filebeat这些，看了下对比：[参考指南](https://blog.csdn.net/a934079371/article/details/107328732)最终，因为分析工具是基于Golang的，而且我们整个工作台也是奔着云原生的方向去走，所以，我们在采集这端优先用Filebeat，主要洞察是：

- 针对中小型的客户，比较好融合，基于Kafka实现，订阅消费的模式比较容易解耦。（正正合适，对业务0嵌入）
- 针对中大型的客户，如日志量到PB级一天，这种基于Hadoop集群的模式，可以单独再聊，不在本文范畴里。（从画像来讲，这类客户极其罕见，真有，我们可以定制）
- Kafka相对稳健成熟的基础设施，不需要过多对比，什么性能指标、各种推理，客户不用有过多的顾虑，怕用一个不成熟的第三方，导致各种问题，怕自研的采集器要么不维护，要么不稳定，中立挺好，kafka大家都熟，没有顾虑。
- 兼容容器集群（K8S）的场景，在容器中基于Filebeat采集有天然优势，这个在后续融合HIDS(平台后续具备的能力)的时候，就更加简单容易，天然结合。

## 配置&解释

从docker-compose的配置文件上，就能看到这个采集的用法。

### filebeat基础配置

```
filebeat.inputs:
- type: log
  enabled: true
  fields:
    source: nginx_access_log
  paths:
    - /var/log/nginx/access*.log
  json.keys_under_root: true
  json.overwrite_keys: true

output.kafka:
  hosts: ["kafka:9092"]
  topic: weblogs
  required_acks: 1
```

这端主要做的是：

- 把Nginx日志采集到Kafka中，进行存储，用于Golang写的分析工具对日志数据进行分析。
- 要把Nginx的日志基于Json方式格式化好放进去，好让工具解析。
- hosts: ["kafka:9092"] 这个地方的「kafka」可以替换成kafka的域名、或者ID，如果有自建的就写自建的就行了，我这里写的是docker-compose里的服务名
- topic: weblogs 这个「weblogs」可以替换成你想要的日志名称，我建议就用这个，因为默认就是分析Web日志。

### Nginx基础配置

在nginx这一端，也要做些轻微改动，如果本身就是JSON格式，对一下配置就可以，主要看 **log_format** 的 **log_json**这个部分，因为里面的字段都是对应的，如果已经有了Nginx的配置，就加一个这个就行。

这里我们自己用的是openresty，用原生的Nginx也可以的，不限制具体用哪个。

```
worker_processes  auto;
#error_log         "/opt/bitnami/nginx/logs/error.log";
pid               "/tmp/nginx.pid";

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format    main '$remote_addr - $remote_user [$time_local]'
                       '"$request" $status  $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for" "$http_host" "$request_time" "$upstream_response_time" "$request_body"';
    #access_log    "/opt/bitnami/nginx/logs/access.log" main;
    log_format    log_json '{"@timestamp": "$time_local","request_body":"$request_body","remote_addr":"$remote_addr","http_host":"$http_host","request":"$request","status":"$status","body_bytes_sents":"$body_bytes_sent","req_time":"$request_time","http_user_agent":"$http_user_agent", "http_referer":"$http_referer", "request_method":"$request_method", "http_x_forwarded_for":"$http_x_forwarded_for"}';
    #access_log    "/usr/local/openresty/nginx/logs/access.log" log_json;
    add_header    X-Frame-Options SAMEORIGIN;

    client_body_temp_path  "/tmp/client_body" 1 2;
    proxy_temp_path        "/tmp/proxy" 1 2;
    fastcgi_temp_path      "/tmp/fastcgi" 1 2;
    scgi_temp_path         "/tmp/scgi" 1 2;
    uwsgi_temp_path        "/tmp/uwsgi" 1 2;
    client_body_buffer_size 1024k;
    fastcgi_buffers 32 16k;
    sendfile           on;
    gzip               on;
    gzip_http_version  1.0;
    gzip_comp_level    2;
    gzip_proxied       any;
    gzip_types         text/plain text/css application/javascript text/xml application/xml+rss;
    keepalive_timeout  65;
    client_max_body_size 80M;
    server_tokens off;

    #include  "/opt/bitnami/nginx/conf/server_blocks/*.conf";

    # HTTP Server
    server {
        lua_need_request_body on;
    	access_log    "/usr/local/openresty/nginx/logs/access.log" log_json;
        listen  8080;
        server_name  api.example.com;

        location / {
            proxy_pass "http://www.aidolphins.com";
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    root html;
	    index index.html index.htm;
        }
    }
}
```
