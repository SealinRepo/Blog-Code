---
title: Markdown转Word
tags:
  - 工具
  - Swagger
  - Writage
originContent: ''
categories:
  - 工具
  - 经验
toc: true
date: 2019-12-06 11:47:55
---

# 工具介绍
项目采用Swagger生成在线文档, 年度备份时是用word文件保存, swagger本身没有产生Markdown文本的功能, 可以使用[knife4j](https://doc.xiaominfo.com/guide/useful.html)这个增强UI进行生成.
生成Markdown格式的文本后, 可以安装[Writage](http://www.writage.com/)这个word插件, 让word支持markdown文本解析.
[Writage下载地址(Windows)](/download/Markdown2Word-Writage-1.12.msi)

# 使用
knife4j提供了SpringBoot starter, 会自动依赖swagger, 所以以前的项目中依赖的swagger和swaggerUI这些包可以拿掉了, springboot项目直接依赖starter, 其他项目可以根据[ Knife4j文档 ](https://doc.xiaominfo.com/guide/useful.html)进行配置.
启动后访问 /doc.html 界面如下:
![image.png](/images/2019/12/06/a871a170-17da-11ea-badd-f984b5c223a4.png)
在**离线文档(MD)**界面可以快速拷贝为Markdown文本.
安装 Writage 后, *.md文件的图标会变成word, 将复制的文本信息复制到一个新建的md格式文件里, 使用word打开就可以自动解析了.
![image.png](/images/2019/12/06/fbb9ab20-17da-11ea-badd-f984b5c223a4.png)
打开后另存为 .docx 格式的文件即可.