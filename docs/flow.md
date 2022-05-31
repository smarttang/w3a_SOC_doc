---
layout: default
title: 入侵检测
nav_order: 3
permalink: /入侵检测
---

### 背景描述

W3A SOC工作台有个很特别的地方就是针对流量的入侵检测，之前我们做的更多偏Web日志，然而流量这块有缺失，现在基于Suricate/Snort/zeek(bro)做集成，实现入侵的检测和分析，把真正存在问题的部分给上报到平台，企业只需要在创建的时候一键部署入侵检测的服务，即可和工作台打通，实现攻击行为在工作台上实时查看。
底层的处理逻辑：

- 将产出的流量日志通过filebeat进行日志的采集，并且上报到kafka。
- 工具Agent直接分析Kafka的日志进行捕获，抓取有问题的部分，上报到工作台。
- 同时适配多种工具数据源（Suricate/Snort/zeek）。

### 使用配置

首先，基于filebeat对数据源进行配置，以suricate为例:

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/nids/logs/eve.json
  json.keys_under_root: true
  json.overwrite_keys: true

output.kafka:
  hosts: ["kafka:9092"]
  topic: nidslogs
  required_acks: 1

```

配置完之后，还需要增加服务，把filebeat采集启动起来:(参考docker-compose)

```
  # 采集流量日志到KAFKA
  filebeat3:
    image: docker.elastic.co/beats/filebeat:8.1.3
    entrypoint: "filebeat -e -strict.perms=false"
    volumes:
      - ./filebeat-nids.yml:/usr/share/filebeat/filebeat.yml
      - ./nids/log:/var/nids/logs
    depends_on: 
      - suricate
```

除此之外，suricate也要启动起来，不然无效:(参考docker-compose)

```
  # NIDS-suricate
  suricate:
    image: jasonish/suricata:6.0
    privileged: true
    command: -i eth0
    volumes:
      - ./nids/log:/var/log/suricata
    cap_add:
      - NET_ADMIN
    #  - NET_RAW
      - SYS_NICE
    network_mode: "host"
```

完事之后自行对受保护的业务进行攻击就行，如容器内的服务，然后去工作台就能看到实际的入侵攻击行为。
