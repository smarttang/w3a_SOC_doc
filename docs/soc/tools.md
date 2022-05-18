---
layout: default
parent: 非容器部署
title: 部署W3A工具端
grand_parent: 部署管理
permalink: /部署管理/非容器部署/部署W3A工具端
nav_order: 3
---
### 工具端组成

工具端这块，主要就1个端负责所有的事，但是这个端基于参数化区分启动，整体基于Golang开发，一方面是安全性更好，其次就是它小巧灵动，还有就是更加贴近云原生，后续可能会整体都往Golang上走，因为Golang确实很舒服，这个端目前实现如下：

- 跟OpenAPI整体联动，处理各种交互。
- 实现日志的分析、攻击的识别上报。
- 实现告警的Alter，还有通知。
- 默认启动失败5次重试，关键的部分都有重试机制。
- ...(未来无限的可能)

### 命令行

直接 <font color='red'>-h</font> 一下就有介绍，比较简单粗暴。

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

启动Web日志审计的具体命令如下:

```
./w3a-soc-agent -M analyze -O openapi.my.com -T weblogs -A kafka.my.com:9092
```

具体参数解释：

```
-O openapi.my.com 替换成自己的部署的W3A服务端开放平台对应的域名，或者IP。
-T weblogs 替换成自己的kafka的topic名称
-A kafka.my.com:9092 替换成自己的Kafka地址和端口
-M analyze 具体启动的模块(日志分析模块)
```

启动规则告警的具体命令如下：

```
./w3a-soc-agent -M alters -O openapi.my.com
```

具体参数解释：

```
-O openapi.my.com 替换成自己的部署的W3A服务端开放平台对应的域名，或者IP。
-M alters 具体启动的模块(告警模块)
```

### 异常情况

- 如果出现以下异常情况，不用慌张，就是你的Kafka没配置正确，访问不了。

```
2022/05/18 16:10:14 kafka: client has run out of available brokers to talk to (Is your cluster reachable?)
2022/05/18 16:10:18 kafka: client has run out of available brokers to talk to (Is your cluster reachable?)
```

- 如果出现中文错误，一定是接口数据没导入进去，或者链接不上API，这个得看看MYSQL是否导入或者地址有没有错。

```
➜  bin ./w3a-soc-agent -M alters -O openapi.my.com                         
2022/05/18 16:17:45 启动告警端成功,standby中...
2022/05/18 16:18:01 获取告警规则ID列表失败,请稍后再试!
2022/05/18 16:19:00 获取告警规则ID列表失败,请稍后再试!
2022/05/18 16:20:00 获取告警规则ID列表失败,请稍后再试!
```