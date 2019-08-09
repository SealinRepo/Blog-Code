---
title: 解决Spring Boot在IDEA中热部署失效的问题
tags:
  - 工具
  - IDEA
  - Spring
originContent: ''
categories:
  - 工具
  - 经验
toc: true
date: 2019-08-09 19:49:29
---

在IDEA中开发Spring Boot应用, 添加了spring-boot-devtools依赖发现热部署实际上并没有生效, 原因是IDEA代码自动保存后并没有触发编译.
知道问题后解决也简单, 完整步骤如下:

# 添加依赖
```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>

```
# 配置依赖
```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```
# IDEA 配置
## 自动编译
配置路径:
> File -> Settings -> Compiler
>勾选自动编译选项

![image.png](/images/2019/08/09/eaf622c0-ba9a-11e9-82e8-e18e6cf7ed07.png)
## 修改IDEA行为
查找行为(默认快捷键 CTRL + SHIFT + A), 查找
>Registry

![image.png](/images/2019/08/09/5ad3d4c0-ba9b-11e9-82e8-e18e6cf7ed07.png)
找到以下选项, 勾选(立即生效)
>compiler.automake.allow.when.app.running

配置完成