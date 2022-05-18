---
layout: default
title: 容器部署
parent: 部署管理
has_children: true
nav_order: 6
---


### docker-compose

这个是我们最优推荐的部署方式，因为简单直接。我们看到很多友商，部署上都特别费劲，不能一键启动，各种参数搞得头昏脑胀，我们想的是：「让我们复杂，让用户简单」。

优先建议，直接基于docker-compose启动，这个最好。在docker-compose上，有些参数只需要微调，就能满足大家需求。后续包括漏洞扫描、Waf之类的，基于Docker-compose一瞬间就能启动，简单直接的很，不需要有太多的部署参数，一瞬间就能搞起来。


### 分析docker-compose的内容

所有W3A开头的项都是W3A SOC必备的内容，其中，Mysql这块全自动化给你导入数据了，你就不用管了，一下就起来，中间不用操作什么，其它的参数需要给大家解释下：


```
  # 分析端
  w3aAgent:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent:v1.0.8
    environment:
      - topic=weblogs
      - kafka=kafka:9092
      - openapi=w3aopenapi:8080
      - modes=analyze

  # 告警端
  w3aAlterAgent:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent:v1.0.8
    environment:
      - topic=weblogs
      - kafka=kafka:9092
      - openapi=w3aopenapi:8080
      - modes=alters

  # Web前端
  w3aFrotend:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-frontend:v1.0.8
    ports:
     - '81:80'
    depends_on:
      - w3aDashboard

  # 后端
  w3aDashboard:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-dashboard:v1.0.8
    ports:
     - '8081:8080'
    environment:
      - MYSQL_ADDRESS=jdbc:mysql://w3aMysql:3306/w3a_soc?characterEncoding=utf-8&useSSL=false
      - MYSQL_USERNAME=root
      - MYSQL_PASSWORD=testw3a
      - REDIS_HOST=w3aRedis
      - REDIS_PORT=6379
      - REDIS_DATABASE=5
     # - COS_ACCESSKEY=[自己的accessKeyId]
     # - COS_SECRETKEY=[自己的secretkey]
     # - COS_REGIONNAME=[对应cos的地区]
     # - COS_BUCKETNAME=[bucket]
     # - COS_BASEURL=[具体的cos的主地址，记得要带上https://]
    depends_on:
      - w3aMysql
      - w3aRedis

  # openAPI
  w3aopenapi:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-openapi:v1.0.8
    ports:
     - '8082:8080'
    environment:
      - MYSQL_ADDRESS=jdbc:mysql://w3aMysql:3306/w3a_soc?characterEncoding=utf-8&useSSL=false
      - MYSQL_USERNAME=root
      - MYSQL_PASSWORD=testw3a
      - REDIS_HOST=w3aRedis
      - REDIS_PORT=6379
      - REDIS_DATABASE=4
    depends_on:
      - kafka
    depends_on:
      - w3aMysql
      - w3aRedis

  # mysql
  w3aMysql:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/mysql:v1
    volumes:
     - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
     - '3306:3306'
    environment:
     - MYSQL_ROOT_PASSWORD=testw3a

  # redis
  w3aRedis:
    image: registry.cn-beijing.aliyuncs.com/aidolphins_com/redis:v1
    ports:
     - '6379:6379'
```
