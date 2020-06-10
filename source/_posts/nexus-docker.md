---
title: Nexus3作为Docker仓库登陆401
tags:
  - 基础服务
  - Docker
originContent: ''
categories:
  - 经验
toc: true
date: 2020-06-01 14:16:56
---

# 问题：
配好了docker仓库，但是在客户端使用
```bash
docker login -u admin -p admin 服务器地址:端口
```
提示认证失败，401.
![image.png](/images/2020/06/01/1870e0f0-a3cf-11ea-90bc-1986d0282c7a.png)
经过反复验证用户名密码无误。

# 解决
![image.png](/images/2020/06/01/4009e710-a3cf-11ea-90bc-1986d0282c7a.png)

# 牛逼！
![image.png](/images/2020/06/01/6a4cd780-a3cf-11ea-90bc-1986d0282c7a.png)