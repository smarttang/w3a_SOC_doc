---
layout: default
title: W3A SOC
nav_order: 1
description: "W3A SOC 是一个针对于微服务（分布式场景）下的架构治理工具。它可以在开发过程中，帮助架构师、开发人员分析系统间的远程服务依赖情况、数据库依赖、API 依赖等。并根据一些架构治理模型，对现有系统提出改进建议。"
permalink: /
---

<img style="width:660px" title="Run example" alt="Run example" src="/WechatIMG204.png">



<p align="center">
基于日志安全分析做切入，做最好用的「安全运维工作台」。<br>
</p>

![](https://img.shields.io/badge/golang-1.17.2%20-green)
![](https://img.shields.io/badge/openjdk-15.0.5-green)
![](https://img.shields.io/badge/W3A%20SOC-v2.0-green)
![](https://img.shields.io/badge/%E7%AD%89%E7%BA%A7%E4%BF%9D%E6%8A%A4%E4%B8%89%E7%BA%A7-%E6%97%A5%E5%BF%97%E5%AE%A1%E8%AE%A1-green)
![](https://img.shields.io/badge/%E5%91%8A%E8%AD%A6%E7%9B%91%E6%8E%A7-%E9%92%89%E9%92%89-green)
![](https://img.shields.io/badge/%E5%91%8A%E8%AD%A6%E7%9B%91%E6%8E%A7-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1-green)
![](https://img.shields.io/badge/Kubernetes-1.20.6-green)
![](https://img.shields.io/badge/%20docker--compose-1.29.2-green)


**主要特性**
- 日志分析: 基于kafka+GoLang的方式，对采集的Web和系统应用日志进行攻击行为的分析。
- 业务连续性监控: 基于网络的业务连续性监控服务，确定业务是否有中断。
- 告警整合: 实现钉钉、企业微信的联动告警机制。
- 部署支持：docker-compose、Kubernetes。
- 整体架构：基于 Filebeat(采集/清洗) + Kafka(汇聚) + ElasticSearch(检索)
- 技术实现：后端基于Java，前端基于Vue，数据库基于MYSQL。

**目标**
- 满足等保二级、三级的需求，直接部署就能用那种。
- 让客户少花钱，然后也能用，不串联到业务中，对业务0影响。
- 部署简单，一键部署，或者直接随着元豚科技生态自动部署。

**用户画像**
- 安全、运维负责人，全方位了解当前安全现状，做摸底排查
- 安全开发、运维开发，集成全方面的安全、运维自动化平台，提效
- 安全运营，了解、运营整体安全策略、规则、插件
- CTO，清晰的知道安全态势、运维态势

### 关键输出

**Web日志分析**
- 通过Logstash/filebeat采集日志到ES上。
- Golang通过开放平台，获取规则信息针对Kafka的日志进行实时分析。
- 将存在问题的部分直接存到平台里，平台只存落地的攻击日志、记录分析日志数。
- 攻击源IP地址分析，结合IP来源进行分析。
- 输出可以用于封禁的API接口，查可封禁的IP。

**存活监控/篡改监控告警**
- 针对提交的IP进行检测，看是否存活，可以分布式，持续的监测。
- 针对目标进行篡改监控。

**问题告警**
- 针对出现的问题，统一告警输出。
- 支持钉钉、企业微信。


## Docker Compose（快速体验）

```
git clone https://github.com/smarttang/w3a_SOC
cd w3a_SOC/deploy/docker-compose/test-environment/
docker-compose up -d
```

## 漏洞管理图片上传

如果需要用图片上传功能，在腾讯云开一个COS，然后再docker-compose上改改对应的配置即可。


## 服务启动后的效果

```
➜  test-environment git:(master) ✗ docker-compose ps -a
              Name                            Command               State                          Ports                        
--------------------------------------------------------------------------------------------------------------------------------
test-environment_elasticsearch_1   /bin/tini -- /usr/local/bi ...   Up      0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp      
test-environment_filebeat1_1       filebeat -e -strict.perms= ...   Up                                                          
test-environment_filebeat2_1       filebeat -e -strict.perms= ...   Up                                                          
test-environment_kafka_1           /opt/bitnami/scripts/kafka ...   Up      0.0.0.0:29092->29092/tcp, 0.0.0.0:9092->9092/tcp    
test-environment_kibana_1          /bin/tini -- /usr/local/bi ...   Up      0.0.0.0:5601->5601/tcp                              
test-environment_nginx_1           /usr/local/openresty/bin/o ...   Up      0.0.0.0:80->8080/tcp                                
test-environment_w3aAgent_1        /usr/local/sbin/dolphins         Up                                                          
test-environment_w3aAlterAgent_1   /usr/local/sbin/dolphins         Up                                                          
test-environment_w3aDashboard_1    sh -c java $JAVA_OPTS -Dja ...   Up      0.0.0.0:8081->8080/tcp                              
test-environment_w3aFrotend_1      /docker-entrypoint.sh ngin ...   Up      0.0.0.0:81->80/tcp                                  
test-environment_w3aMysql_1        docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp, 33060/tcp                   
test-environment_w3aRedis_1        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp                              
test-environment_w3aopenapi_1      sh -c java $JAVA_OPTS -Dja ...   Up      0.0.0.0:8082->8080/tcp                              
test-environment_zookeeper_1       /opt/bitnami/scripts/zooke ...   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp
```

## 访问Web部分

Docker Compose 启动后，访问：[http://localhost:81/](http://localhost:81/)，就可以看到W3A SOC平台的服务了，登录账号/密码：admin/w3admin

- 记得先「创建应用」，基于docker-compose部署的话，ES地址填「elasticsearch」,端口9200，站点地址写：127.0.0.1，站点日志索引: web*，进去就能看到样例DEMO了。
- 攻击测试针对本地80进行攻击就行，那个就是靶场，扫描的时候自动激活分析。
- 所有的上报会在W3A SOC平台直观展现。
- 默认自带了少量规则，可以激活攻击用。

攻击后就能看到如下图：

![Backend Overview](/assets/screenshots/dashboard.png)

### 备注

- 一直以来都有人问我到底还是否继续更新迭代，不断有人加我，还给我鼓励，时隔六年，沉淀积累之后，准备再做一套新的，可以持续更新维护的版本。
- 诸位发现问题请直接提交issue，如果有定制化需求，麻烦支持下收费版本，在内部推动下元豚科技的产品啥的，感激不尽。
- 项目元豚科技接管之后，起码不会荒废，只要公司一直在，项目就会更新。
- 旧的版本在release，那个版本是开源的，代码开放，有需要自行提取。

### 客户体验计划

- 欢迎种子用户，早期的种子用户后期可以优先体验商业版功能，欢迎扫码加我微信，我后面拉群。
- 欢迎提意见和场景，我们想办法满足，如果有好的日志样本也可以提供给我们。

### 加我

<img style="width:200px" title="Run example" alt="Run example" src="/wechat.png">
