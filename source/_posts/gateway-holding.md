---
title: Spring Gateway网关长时间不返回数据问题
tags:
  - JAVA
  - SpringCloud
  - Gateway
categories:
  - 技术
  - JAVA
  - 微服务
toc: true
date: 2021-07-27 13:30:44
---

# 问题

SpringCloud项目经过网关后,有时候会出现服务返回了数据, 但是客户端收不到数据, 持续等待的情况. 通过查看请求头, 发现除了大量的`forward`头外, 有一个`expect`不认识, 通过查阅资料发现作用如下:

> 在使用curl做POST的时候（比如通过PHP发起post请求），当要POST的数据大于1024字节的时候，curl并不会直接就发起POST请求, 而是会分为2步：
> 
> 发送一个请求，包含一个Expect:100-continue，询问Server使用愿意接受数据。接收到Server返回的100-continue应答以后, 才把数据POST给Server。

# 解决

参考[官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.3.RELEASE/single/spring-cloud-gateway.html#_removerequestheader_gatewayfilter_factory), 有以下解决方案:

在服务配置添加过滤器:

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remove-header
      uri: https://example.org
      filters:
      - RemoveRequestHeader=Expect
```

添加全局过滤器

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - RemoveRequestHeader=Expect
```

