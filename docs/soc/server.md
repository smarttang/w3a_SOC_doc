---
layout: default
parent: 非容器部署
title: 部署W3A服务端
grand_parent: 部署管理
permalink: /部署管理/非容器部署/部署W3A服务端
nav_order: 3
---

### 服务端组成

服务端这块有两个部分，两个Java的Jar包组成，主要承担平台的业务还有就是跟工具、第三方交互：

- 开放平台OpenAPI，用于跟工具交互、外部第三方交互。
- 平台端Dashboard，用于跟前端用户侧交互，我们严格按照前后端分离来搞，尽量不整合到一起，因为那样不好维护，也不优雅。

### 部署（平台侧）

Java端的部署和启动特别简单，如果不是容器，则直接基于Java的入参就行，平台端跟OpenApi端不太一样，平台端具体如下：

```
java -Dspring.datasource.url="jdbc:mysql://localhost:3306/w3a_soc?characterEncoding=utf-8&useSSL=false" \
    -Dspring.datasource.username=root \
    -Dspring.datasource.password=testw3a \
    -Dspring.redis.host=127.0.0.1 \
    -Dspring.redis.port=6379 \
    -Dspring.redis.database=4 \
    -Dspring.cos.accesskey=[cos的AccessKey] \
    -Dspring.cos.secretkey=[cos的secretKey] \
    -Dspring.cos.regionname=[归属地] \
    -Dspring.cos.bucketname=[bucket] \
    -Dspring.cos.baseurl=[具体地址] \
    -Dserver.port=[端口号] \
    -jar w3a-dashboard-service-*.jar
```

解释下这个参数:

```
- Dspring.cos.* 都是腾讯云的配置
- Dspring.datasource.* 都是数据库的配置
- Dspring.redis.* 都是缓存的配置
- Dserver.port 基础的配置
```

具体参数解释：***<font color='red'>这里一定要注意，baseurl一定要完整的地址，因为要展示图片用!</font>***

```
- Dspring.datasource.url 数据库地址
- Dspring.datasource.username 数据库账号
- Dspring.datasource.password 数据库密码
- Dspring.redis.host redis地址
- Dspring.redis.port redis端口
- Dspring.redis.database redis用的库
- Dspring.cos.accesskey AccessKey
- Dspring.cos.secretkey SecretKey
- Dspring.cos.regionname 归属地，比如：ap-beijing
- Dspring.cos.bucketname bucket,比如：test-19281913
- Dspring.cos.baseurl cos的地址，比如：https://test-19281913.cos.ap-beijing.myqcloud.com 
- Dserver.port 服务的对外暴露的端口
```

### 部署（OpenAPI侧）

OpenAPI部署并不复杂，跟上面的一样，但是有点不同的地方是，这里不用配置COS，因为接口API按理来讲不投递图片，更多是投递图片链接，外链，启动方式如下：


```
java -Dspring.datasource.url="jdbc:mysql://localhost:3306/w3a_soc?characterEncoding=utf-8&useSSL=false" \
    -Dspring.datasource.username=root \
    -Dspring.datasource.password=testw3a \
    -Dspring.redis.host=127.0.0.1 \
    -Dspring.redis.port=6379 \
    -Dspring.redis.database=4 \
    -Dserver.port=[端口号] \
    -jar w3a-openapi-service-*.jar
```

参数就跟上面解释一样，我就不重复了，启动成功的效果如下：

```

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::       (v2.5.6-SNAPSHOT)

2022-05-18 15:49:19.568  INFO 50370 --- [           main] c.w.d.s.W3aDashboardServiceApplication   : Starting W3aDashboardServiceApplication v0.0.1-SNAPSHOT using Java 15.0.5 on smarttangdeMacBook-Pro-2.local with PID 50370 (/dashboard/w3a-dashboard-service-v1.0.8.jar started by smarttang in /dashboard)
2022-05-18 15:49:19.569  INFO 50370 --- [           main] c.w.d.s.W3aDashboardServiceApplication   : No active profile set, falling back to default profiles: default
2022-05-18 15:49:20.069  INFO 50370 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
2022-05-18 15:49:20.071  INFO 50370 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data Redis repositories in DEFAULT mode.
2022-05-18 15:49:20.088  INFO 50370 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 7 ms. Found 0 Redis repository interfaces.
2022-05-18 15:49:20.550  INFO 50370 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 9999 (http)
2022-05-18 15:49:20.559  INFO 50370 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-05-18 15:49:20.559  INFO 50370 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.54]
2022-05-18 15:49:20.610  INFO 50370 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-05-18 15:49:20.611  INFO 50370 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1000 ms
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.4.3 
2022-05-18 15:49:22.060  INFO 50370 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 9999 (http) with context path ''
2022-05-18 15:49:22.068  INFO 50370 --- [           main] c.w.d.s.W3aDashboardServiceApplication   : Started W3aDashboardServiceApplication in 3.072 seconds (JVM running for 3.459)
```