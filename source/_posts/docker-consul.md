---
title: 单实例Docker部署Consul集群
tags:
  - 基础服务
  - Docker
  - Consul
categories:
  - 技术
  - 微服务
  - 注册中心
toc: true
date: 2019-09-12 21:31:30
---

# 获取consul镜像
```bash
docker pull consul
```

# 运行Consul服务
单机部署Docker内已实现互通, 不用人为干预通信端口转发, 启动后地址为172.17.0.2, 172.17.0.3 .... 只需要将8500端口转发出来即可.
多机部署需要将Consul需要的端口转发到宿主机, 以下是Consul运行用到的端口

|端口|说明|
|-|-|
|8300|8300 端口用于服务器节点。客户端通过该端口 RPC 协议调用服务端节点。服务器节点之间相互调用|
|8301|8301 端口用于单个数据中心所有节点之间的互相通信，即对 LAN 池信息的同步。它使得整个数据中心能够自动发现服务器地址，分布式检测节点故障，事件广播（如领导选举事件）。|
|8302|8302 端口用于单个或多个数据中心之间的服务器节点的信息同步，即对 WAN 池信息的同步。它针对互联网的高延迟进行了优化，能够实现跨数据中心请求。|
|8500|8500 端口基于 HTTP 协议，用于 API 接口或 WEB UI 访问。|
|8600|	8600 端口作为 DNS 服务器，它使得我们可以通过节点名查询节点信息。|

```bash
# 任选一个服务打开Web管理界面
docker run -d --name consul1 -p 8500:8500 -e CONSUL_BIND_INTERFACE=eth0 consul agent --node=node1 --server=true --bootstrap-expect=3 --client=0.0.0.0 -ui

# 将后续打开的服务加入集群
docker run -d --name consul2 -e CONSUL_BIND_INTERFACE=eth0 consul agent --node=node2 --server=true --client=0.0.0.0 --join 172.17.0.2
docker run -d --name consul3 -e CONSUL_BIND_INTERFACE=eth0 consul agent --node=node3 --server=true --client=0.0.0.0 --join 172.17.0.2
```
# 完成
控制台地址: http://IP:8500