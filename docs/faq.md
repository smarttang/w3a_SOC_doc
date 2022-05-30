---
layout: default
title: FAQ
nav_order: 80
permalink: /W3A
---

## 问题清单

Q1：发现采集日志的Agent报exit42 如下图：
```
CONTAINER ID   IMAGE                                                                   COMMAND                  CREATED          STATUS                      PORTS                                                  NAMES
38ef14e98540   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-frontend:v1.0.12    "/docker-entrypoint.…"   44 minutes ago   Up 44 minutes               0.0.0.0:81->80/tcp                                     docker-compose-m1_w3aFrotend_1
d59f2f330d13   docker.elastic.co/beats/filebeat:8.1.3                                  "filebeat -e -strict…"   44 minutes ago   Up 44 minutes                                                                      docker-compose-m1_filebeat2_1
84ffe3795e59   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-dashboard:v1.0.12   "sh -c 'java $JAVA_O…"   44 minutes ago   Up 44 minutes               0.0.0.0:8081->8080/tcp                                 docker-compose-m1_w3aDashboard_1
da1dca0b1cad   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-openapi:v1.0.12     "sh -c 'java $JAVA_O…"   44 minutes ago   Up 44 minutes               0.0.0.0:8083->8080/tcp                                 docker-compose-m1_w3aopenapi_1
89d1cf84817c   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-workapi:v1.0.12     "sh -c 'java $JAVA_O…"   44 minutes ago   Up 44 minutes               0.0.0.0:8082->8080/tcp                                 docker-compose-m1_w3aworkapi_1
f71cc6fa6d07   bitnami/kafka:3.1                                                       "/opt/bitnami/script…"   44 minutes ago   Up 44 minutes               0.0.0.0:9092->9092/tcp, 0.0.0.0:29092->29092/tcp       docker-compose-m1_kafka_1
d0d2307c347d   docker.elastic.co/beats/filebeat:8.1.3                                  "filebeat -e -strict…"   44 minutes ago   Up 44 minutes                                                                      docker-compose-m1_filebeat1_1
98e66a410ab1   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent:v1.0.12       "/usr/local/sbin/dol…"   44 minutes ago   Up 44 minutes                                                                      docker-compose-m1_w3aAssetsAgent_1
7268760ff12a   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent:v1.0.12       "/usr/local/sbin/dol…"   44 minutes ago   Up 44 minutes                                                                      docker-compose-m1_w3aCodeScanAgent_1
97e8eb9a0480   bitnami/zookeeper:3.8                                                   "/opt/bitnami/script…"   44 minutes ago   Up 44 minutes               2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, 8080/tcp   docker-compose-m1_zookeeper_1
16b1add0a6c4   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent:v1.0.12       "/usr/local/sbin/dol…"   44 minutes ago   Exited (0) 42 minutes ago                                                          docker-compose-m1_w3aAnalysisAgent_1
23f301f24ccc   registry.cn-beijing.aliyuncs.com/aidolphins_com/redis:v1                "docker-entrypoint.s…"   44 minutes ago   Up 44 minutes               0.0.0.0:6379->6379/tcp                                 docker-compose-m1_w3aRedis_1
c6a73fcdcae3   docker.elastic.co/elasticsearch/elasticsearch:8.1.3-arm64               "/bin/tini -- /usr/l…"   44 minutes ago   Up 44 minutes               0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp         docker-compose-m1_elasticsearch_1
fd802e5c6d37   registry.cn-beijing.aliyuncs.com/aidolphins_com/mysql:v1                "docker-entrypoint.s…"   44 minutes ago   Up 44 minutes               0.0.0.0:3306->3306/tcp, 33060/tcp                      docker-compose-m1_w3aMysql_1
ae468803f3ce   registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent:v1.0.12       "/usr/local/sbin/dol…"   44 minutes ago   Up 44 minutes                                                                      docker-compose-m1_w3aAlterAgent_1
8394e558b14f   openresty/openresty:alpine                                              "/usr/local/openrest…"   44 minutes ago   Up 44 minutes               0.0.0.0:80->8080/tcp                                   docker-compose-m1_nginx_1

```
答案：
1. 这种情况百分之99都是因为kafka的问题，因为kafka在这个地方就是一个测试环境的配置，生产用的话，建议自己再搭建。
2. 排查Kafka的方式，直接docker-compose logs -f kafka，基本啥能看到以下情况：(基本上就是kafka没起来)

```
➜  docker-compose-m1 git:(dev) ✗ docker logs -f f71cc6fa6d07
kafka 12:30:24.56 
kafka 12:30:24.64 Welcome to the Bitnami kafka container
kafka 12:30:24.71 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-kafka
kafka 12:30:24.79 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-kafka/issues
kafka 12:30:24.89 
kafka 12:30:24.99 INFO  ==> ** Starting Kafka setup **
kafka 12:30:26.95 WARN  ==> You set the environment variable ALLOW_PLAINTEXT_LISTENER=yes. For safety reasons, do not use this flag in a production environment.
kafka 12:30:27.29 INFO  ==> Initializing Kafka...
kafka 12:30:27.45 INFO  ==> No injected configuration files found, creating default config files
kafka 12:30:31.62 INFO  ==> ** Kafka setup finished! **

kafka 12:30:31.94 INFO  ==> ** Starting Kafka **

```

3. Agent端大概率是这样的：（默认有重试机制，所以会出现这种提示）

```
2022/05/30 12:30:25 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
2022/05/30 12:30:29 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
2022/05/30 12:30:38 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
2022/05/30 12:31:00 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
2022/05/30 12:31:56 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
2022/05/30 12:31:56 重试了5次失败,W3A的Agent无法启动,请稍后再试,或联系开发者!
2022/05/30 13:14:42 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
2022/05/30 13:14:46 kafka: client has run out of available brokers to talk to: dial tcp 172.27.0.12:9092: connect: connection refused
```

这个时候，解决办法如下：

```
docker-compose stop kafka
docker-compose rm kafka
docker-compose up -d kafka
docker-compose restart w3aAnalysisAgent
```

Q2: 发现怎么都起不来，无法启动成功!
答案：
1. 先检查是不是空间不够了，如果是，请释放空间.(docker image prune / docker image prune -a)

```
➜  docker-compose-m1 git:(dev) ✗ docker image prune 
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-dashboard@sha256:c028c75acd0cf8e802842bb0f80d3a1e8f451a62f1e4dd28f441d66b7bb31ba6
deleted: sha256:fd8accb0efa54feae18f68c9dd7c1662b2017c2f45a303533675a2b40005ce88
untagged: registry.cn-beijing.aliyuncs.com/aidolphins_com/w3a-agent@sha256:2f11ce91fdd15ef12633b6ae12089c2112265f2712dab30e5eb66b3388433364
deleted: sha256:7085831945d786fbfaddc8d1393c0780566e8fdd1ec38f9193c5c6df8ccb212d
...太多了就不贴了
```

2. 清理system的空间:(docker system prune)




