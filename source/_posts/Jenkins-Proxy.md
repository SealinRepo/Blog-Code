---
title: Jenkins+Nginx 反向代理错误
tags:
  - 工具
  - CI
originContent: ''
categories:
  - 工具
  - 经验
toc: true
date: 2019-07-19 11:03:51
---

在nginx中配置Jenkins服务代理规则, 将请求头转发到Jenkins服务器即可.
```sh
server {
  listen 80;
  server_name jenkins.mydomain.com;
  location / {
    proxy_pass http://jenkins.localnet:8080;
    proxy_read_timeout  90;
    proxy_set_header X-Forwarded-Host $host:$server_port;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
  }
}

```