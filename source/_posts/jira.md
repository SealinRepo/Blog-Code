---
title: Jira安装及激活方法
tags:
  - 工具
originContent: ''
categories:
  - 工具
toc: true
date: 2019-12-05 13:57:50
---

# 下载Jira
[下载地址:](https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.5.1-x64.bin)
# 安装
```bash
chmod +x atlassian-jira-software-8.5.1-x64.bin
./atlassian-jira-software-8.5.1-x64.bin
```
安装到最后一步, 会提示是否启动, 此时选择不启动, 因为我们还有部分后续配置没完成.
# 下载Mysql驱动
不使用mysql数据库此步骤可忽略
如果需要使用Mysql数据库保存jira数据, 需要额外下载Mysql驱动, [下载地址](https://mvnrepository.com/artifact/mysql/mysql-connector-java)
根据自己数据库版本下载连接驱动, 放到 Jira安装目录/atlassian-jira/WEB-INF/lib/
# 激活补丁
[下载地址](https://gitee.com/pengzhile/atlassian-agent/attach_files/283101/download)
下载好补丁后, 执行
```bash
java -jar atlassian-agent.jar
```
输出结果如下:
![image.png](/images/2019/12/05/9e05f400-170f-11ea-b10a-61d52cc50a10.png)
将框选部分内容添加到: Jira安装目录/bin/setenv.sh
![image.png](/images/2019/12/05/cae58850-170f-11ea-b10a-61d52cc50a10.png)
# 启动Jira服务
```bash
/jira安装目录/bin/startup.sh
ps -aux | grep java # 正常情况应包含我们添加的agent信息
```

## 选择自定义安装
![image.png](/images/2019/12/05/13abe750-1710-11ea-b10a-61d52cc50a10.png)

## 连接数据库
此处我为了方便以后保留数据,重新安装什么的, 选择Mysql数据库, 当然也可以选择内置数据库进行安装
填好数据库信息, 点 **测试连接**
![image.png](/images/2019/12/05/30a209b0-1711-11ea-b10a-61d52cc50a10.png)
没问题的话可以点**下一步**进行初始化数据库了.
![image.png](/images/2019/12/05/a009ef70-1711-11ea-b10a-61d52cc50a10.png)
## 生成激活码
到下一步界面上会有一个**B6K8-XXXX-XXXX-UPLZ**这样的序列号, 记录下来, 使用agent这个jar包生成激活码
```bash
java -jar atlassian-agent.jar -p jira -m 邮箱 -n 名字 -o 组织 -s 序列号
```
将**邮箱,名字,组织,序列号**替换为自己的, 生成激活码
![image.png](/images/2019/12/05/fdf09c80-1723-11ea-b10a-61d52cc50a10.png)
拷贝框选部分到页面上的输入框, 下一步, 等几分钟就可以转到设置管理员用户的界面了.