---
layout: default
parent: 容器部署
title: 服务端
grand_parent: 部署管理
permalink: /部署管理/容器部署/服务端
nav_order: 4
---

### 服务端参数解释：

服务端也是同理，基于Env获取参数，默认Dashboard端如果不需要上传图片，可以把COS去掉，但是我建议配置上，因为漏洞管理最好有图片方便验证。
```
  - MYSQL_ADDRESS=jdbc:mysql://w3aMysql:3306/w3a_soc?characterEncoding=utf-8&useSSL=false
  - MYSQL_USERNAME=root
  - MYSQL_PASSWORD=testw3a
  - REDIS_HOST=w3aRedis
  - REDIS_PORT=6379
  - REDIS_DATABASE=5
  - COS_ACCESSKEY=[自己的accessKeyId]
  - COS_SECRETKEY=[自己的secretkey]
  - COS_REGIONNAME=[对应cos的地区]
  - COS_BUCKETNAME=[bucket]
  - COS_BASEURL=[具体的cos的主地址，记得要带上https://]
```

具体参数解释：***<font color='red'>这里一定要注意，COS_BASEURL一定要完整的地址，因为要展示图片用!</font>***

```
- MYSQL_ADDRESS 数据库地址
- MYSQL_USERNAME 数据库账号
- MYSQL_PASSWORD 数据库密码
- REDIS_HOST redis地址
- REDIS_PORT redis端口
- REDIS_DATABASE redis用的库
- COS_ACCESSKEY AccessKey
- COS_SECRETKEY SecretKey
- COS_REGIONNAME 归属地，比如：ap-beijing
- COS_BUCKETNAME bucket,比如：test-19281913
- COS_BASEURL cos的地址，比如：https://test-19281913.cos.ap-beijing.myqcloud.com 
```
