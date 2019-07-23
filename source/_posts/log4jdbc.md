---
title: 使用log4jdbc查看带参数的完整SQL语句
tags:
  - JAVA
  - 数据库
  - 日志处理
originContent: "一般情况下日志中的SQL语句参数都是以占位符(?)进行显示, 配置参数打印后也不能打印出完整的SQL, 而是把参数在SQL语句后单独打印出来, 这样难免要跟踪SQL的时候会非常麻烦, log4jdbc可以完美的帮我们解决上述的烦恼, 并且整合起来也非常简单.\n\n## 添加依赖\n```xml\n<dependency>\n    <groupId>com.googlecode.log4jdbc</groupId>\n    <artifactId>log4jdbc</artifactId>\n    <version>1.2</version>\n</dependency>\n```\n## 修改数据库连接\n### Driver\n```yml\n# 一般配置\njdbc.driver=oracle.jdbc.driver.OracleDriver\n\n# 改为如下配置:\njdbc.driver=net.sf.log4jdbc.DriverSpy\n```\n### Url\n```yml\n# 一般配置\njdbc.url=jdbc:oracle:thin:@localhost:1521:orcl\n\n# 改为如下配置\njdbc.url=jdbc:log4jdbc:oracle:thin:@localhost:1521:orcl\n```\n## 配置完成\n此时启动项目, 可以发现在执行sql时已经可以输出SQL执行的相关信息了, 但是输出了很多跟踪信息/数据明细等日志, 这些东西可能又不是我们需要的, 例如我们只需要查看SQL, 那这些信息无疑是画蛇添足, 下边咱们来优化一波.\n\n## 支持的配置\n|logger|描述|since|\n|-|-|-|\n|jdbc.sqlonly|仅仅记录 SQL 语句，会将占位符替换为实际的参数|1.0|\n|jdbc.sqltiming|包含 SQL 语句实际的执行时间|1.0|\n|jdbc.audit|除了 ResultSet 之外的所有JDBC调用信息，篇幅较长|1.0|\n|jdbc.resultset|包含 ResultSet 的信息，输出篇幅较长|1.0|\n|jdbc.connection|输出了 Connection 的 open、close 等信息|1.2alpha1|\n\n## 优化输出格式\n既然咱们只想看SQL语句, 那从上表可以看出sqlonly这种模式正是我们需要的, 其他的选项log4jdbc默认是打开的, 我们将之关闭就可以了, 配置如下.\n### logback配置\n```xml\n\t<logger name=\"jdbc.sqlonly\" level=\"INFO\" />\n\t<logger name=\"jdbc.audit\" level=\"OFF\" />\n\t<logger name=\"jdbc.sqltiming\" level=\"OFF\" />\n\t<logger name=\"jdbc.connection\" level=\"OFF\" />\n\t<logger name=\"jdbc.resultset\" level=\"ERROR\" />\n```\n\n### log4j配置\n既然知道了配置格式, log4j也就大同小异了\n```yml\nlog4j.logger.jdbc.resultset=OFF\n...\n```\n\n\n"
categories:
  - JAVA
  - 技术
toc: true
date: 2019-07-23 16:35:07
---

一般情况下日志中的SQL语句参数都是以占位符(?)进行显示, 配置参数打印后也不能打印出完整的SQL, 而是把参数在SQL语句后单独打印出来, 这样难免要跟踪SQL的时候会非常麻烦, log4jdbc可以完美的帮我们解决上述的烦恼, 并且整合起来也非常简单.

## 添加依赖
```xml
<dependency>
    <groupId>com.googlecode.log4jdbc</groupId>
    <artifactId>log4jdbc</artifactId>
    <version>1.2</version>
</dependency>
```
## 修改数据库连接
### Driver
```yml
# 一般配置
jdbc.driver=oracle.jdbc.driver.OracleDriver

# 改为如下配置:
jdbc.driver=net.sf.log4jdbc.DriverSpy
```
### Url
```yml
# 一般配置
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl

# 改为如下配置
jdbc.url=jdbc:log4jdbc:oracle:thin:@localhost:1521:orcl
```
## 配置完成
此时启动项目, 可以发现在执行sql时已经可以输出SQL执行的相关信息了, 但是输出了很多跟踪信息/数据明细等日志, 这些东西可能又不是我们需要的, 例如我们只需要查看SQL, 那这些信息无疑是画蛇添足, 下边咱们来优化一波.

## 支持的配置
|logger|描述|since|
|-|-|-|
|jdbc.sqlonly|仅仅记录 SQL 语句，会将占位符替换为实际的参数|1.0|
|jdbc.sqltiming|包含 SQL 语句实际的执行时间|1.0|
|jdbc.audit|除了 ResultSet 之外的所有JDBC调用信息，篇幅较长|1.0|
|jdbc.resultset|包含 ResultSet 的信息，输出篇幅较长|1.0|
|jdbc.connection|输出了 Connection 的 open、close 等信息|1.2alpha1|

## 优化输出格式
既然咱们只想看SQL语句, 那从上表可以看出sqlonly这种模式正是我们需要的, 其他的选项log4jdbc默认是打开的, 我们将之关闭就可以了, 配置如下.
### logback配置
```xml
	<logger name="jdbc.sqlonly" level="INFO" />
	<logger name="jdbc.audit" level="OFF" />
	<logger name="jdbc.sqltiming" level="OFF" />
	<logger name="jdbc.connection" level="OFF" />
	<logger name="jdbc.resultset" level="ERROR" />
```

### log4j配置
既然知道了配置格式, log4j也就大同小异了
```yml
log4j.logger.jdbc.resultset=OFF
...
```
这样配置后输出的日志中就只有SQL信息了

