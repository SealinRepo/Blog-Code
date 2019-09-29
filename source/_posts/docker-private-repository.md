---
title: Docker私有仓库搭建指南
tags:
  - 基础服务
  - Docker
originContent: ''
categories:
  - 技术
  - 环境配置
  - 私有仓库
toc: true
date: 2019-09-29 21:26:21
---

# 简述
docker官方已经贴心的准备了一个私服镜像: registry, 所以我们直接拉下来启动就可以了.
官方镜像默认使用5000端口作为http服务启动端口, 所以我们要使用私服, 需要将5000转发到宿主机.
到这一步以后有一个坑, 启动容器并且转发端口后发现可以通过curl等工具进行请求访问, 但是无法push镜像进去. 是因为容器默认只接受https协议的push请求, 要允许http协议进行提交, 需要进行配置docker启动参数.
以下是搭建过程.
# 获取镜像
```bash
docker pull registry
```

# 配置允许http提交镜像
## 创建配置文件
```bash
vim /etc/docker/daemon.json

# 添加如下内容:
# 192.168.1.100:5000替换为宿主机 IP:端口
{
  "insecure-registries":["192.168.1.100:5000"]
}
```
## 重启Docker
```bash
service docker restart
```

# 启动容器
```bash
docker run -d --name reg -p 5000:5000 registry
```

# 推送镜像到私服
```bash
# 拉取任意一个镜像做测试, 也可以用registry
docker pull hello-world
# 为hello-world镜像打上私服tag
docker tag hello-world 192.168.1.100:5000/hello-world:1.0
# 推送
docker push 192.168.1.100:5000/hello-world:1.0
```