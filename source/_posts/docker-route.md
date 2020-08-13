---
title: Docker多宿主机通信
tags:
  - Docker
  - CentOS
originContent: ''
categories:
  - 经验
  - 环境配置
toc: true
date: 2020-08-13 16:30:38
---

# 平台
CentOS

# 方案
添加静态路由
本文中, docker网段如下:
172.17.10.0/24
172.17.11.0/24
分别对应宿主机IP
192.168.1.10
192.168.1.11

## 临时路由
在.10主机添加11的静态路由
```bash
sudo ip route add 172.17.11.0/24 via 192.168.1.11 dev eth0
```
在.11主机添加10的静态路由
```bash
sudo ip route add 172.17.10.0/24 via 192.168.1.10 dev eth0
```

## 永久方案
以上命令执行后立即生效, 在10可以ping通11的docker网段, 反之也一样, 不过重启后失效, 需要再次配置. 可以把上述路由规则写入网卡路由配置文件.
.10执行
```bash
sudo vim /etc/sysconfig/network-scripts/route-eth0
```
添加一行
```bash
172.17.11.0/24 via 192.168.1.11 dev eth0
```

.11执行
```bash
sudo vim /etc/sysconfig/network-scripts/route-eth0
```
添加一行
```bash
172.17.10.0/24 via 192.168.1.10 dev eth0
```