---
layout: default
parent: 日志审计
title: 配置Kafka采集器
permalink: /日志审计/配置Kafka采集器
nav_order: 3
---

## 配置&解释

日志进入Kafka之后，多个端进行消费，本身W3A SOC 的Agent端就会进行读取分析，同时也需要将这个日志放到ES上(主要用于日志查询，在平台里有个Web日志查询的功能，直接联动ES的，实现简单的查询服务，后续会做一些特殊的查询功能放到平台里，具体有需要可以提需求定制也行)，这个时候就可以配置一个Filebeat端进行日志的推送，从Kafka消费到ES里。在docker-compose里我有写好的配置文件可作参考，具体解释下：

- hosts: ["elasticsearch:9200"] 中 「elasticsearch」是实际的es的地址，这里用的是docker-compose的服务名，如果你有自己的ES，可以直接写自己的ES的IP地址或域名，直接替换就行，但是记得一定要能通啊，不然发不过去。
- hosts: ["kafka:9092"] 中「kafka」是实际的Kafka数据源地址，如果有自己的Kafka并且已经配置好[配置Nginx采集器](./配置Nginx采集器)的话，直接就用那个kafka地址就行，我这里写的是docker-compose的服务名称。

```
filebeat.inputs:
- type: kafka
  enabled: true
  hosts: ["kafka:9092"]
  topics: ["weblogs"]
  group_id: "weblogs"

  json.keys_under_root: true
  json.add_error_key: true
  json.overwrite_keys: true

output.elasticsearch:
   hosts: ["elasticsearch:9200"]
   index: "weblog-%{+yyyy.MM.dd}"
setup.template.name: "filebeattest"
setup.template.pattern: "filebeattest-*"
processors:
  # kafka的消息会在message字段，通过该processor将json解析出来
  - decode_json_fields:
      fields: ["message"]
      process_array: true
      max_depth: 1
      target: ""
      overwrite_keys: true
      add_error_key: true
  # 下边这两个处理器根据自身需求设置    
  # 将json中start_time字段的时间放到@timestamp中
  - timestamp:
      # 格式化时间值 给 时间戳 
      field: start_time
      # 使用我国东八区时间  格式化log时间
      timezone: Asia/Shanghai
      layouts:
        # - '2006-01-02 15:04:05'
        - '2006-01-02 15:04:05.999'
      test:
        - '2019-06-22 16:33:51.111'
```
