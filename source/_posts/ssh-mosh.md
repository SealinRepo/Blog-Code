---
title: SSH神器-Mosh
tags:
  - 工具
  - 终端
  - UDP
categories:
  - 工具
toc: true
date: 2019-11-20 20:15:41
---

# 介绍
	Mosh（mobile shell）是一款基于UDP的远程终端软件，包含客户端和服务器两部分，用于代替SSH。因为Mosh基于UDP，所以它可以提供不间断的连接，非常适用于在网络状况不好或时延较大的网络中进行远程终端访问。比如，在GPRS或3G移动网络访问远程服务器，或者从国内访问国外服务器等特殊场景。

	简单的说,  就是如果你的服务器是个动不动就断线, 打个字要等两秒钟的磨人小妖精, 那可以试试这款神器了. 将SSH的TCP协议改用UDP协议, 解放网速, 放飞自我...

# 服务端安装
  主流的Linux分支仓库基本已经有Mosh了, 如果是CentOS这类更新较慢的分支, 可以先安装epel-release仓库, 然后就可以yum install了
  安装好就可以了, 不用启动服务.
# 客户端
  个人推荐谷歌插件, [下载地址](https://chrome.google.com/webstore/detail/mosh/ooiklbnjmhbcgemelgfhaeaocllobloj)
  安装后一样会有快捷方式.
  ![image.png](/images/2019/11/20/0dfb9ff0-0b8d-11ea-8fef-dbe8da5be383.png)
  我试过Windows版本了, 很大一个安装包, 安装后跟这货一模一样...
# 使用
  打开后界面长这样
	![image.png](/images/2019/11/20/bb3071f0-0b8d-11ea-8fef-dbe8da5be383.png)
  填写用户名 + 主机 + 端口 就可以连接了, 遗憾的是不能保存密码, 所支持的私钥认证方式在RSA类型的密码上也有问题, 据说只支持 ed25519 类型的私钥, 由于服务器是centos6, 也不支持这种类型的密钥生成, 所以也没办法测试了.
  可以自己试试是否可以使用ed25519类型的私钥进行认证.
```bash
ssh-keygen -t ed25519
```
  生成后将私钥文件(~/.ssh/id_ed25519)的内容复制到mosh的[Add ssh key]界面中.