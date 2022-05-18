---
layout: default
parent: 非容器部署
title: 部署W3A界面端
grand_parent: 部署管理
permalink: /部署管理/非容器部署/部署W3A界面端
nav_order: 3
---

### 界面端组成

界面端就一个dist，基于Vue生成的，后续可能改成React，目前主要以Vue为主，因为成本最低原则。
部署界面这端需要Nginx配置文件、静态文件，即可。

### 部署

在开源仓库里有个frontend下有个nginx.conf的文件，写的比较清晰了，这里贴下并解释下：

```
server {
    listen 80;
    server_name  _;
    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    root /usr/share/nginx/html;
    include /etc/nginx/mime.types;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /prod-api/ {
        proxy_pass http://w3aDashboard:8080/;
    }
}
```

这主要需要关注的就只有<font color='red'>proxy_pass</font>这个字段。
只要把这个字段里的内容替换成你的W3A服务端API地址即可，然后把Nginx配置文件放进去/etc/nginx/conf.d/里面就可以了。