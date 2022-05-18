---
layout: default
parent: 容器部署
title: 工具端
grand_parent: 部署管理
permalink: /部署管理/容器部署/工具端
nav_order: 4
---

### 工具端参数解释：

针对Agent端解释下，咱这个都是基于Env来获取的，默认情况下在K8S里是configmap，在docker-compose就是environment，这个具体的概念可以百度下，我就不说了。
不管入参是什么，工具默认优先就是本地环境的Env的参数，如果有设置，就按照Env的走，最后会有判断，因为我们优先给容器环境的同学使用更加便利一些，毕竟容器环境比较省事。

```
  - topic=weblogs 替换成自己的kafka的topic名称
  - kafka=kafka:9092 替换成自己的Kafka地址和端口
  - openapi=w3aopenapi:8080 替换成自己的部署的W3A服务端开放平台对应的域名，或者IP。
  - modes=analyze 具体启动的模块
```

工具这块，直接 <font color='red'>-h</font> 一下就有介绍，对标就明白了。

```
➜  bin ./w3a-soc-agent -h
Usage of ./w3a-soc-agent:
  -A string
    	日志归一的kafka的地址:端口,可以多个,以','分割,默认: kafka:9092 (default "kafka:9092")
  -M string
    	设置启动模式,analyze:攻击分析Agent,alters:告警监测Agent，默认:analyze (default "analyze")
  -O string
    	设置openApi的地址，用于获取规则、上报信息,默认: open.aidolphins.com (default "open.aidolphins.com")
  -T string
    	日志归一的kafka的Topic名称,默认: weblogs (default "weblogs")
```